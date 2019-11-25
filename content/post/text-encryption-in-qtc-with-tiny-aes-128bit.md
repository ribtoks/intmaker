---
title: Text encryption in Qt/C++ with tiny AES 128bit
date: 2015-02-24T15:12:39+00:00
author: "Taras Kushnir"
permalink: /text-encryption-in-qtc-with-tiny-aes-128bit/
image: lock-411227.jpeg
categories:
  - C++
  - Programming
  - Qt
keywords:
  - aes
  - c++
  - data
  - encryption
  - qt
  - text
---
Have you ever needed a small, really small encryption in your C++ project for some piece of text? Say, credentials, login details or any other sensitive data? Of course, the best way is to keep just hash of salted password, but... What if you just **need** to do it and the size is so much critical for you?

There're <a href="https://www.openssl.org/" target="_blank" rel="noopener">openSSL</a> library and <a href="http://www.cryptopp.com/" target="_blank" rel="noopener">Crypto++</a> library which are monsters with tons of encryption algorithms, used in a number of solid projects etc. But.. they are big! I don't want 30Mb library in my tiny project, which weights 10 Mb with high-resolution icons for OS X which weight by itself 5Mb. So I don't want to sacrifice the size but still need encryption. Meet <a href="https://github.com/kokke/tiny-AES128-C" target="_blank" rel="noopener">tiny-AES.</a> It's really small AES 128-bit library which does encryption in <a href="https://en.wikipedia.org/wiki/Block_cipher_modes_of_operation" target="_blank" rel="noopener">CBC and ECB modes</a>. It really contains everything you needed just to encrypt and decrypt your sensitive data and forget about it.

You can find example under the hood.

<!--more-->

Tiny-AES is not super-strong, it implements AES, it's build for ARMs.. but it works! It's easy to adopt to any C++ project. Here's how I implemented encryption in my small Qt C++ project.

Key for AES-128 should be 128 bit length, so I used MD5 hashing to get exactly 128 bit buffer for AES key. Cypher-text should be 16 bit aligned, so I use my own inlined alignment function (which could be macro etc.). Also I used _utf8()_ method of QString to get pointer to underlying _ushort*_ buffer and to encode directly it.

```cpp
#ifndef AESQT_H
#define AESQT_H

# other includes
#include <path/to/tiny-aes/aes.hpp>

namespace Encryption {

    const uint8_t iv[] = { 0xf0, 0xe1, 0xd2, 0xc3, 0xb4, 0xa5, 0x96, 
              0x87, 0x78, 0x69, 0x5a, 0x4b, 0x3c, 0x2d, 0x5e, 0xaf };

    inline int getAlignedSize(int currSize, int alignment) {
        int padding = (alignment - currSize % alignment) % alignment;
        return currSize + padding;
    }

    QString encodeText(const QString &rawText, const QString &key) {
        QCryptographicHash hash(QCryptographicHash::Md5);
        hash.addData(key.toUtf8());
        QByteArray keyData = hash.result();

        const ushort *rawData = rawText.utf16();
        void *rawDataVoid = (void*)rawData;
        const char *rawDataChar = static_cast<const char*>(rawDataVoid);
        QByteArray inputData;
        inputData.append(rawDataChar, rawText.size() * sizeof(QChar) + 1);

        const int length = inputData.size();
        int encryptionLength = getAlignedSize(length, 16);
        Q_ASSERT(encryptionLength % 16 == 0 && encryptionLength >= length);

        inputData.resize(encryptionLength);
        for (int i = length; i < encryptionLength; i++) { inputData[i] = 0; }

        struct AES_ctx ctx;
        AES_init_ctx_iv(&ctx, (const uint8_t*)keyData.data(), iv);
        AES_CBC_encrypt_buffer(&ctx, (uint8_t*)inputData.data(), encryptionLength);

        QString hex = QString::fromLatin1(inputData.toHex());
        return hex;
    }

    QString decodeText(const QString &hexEncodedText, const QString &key) {
        QCryptographicHash hash(QCryptographicHash::Md5);
        hash.addData(key.toUtf8());
        QByteArray keyData = hash.result();

        const int length = hexEncodedText.size();
        int encryptionLength = getAlignedSize(length, 16);

        QByteArray encodedText = QByteArray::fromHex(hexEncodedText.toLatin1());
        const int encodedOriginalSize = encodedText.size();
        Q_ASSERT(encodedText.length() <= encryptionLength);
        encodedText.resize(encryptionLength);
        for (int i = encodedOriginalSize; i < encryptionLength; i++) { encodedText[i] = 0; }

        struct AES_ctx ctx;
        AES_init_ctx_iv(&ctx, (const uint8_t*)keyData.data(), iv);
        AES_CBC_decrypt_buffer(&ctx, (uint8_t*)encodedText.data(), encryptionLength);

        encodedText.append("\0\0");
        void *data = encodedText.data();
        const ushort *decodedData = static_cast<ushort *>(data);
        QString result = QString::fromUtf16(decodedData, -1);
        return result;
    }
}

#endif // AESQT_H
```

I wrote <a href="https://github.com/ribtoks/xpiks/blob/master/src/xpiks-tests/xpiks-tests-core/encryption_tests.cpp" target="_blank">a bunch of tests</a> against that functions so the solution is proven to be working. So if you're looking for really tiny encryption or AES implementation, use <a href="https://github.com/kokke/tiny-AES128-C" target="_blank" rel="noopener">tiny-AES-128</a>!
