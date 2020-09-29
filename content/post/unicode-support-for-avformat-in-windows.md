---
title: Unicode support for avformat_open_input in Windows
date: 2017-03-31T13:59:09+00:00
author: "Taras Kushnir"
image: hieroglyphs-590663.jpeg
categories:
  - C++
  - Programming
keywords:
  - ffmpeg
  - libav
  - path
  - thumbnail
  - unicode
  - video
  - windows
aliases:
  - /2017/unicode-support-for-avformat_open_input-in-windows
---
For those of us ever writing cross-platform application there has always been enough quires and quests to accomplish. Typical one is to correctly handle multibyte/unicode filepaths in Windows. And though they are handled pretty good in Qt, when you write your own library you have to do it yourself.

Another level of quests is using third-party libraries which were not designed for cross-platform usage. For example if you wanted to use ffmpeg / libav libraries in Windows, you have to deal with lack of support of `std::wstring` parameters in the API. One way to deal with it - arrange a custom IO using `AVFormatContext` and handle file paths by yourself. I have found a wonderful article and code example of how to do it in the [blog of Marika Wei](https://mw.gl/posts/ffmpeg_custom_io/). Slightly adapted, the solution will handle all Windows paths

```cpp
struct {
#ifdef _WIN32
    std::wstring m_FilePath;
#else
    std::string m_FilePath;
#endif
    AVIOContext *m_IOCtx;
    uint8_t *m_Buffer; // internal buffer for ffmpeg
    int m_BufferSize;
    FILE *m_File;
}

#ifdef _WIN32
    m_File = _wfopen(m_FilePath.c_str(), L"rb");
#else
    m_File = fopen(m_FilePath.c_str(), "rb");
#endif

m_IOContext = avio_alloc_context(
    m_Buffer, m_BufferSize, // internal buffer and its size
    0, // write flag (1=true, 0=false)
    (void*)this, // user data, will be passed to our callback functions
    IOReadFunc,
    0, // no writing
    IOSeekFunc
);
```
