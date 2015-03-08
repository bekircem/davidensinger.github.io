---
date: 2013-08-04 12:19:00
layout: post
title: 'How I use Reduce to Minify and Optimize Assets for Production'
description: 'An easy way to both minify CSS, JavaScript, and HTML and optimize images.'
tags: ['Minify', 'Rake', 'Webperf']
sitemap:
  lastmod: 2014-11-07
suggested_tweet:
  hashtags: ['Jekyll', 'jekyllrb']
  related: ['grosser‎', 'pacbard', 'jekyllrb']
---

Now that I’ve set up [my development environment with Rake](http://davidensinger.com/2013/07/easy-sass-source-maps-with-development-environments-and-rake/), I’m able to easily minify and optimize the assets I use for my site. To do this I use [Reduce](https://github.com/grosser/reduce), which is a fantastic Ruby gem by [Michael Grosser](http://grosser.it/) to minify and compress CSS, HTML, JavaScript and Jpeg and PNG, amongst other formats.

<div class="yellow-box">
  <p><strong>Please Note:</strong> I no longer use source maps with Sass or use Rake to manage development environments. The following code may still be helpful for some folks, even though I’ve moved on to using Grunt to compile my Sass.</p>
</div>

## Why Compress Assets?
The obvious benefit to minifying text and optimizing images is that it reduces filesize, so the assets load faster. This is the low hanging fruit of front end optimization and usually quite impactful on performance, despite being so easy to do.

## My Set Up
To use Reduce I added it to my site’s project with **Bundler** so that I may then invoke it in my **Rake** task to build for production. A benefit to having separate environments for development and production is that I can delay all optimization to my production build, which saves considerable time during development.

I previously tried several **Jekyll** plugins to manage this, but didn’t like the delay between saving a file and then waiting for Jekyll to rebuild. Invariably, the bottleneck was the file compression, especially in regards to the minification of the HTML, which is perhaps due to the sheer quantity of files involved.

The task I invoke to minify the assets is courtesy of Emanuele Bardelli, who added the HTML compression functionality to Reduce. See [his rake task](https://github.com/pacbard/blog/blob/master/_rake/minify.rake), which I’ve only superficially modified:

{% highlight ruby %}
desc "Minify _site/"
task :minify do
  puts "\n## Compressing static assets"
  original = 0.0
  compressed = 0
  Dir.glob("_site/**/*.*") do |file|
    case File.extname(file)
      when ".css", ".gif", ".html", ".jpg", ".jpeg", ".js", ".png", ".xml"
        puts "Processing: #{file}"
        original += File.size(file).to_f
        min = Reduce.reduce(file)
        File.open(file, "w") do |f|
          f.write(min)
        end
        compressed += File.size(file)
      else
        puts "Skipping: #{file}"
      end
  end
  puts "Total compression %0.2f\%" % (((original-compressed)/original)*100)
end
{% endhighlight %}

How easy is that? With a simple `rake build:pro` command I’m able to build my site with production ready assets, which I then commit and deploy to GitHub.
