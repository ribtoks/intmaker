---
title: Migrating blog from Wordpress to Jekyll
date: 2018-03-01T00:23:10+00:00
author: "Taras Kushnir"
permalink: /migrate-wordpress-to-jekyll/
image: carry.jpeg
categories:
  - Programming
tags:
  - wordpress
  - jekyll
  - ruby
  - bash
---

Living in the modern age of 0-day vulnerabilities is complicated when you have a Wordpress blog. Since I didn't have anything fancy in there I decided to migrate it to static pages. Sounds easy? I'm sure it does. See what it takes to migrate an average blog with images and crosslinks. I've organized my process in a sort of a step by step manual.

<!--more-->

## Step 1. Export all Wordpress data

Go to Wordpress admin panel, find there "Export everything to XML" and download it to your computer. "Everything" for sure should _not include images_. If you have many of them, your server will timeout especially if you bought a cheap hosting.

Download your media using FTP client like [FileZilla](https://filezilla-project.org/) to a location on your computer.

## Step 2. Initial import

Jekyll provides you with rubygem which allows to do import of Wordpress website. This gem is not very nice but at least it's something. The problem with it that it tries literally to replicate your Wordpress website in static pages, but what we actually need is raw data (posts, pages). But we'll get to that as well.

As for now, go to [official page of wordpress importer](http://import.jekyllrb.com/docs/wordpressdotcom/) and proceed with instructions there.

## Step 3. Posts and metadata cleanup

All posts imported with Jekyll plugin are pretty "dirty". They contain tons of useless metadata in post header which should definitely be pruned. Also many if not all links are broken along with gallery media references.

It will be a damn good idea to use source control and commit all the original ("dirty") files in there so you will be sure you will not screw them.

In order to clean them up, I wrote a [simple script](https://github.com/ribtoks/heap/blob/master/scripts/fix_jekyll_post.rb) in Ruby, but it is very specific for my needs for you to use it as is. The main idea is that I just go through each post and fix it with my script line by line:

```ruby
filepath = ARGV[0]

text = File.read(filepath)
out = File.open(filepath, "w")

text.each_line do |line|
  next if line =~ /^type:/
  next if line =~ /^published:/
  next if line =~ /^password:/
  next if line =~ /^status:/
  next if line =~ /^parent_id:/

  line.gsub!("<p>", "\n")
  line.gsub!("</p>", "")

  line.gsub!(/<br[ \/]+?>/, "\n")

  h3_match = line.match(/^<h3>(.+)<\/h3>$/)
  if h3_match != nil then
    text = h3_match.captures.first
    out.puts "\n### #{text}\n"
    next
  end

  line = remove_gallery(line)
  line = fix_caption_images(line)
  line = fix_images(line)
  line = fix_links(line)

  out.puts line
end

puts "Done"
```

As you can see I'm just moving data from input file to output file letting all lines through some filtering and fixing.

For example, here's a code of `fix_caption_images()` routine

```ruby
def fix_caption_images(str)
  # image with caption
  str.gsub(/\[caption.*?\]<a href=\"(.+?)\".*?><img .*?\/><\/a>(.+?)\[\/caption\]/) { |match|
    href, title = $1, $2
    img_path = href.sub("http://url.of.my.blog.com/wp-content/uploads/", "")
    img_title = title.strip

    "![#{img_title}]({{ site.baseurl }}/assets/images/posts/#{img_path})\n*#{img_title}*"
  }
end
```

Typical Ruby/Perl regex style: you debug it, it works and you forget how it works in a month. Nevertheless, it worked flawlessly. Now all bad links, galleries and images are normalized. You just launch a script like that on every file in `_posts/` directory.

Also if you have non-latin characters, Jekyll importer will make all links and filenames url-encoded. In order to fix that, just use `URI.decode()` method for links and filenames.

## Step 4. Images cleanup

If you will look though what you have download using FTP client, you will see tons of garbage like pregenerated all sorts of thumbnails of different sizes of your images. Of course you want to remove them:

```
find -E . -type f -regex '.*-.*[0-9]+x[0-9]+\.jpg' -exec rm -rf {} \;
```

Also if your blog contains some worthy images, somebody can use them to sell at microstocks. You might want to use watermarks, but it's much easier just to scale them below 4 Megapixels so they will be useless for selling. I personally chose second option hence another `find` call:

```
find . -type f \( -name '*.jpg' -o -name '*.png' -o -name '*.jpeg' \) -exec mogrify -resize 1500x1500\> -quality 90 {} \;
```

I used `mogrify` command from [ImageMagic](https://www.imagemagick.org) in order to scale down images and reduce their quality to 90% which is more than enough for web, but renders them useless for microstocks and saves lots of space. Also you can host images on Flickr or any other external source.

## Step 5. New blog setup

Well, now you're kind of good to go with your own style/template. All data is properly imported and cleaned up. Just move it to your shiny template into `_posts` and `assets/` directories.

Edit your `_config.yml`:
* set production `url` of your website
* set `permalink` style
* set `gems` , `title` and other important properties

## Step 6. Prettifying URLs

In case you rely on url style "http://myblog.com/mypost.html" and it looks a little bit ugly for you, I have good news. _It looks ugly for me as well_. I would prefer "http://myblog.com/mypost" much more without that nasty ".html" in the end. And if you cannot do that in Jekyll's `_config.yml` for any reason, it can be easily achieved using URL Rewrite in Apache with the following rules added to your website's root `.htaccess` file:

```
RewriteEngine On
RewriteCond %{REQUEST_URI} !^.*\.html$
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteCond %{REQUEST_FILENAME}\.html -f
RewriteRule ^(.*)$ %{REQUEST_FILENAME}.html
```

Which means that if we go to url without ".html" and we have a file with same name and extension ".html" we should serve it instead of 404 page.

## Enjoy your new blog

Now it's time to check your blog locally with `bundle exec jekyll serve` if everything works as expected. When all the minor issues are fixed would be a great time to finally publish it to GitHub Pages or just to generate static pages and upload them to your hosting (probably the same one which used to run Wordpress).
