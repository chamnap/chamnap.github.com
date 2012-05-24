---
layout: post
title: "Remove focus options on your spec examples"
date: 2012-05-17 00:39
comments: true
categories: [rspec, rails]
---
When using rspec in my rails projects, I often add `:focus => true` on `describe/context/it block` and let `guard` running the tests on those new specs. I need to remove it from various places to run the test fully. It's a bit annoying, actually. Therefore, I decide to write a ruby script just to do this thing:

{% codeblock lang:ruby %}
#!/usr/bin/env ruby
founds = `grep -n -r ":focus => true" spec/`.split("\n")

#path;line_number;matched_content
founds.each do |string|
  array = string.split(":")
  path = array[0]
  line_number = array[1]
  matched_line = array[2..-1].join(':')
  
  if matched_line.match(/^\s*(describe|context|it).+(:focus\s*?=>\s*?true do$)/)
    puts "\e[0;32m#{string}\e[0m"
    
    system "sed -i '#{line_number} s/,\s*:focus\s*=>\s*true//' #{path}"
  end
end
{% endcodeblock %}

What this code does is checking all the spec directory to remove them on every `describe/context/it block`. You would see I use very strict regular expression just to ensure I'm doing the write thing. I placed this script under script directory of my rails app. You may write this code better, but this is how i do it. I also create a rake script and add to hg before commit hook.

{% codeblock lang:ruby %}
namespace :spec do
  desc "remove :focus => true in all specs"
  task :remove_focus do
    require_relative '../../script/remove_focus'
  end
end
{% endcodeblock %}

{% codeblock lang:ruby %}
[hooks]
precommit = script/remove_focus.rb
{% endcodeblock %}
