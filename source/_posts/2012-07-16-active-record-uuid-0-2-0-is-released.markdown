---
layout: post
title: "active_record_uuid 0.2.0 is released"
date: 2012-07-16 00:20
comments: true
categories: [gem, ruby, uuid]
---

After releasing version `0.1.0`, I get the feeling I need to add one more feature to this gem, the ability to configure in global and per model. Actually, it's a small change, but I haven't had enough time to do it. Now, it's complete.

{% codeblock lang:ruby %}
  ActiveRecordUuid.configure do
    column      :uuid
    primary_key true
    association false
    store_as    :binary
  end
  
  class People < ActiveRecord::Base
    has_uuid :association => true
    has_many :comments
  end
{% endcodeblock %}

In the example above, it will merge the global configuration options with the passed in options in `has_uuid`.

There's a config generator that generates the default configuration file into config/initializers directory. Run the following generator command, then edit the generated file.

{% codeblock lang:ruby %}
  $ rails g active_record_uuid:config
{% endcodeblock %}
