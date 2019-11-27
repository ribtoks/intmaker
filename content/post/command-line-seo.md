---
title: Command-line SEO alternative to Screaming Frog Spider
date: 2019-11-27T08:19:03-08:00
author: "Taras Kushnir"
image: seo.jpg
categories:
  - Programming
keywords:
  - seo
  - go
  - command
  - line
---

Did you ever wanted to crawl a list of URLs and figure out which of them are alive? Or what number of external links any of them has? If you did, most probably [Screaming Frog SEO Spider](https://www.screamingfrog.co.uk/seo-spider/) was one of the popular choices. Bad news is that free version has a limitation to 500 URLs and paid version costs lots of money (around $150 per year). And worst is that 80% of that crawling functionality can be done in pure Bash.

<!--more-->

However, pure Bash has also some limitations, so I decided to spend a day and create a better tool, written in Go. Meet [mink](https://github.com/ribtoks/mink), a command-line SEO spider. Mink can crawl a list of URLs and extract useful metrics like HTTP status code, indexibility, number of external and internal links and many many others.

## Usage

Here's a list of options:

```
Usage of mink:
  -d int
    	Maximum depth for crawling (default 1)
  -f string
    	Format of the output table|csv|tsv (default "table")
  -v	Write verbose logs
```

`mink` reads URLs from `STDIN` and writes reports to `STDOUT`. Report can be written in a form of a table, comma-separated values and tab-separated values.

Probably most viable report format is CSV since you can do fancy things like [resource page link building](https://www.youtube.com/watch?v=8f4YTubL6cM) in a Spreadsheet software like [Google Docs](https://docs.google.com/) or [LibreOffice](https://www.libreoffice.org/).

Here are some examples of usage. In order to crawl the whole website, you can do this:

`echo "https://your-website.com" | mink -d 1000 -f csv > report.csv`

whereas to crawl a file with an URL per line, do this instead:

`cat urls.txt | mink -f csv > report.csv`

## Contibuting

[mink](https://github.com/ribtoks/mink) is an open-source project so you can make it better too! Feel free to send a pull request or reach out to me if you have any feature requests.
