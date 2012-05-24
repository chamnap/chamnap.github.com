---
layout: post
title: "Meta-programming in Ruby and JavaScript"
date: 2010-02-19 17:50
comments: true
categories: [ruby, javascript]
---

Recently, I have been working with writing a ruby gem, Yoolk API Gem. What is really interesting for me is I do some meta programming and object-oriented programming in Ruby which I have never experienced before. A few month later, there is a requirement that my team needs to write in JavaScript, but I don't want to touch JavaScript really much. Therefore, my team member took over this task. Whenever I write code in Ruby, I just try to think how to do it in JavaScript as well. Several things that came up to my mind with some from my team member:

- Defer class from a variable

{% codeblock lang:javascript %}
    var klass = "Person";
    p = new window[klass]; //class without namespace
    p = new yoolk[klass]; //class with namespace
{% endcodeblock %}

{% codeblock lang:ruby %}
    klass = "Person"
    p = Object.const_get(klass).new #class without namespace
    p = Yoolk.const_get(klass).new #class with namespace
{% endcodeblock %}

- Access class method from instance object

{% codeblock lang:javascript %}
    var p = new Person();
    p.constructor.getCount();
{% endcodeblock %}

{% codeblock lang:ruby %}
    p = Person.new
    p.class.count
{% endcodeblock %}

- Define method of an object

{% codeblock lang:javascript %}
    var p = new Person();
    p.hello = function() {
      alert('hello');
    };
{% endcodeblock %}

{% codeblock lang:ruby %}
    p = Person.new
    def p.hello
      puts "hello"
    end
{% endcodeblock %}

- Define class methods

{% codeblock lang:javascript %}
    Person.hello = function() {
      alert('hello');
    };
{% endcodeblock %}

{% codeblock lang:ruby %}
    def Person.hello
      puts "hello"
    end
{% endcodeblock %}
