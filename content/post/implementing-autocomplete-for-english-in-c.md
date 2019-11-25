---
title: Implementing autocomplete for English in C++
date: 2016-03-27T22:07:35+00:00
author: "Taras Kushnir"
permalink: /implementing-autocomplete-for-english-in-c/
image: letters-heap.jpeg
categories:
  - C++
  - Programming
keywords:
  - autocomplete
  - c++
  - cross-platform
  - interface
  - linux
  - makefile
  - os x
  - ruby
  - tsv
  - windows
---
When it comes to implementing autocompletion in C++ in some type of input field, the question is which algorithm to choose and where to get the source for completion. In this post I'll try to answer both questions.

As for the algorithm, SO gives us hints about tries, segment trees and others. You can find <a href="http://dhruvbird.blogspot.com.ee/2010/09/very-fast-approach-to-search.html" target="_blank">good article</a> about them. Author has implemented some of them in a repository called FACE (fastest auto-complete in the east). You can easily find it <a href="https://github.com/duckduckgo/cpp-libface" target="_blank">on GitHub</a>. This solution is used for the autocompletion in search engine Duck-Duck-Go which should tell you how good it is. Unfortunately their solution requires dependencies on `libuv` and joyent `http-parser`, which is not good in case you need just to integrate autocompletion functionality into your C++ application, but not build auto-complete server and send queries to it. Another drawback - `libuv` and `cpp-libface` itself fails to compile in Windows which is bad in case you're building cross-platform solution.

You can find out how to built FACE into your cross-platform C++ application below.

<!--more-->

Here comes my fork: <a href="https://github.com/Ribtoks/cpp-libface" target="_blank">library version of cpp-libface</a>. I've removed dependencies on `libuv` and `http-parser`. For Windows, one would need also `mman` - memory-mapped files IO. The only such thing I was able to find was `mman-win32` - cygwin port of mman from *nix'es. After puking thousands of warning, VS compiler did it's job and in real life `mman-win32` worked so it's ready at least for testing. I've created two Makefiles: for Windows and OS X/Linux. After building you will get library FACE (`libface.lib` or `libface.a`) and PoC executable for testing of auto-completion through the command line.

As for the interface of libface, I have left only two methods:

```public:
    bool import(const char *ac_file);
    vp_t prompt(std::string prefix, uint_t n = 16);
```

one of which allows you to import a file and another - generate completion suggestions.

But it's only half of the story. Also you need source for completion. Library FACE is able to digest TSV files (Tab-Separated Values) where first column is frequency of phrase/word and second column - phrase/word itself. After searching through the internet for some time I've found <a href="https://github.com/mozilla-b2g/gaia/tree/master/apps/keyboard/js/imes/latin/dictionaries" target="_blank">frequency tables for different languages</a> for Android. There are in the XML with simple structure and simple Ruby script written in 5 minutes transformed them into TSV:

```ruby
require 'nokogiri'
doc = File.open(ARGV.first) { |f| Nokogiri::XML(f) }

result_filename = ARGV.first.sub(/[.]xml$/, '.tsv')
tsv_file = File.open(result_filename, 'w')

doc.xpath('//w').each do |node|
  frequency = node['f'];
  word = node.content

  tsv_file.puts("#{frequency}\t#{word}")
end

tsv_file.close

puts "Done"
```

Now everything is ready and you can easily integrate library FACE into your C++ solution and fed it with tsv dicts.
