---
title: How to process Nikon D5300 NEF and other unsupported RAW formats in Darktable
date: 2014-09-09T12:26:07+00:00
author: "Taras Kushnir"
permalink: /how-to-process-nikon-d5300-nef-and-other-unsupported-raw-formats-in-darktable/
categories:
  - Linux
keywords:
  - 12 bit
  - 14 bit
  - adobe
  - d5300
  - darktable
  - dng
  - linux
  - nef
  - nikon
  - open
  - photography
  - raw
  - wine
---
My favorite tool, Darktable, does not support new Nikon D5300 NEF format and, obviously, a lot of other RAW formats due to proprietary software for them. But there is a solution. There is a <a href="http://www.adobe.com/support/downloads/product.jsp?product=106&platform=Windows" target="_blank">free tool from Adobe for Windows</a> and Mac: DNG converter, which is free and converts a lot (almost all, I guess) of proprietary RAW formats to <a href="http://en.wikipedia.org/wiki/Digital_Negative" target="_blank">DNG </a>(Digital NeGative - open lossless raw format. <a href="http://helpx.adobe.com/photoshop/camera-raw.html" target="_blank">Complete list of supported RAW formats</a> for DNG Converter.

[<img class="aligncenter wp-image-1123 size-large" src="http://code.jamming.com.ua/wp-content/uploads/2014/09/snapshot2-1024x616.png" alt="snapshot2" width="604" height="363" srcset="http://code.jamming.com.ua/wp-content/uploads/2014/09/snapshot2-1024x616.png 1024w, http://code.jamming.com.ua/wp-content/uploads/2014/09/snapshot2-300x181.png 300w, http://code.jamming.com.ua/wp-content/uploads/2014/09/snapshot2-768x462.png 768w, http://code.jamming.com.ua/wp-content/uploads/2014/09/snapshot2.png 1180w" sizes="(max-width: 604px) 100vw, 604px" />](http://code.jamming.com.ua/wp-content/uploads/2014/09/snapshot2.png)

For now DNG Converter is fairly simple so it runs flawlessly under Wine and you're able to export all your NEF (or other) photos first to DNG and then to process them in your favorite RAW editor. That's it!

(in my case then I switched Alt-Tab from working DNG converter, it crashed under Wine, but when window is active all the time, it works ok)

Also, if you set file format to 12-bit NEF in your camera, it would have green colors if you open them in any RAW editor (Darktable, RAWTherapee, Lighthouse, etc). But if you chose 14-bit RAW, it looks ok despite the fact it would be more sharp in native Nikon Windows- and Mac-only software.

If you have any questions, feel free to ask them in commets!

**UPD:** Darktable 1.6 supports D5300 NEF files if processed without OpenCL
