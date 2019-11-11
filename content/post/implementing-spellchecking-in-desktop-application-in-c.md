---
title: Implementing spellchecking in desktop application in C++
date: 2016-06-05T12:24:38+00:00
author: "Taras Kushnir"
permalink: /implementing-spellchecking-in-desktop-application-in-c/
image: spellcheck-503898.jpg
categories:
  - C++
  - Programming
  - Qt
tags:
  - checking
  - consumer
  - error
  - hunspell
  - myspell
  - producer
  - qt
  - spelling
---
When user is supposed to enter significant amount of text in your application, it's better to help him/her to control it with checking spelling. Basically, to check spelling you need a dictionary with words and algorithm to order these words. Also it might be useful to provide user with possible corrections for any spelling error. Here where [Hunspell](https://hunspell.github.io/) comes handy. It's an open source library built on top of MySpell library and used in a significant number of projects varying from open source projects like Firefox to proprietary like OS X. It contains bindings to a number of platforms (.NET, Ruby etc.) and should be fairly easy to integrate to your project. In this post I'll discuss how to integrate it to C++/Qt project.

<!--more-->

First of all, you should download Hunspell source code and try to build it for your platform. You can take a look at README for the instructions. Once you're done, it's time to link just built library with your project. Also you should add Hunspell include files to your include path in order to use the API.

After you're done, let's include C++ Hunspell API to your header and add following variable to your class

`Hunspell *m_Hunspell;`

Constructor of Hunspell class takes paths to DIC and AFF files (wordlist and affix files). If you're building cross-platform solution, it will be useful to know, that to handle utf-8 paths in Windows, you need to prefix paths to DIC and AFF files with "\\?\". Loading code in my Qt project looks like this:

```cpp
#ifdef Q_OS_WIN
// specific Hunspell handling of UTF-8 encoded pathes
affPath = "\\\\?\\" + QDir::toNativeSeparators(affPath);
dicPath = "\\\\?\\" + QDir::toNativeSeparators(dicPath);
#endif

try {
    m_Hunspell = new Hunspell(affPath.toUtf8().constData(),
                              dicPath.toUtf8().constData());
    LOG_DEBUG << "Hunspell with AFF" << affPath << "and DIC" << dicPath; 
    m_Encoding = QString::fromLatin1(m_Hunspell->get_dic_encoding());
    m_Codec = QTextCodec::codecForName(m_Encoding.toLatin1().constData());
}
catch(...) {
    LOG_DEBUG << "Error in Hunspell with AFF" << affPath << "and DIC" << dicPath;
    m_Hunspell = NULL;
}
```

In this code except of instantiating Hunspell class we also get right Codec to query the dictionary. Now you can use API's of `Hunspell` class to access spellchecking API. To get real AFF and DIC files, you can check out a number of open source projects which use spellchecking and hunspell - e.g. OpenOffice.

The most common operation is, of course, to check if particular word is spelled OK or not:

```cpp
bool isSpellingCorrect(const QString &word) const {
bool isOk = false;
try {
    isOk = m_Hunspell->spell(m_Codec->fromUnicode(word).constData()) != 0;
}
catch (...) {
    isOk = false;
}
return isOk;
}
```

This demonstrates also usage of `Codec` retrieved before.

Besides of checking spelling, it's useful to provide user with corrections for the particular word. `Hunspell` class has API for this and it can be used like this:

```cpp
QStringList suggestCorrections(const QString &word) {
    QStringList suggestions;
    char **suggestWordList = NULL;

    try {
        // Encode from Unicode to the encoding used by current dictionary
        int count = m_Hunspell->suggest(&suggestWordList, m_Codec->fromUnicode(word).constData());
        QString lowerWord = word.toLower();

        for (int i = 0; i < count; ++i) { 
            QString suggestion = m_Codec->toUnicode(suggestWordList[i]);
            suggestions << suggestion;
            free(suggestWordList[i]);
        }
    }
    catch (...) {
        LOG_WARNING << "Error for keyword:" << word;
    }

    return suggestions;
}
```

This code demonstrates usage of `suggest()` API of Hunspell object. Also useful tip would be to check case of the suggestion, since Hunspell can correct you word like "europe" with "Europe" and stuff like that.

If you're going to check spelling on the fly it might be a good idea to combine this approach with [producer-consumer implemented in Qt](http://code.jamming.com.ua/classic-producer-consumer-in-qtc/). So your app's UI will be responsive while background worker will serve spelling requests.

That's all folks.
