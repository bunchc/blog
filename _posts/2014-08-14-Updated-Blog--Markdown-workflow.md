---
title: "Updated Blog / Markdown workflow"
layout: post
date: 2014-08-14
categories: 
---

A while ago I posted about my markdown [workflow](http://openstack.prov12n.com/my-markdown-workflow-for-now/). While that workflow was decent and worked for a goodly little while, it left quite a bit to be desired. Specifically, I the posting process into Wordpress was cumbersome at best.

## A New Blog Engine

Like I said above, working with Wordpress and markdown was cumbersome and then some. So it was time for something new. At first I was going to use similar to [this](https://github.com/chalupaul/chalupaul.github.io/), which is actually what is in use on the [openstackcookbook.com](http://openstackcookbook.com/) site. It has the following benefits:

- Posts are written in Markdown
- Integrated with gh-pages
- Straighforward publishing process ```git push origin master```

It was *almost* what I needed. In the end I went with [Hugo](http://hugo.spf13.com/). What Hugo added was some pluggable themes and templates. Also the ability to run locally before pushing to github.

> To setup Hugo on github, use [this](http://hugo.spf13.com/tutorials/github_pages_blog).

## Current Workflow

My current workflow needed some help then. I kept the same sublime text plugins, that is:

- MarkdownEditing
    + This has a number of really handy keyboard bindings. It also has some decent highlighting and what not.
- Markdown Preview
    + This one allows me to go from Markdown into what it’ll look like on the web.
- Markdown TOC
    + This lets me go from a basic set of sections and files into a more full fledged table of contents.

Additionally, to create a new post, I modified the Rakefile found [here](https://github.com/chalupaul/chalupaul.github.io/blob/master/_posts/Rakefile), to look like this:

```
require 'fileutils'
task :post do
    title = ENV['title'] || "new-post"
    tags = ENV['tags'] || ''
    make_img_dir = ENV['imgdir'] || false
    date = (ENV['date'] ? Time.parse(ENV['date']) : Time.now).strftime('%Y-%m-%d')
    filename = File.join('.', "#{Time.now.strftime('%Y-%m-%d')}-#{title.strip.gsub(' ', '-').gsub(/[^\w-]/, '')}.md")
    open(filename, 'w') do |post|
        post.puts "---"
        post.puts "title: \"#{title}\""
        post.puts "date: #{Time.now.strftime('%Y-%m-%d')}"
        post.puts "categories: "
        post.puts "---"
        post.puts "\nYour content here."
        if make_img_dir
            img_dir = File.basename(filename.chomp(File.extname(filename)))
            FileUtils.mkdir_p("../images/posts/#{img_dir}")
            post.puts "\n" * 5
            post.puts "[imgdir]: /images/posts/#{img_dir}/"
        end
    end
end
```

So creating a new post goes like this:
bunchc: blog/content/posts$ rake post title="Title Here"

In turn, that creates a YYYY-MM-DD-Title-Here.md file for editing in sublime. It also adds the metadata section at the top of the file for me:

```
---
title: "Updated Blog / Markdown workflow"
date: 2014-08-14
categories: 
---
```

From there, I write the file, save the file, and run ```./deploy.sh``` from the Hugo installer linked earlier. That handles all the pushing and bits to git.

## Summary
The gist of it is, Wordpress was a bit much and a bit heavy for what I needed. Hugo, Markdown, and GitHub Pages gave me a streamlined process that looks decent for posting.
