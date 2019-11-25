---
title: How to download huge folder from Dropbox
date: 2015-10-14T15:53:49+00:00
author: "Taras Kushnir"
permalink: /how-to-download-folder-from-dropbox/
image: heavy.jpg
categories:
  - Programming
keywords:
  - download
  - dropbox
  - javascript
  - parse
  - wget
---
If you face a problem to download folder from Dropbox which contains tons of files, no known browser extension can help you. Dropbox moves each file download to it's separate page and you can't do it directly.

When I faced this problem I knew I would need to create my own solution and quick googling just confirmed that.

I opened javascript console and extracted all links from the folder. Then I replaced "dl=0" to "dl=1" to get actual download link.

```javascript
var links = document.querySelectorAll("div.filename a")
var processed = Array.prototype.map.call(links, 
  function(link) { 
    return link["href"].replace("dl=0", "dl=1"); 
})
console.log(processed.join("\n"))
```

After I copied those to file _links\_to\_download_. If your _processed_ array is too big, you can print it to console by chunks, using `slice(start, end)` method from Javascript. Now the problem is to download them.

I came up with _wget_ for such problem. Linux and OS X users should have wget available (OS X users can install it via e.g. homebrew). Windows users have to download it separately, install and add to the _PATH_ environmental variable. Additionaly I used _-trust-server-names_ and _-content-disposition_ parameters to save real filenames instead of dropbox hashed url. Then I faced a problem that it fails to download a file on first request and request timeout is quite big so I've set it to 5 seconds. Now it makes several timed-out requests, but they quickly resolve to the successful one.

```
wget --content-disposition --trust-server-names --timeout=5 -i links_to_download
```

Also in order to download via https in Windows you probably will need to use "_-no-check-certificate_".
