---
layout: post
title: "Lesson learned from alias_method_chain"
date: 2012-07-04 22:36
comments: true
categories: 
---
A couple week ago, while I was developing the [active_record_uuid gem](http://rubygems.org/gems/active_record_uuid), I need to do extension to the existing methods in `active_record` library. The first one is to send `portal_uuid` instead of `portal_id` in the association methods such as `has_many`, .... The second one is about quoting `uuid` value and send it to `mysql`. I went to see the source code of `active_record` to find where the best place to do my job. After spending some times to understand the codebase, I find the place to overwrite, `ActiveRecord::Associations::ClassMethods` and `ActiveRecord::ConnectionAdapters::Quoting`. The first thing that comes to my head is to do `alias_method_chain` of these methods. I didn't think much, write the test and implement it.

{% codeblock lang:ruby %}
module ActiveRecord
  module Associations
    module ClassMethods
      def has_many_with_defaults(name, options = {}, &extension)
        options = uuid_assoc_options(:has_many, name, options)
        
        has_many_without_defaults(name, options, &extension)
      end
      alias_method_chain :has_many, :defaults
      
      ......
    end
  end
end
{% endcodeblock %}

Read that code again, I realized there are two things that my code could be changed in the future. Those are the module and method's name. If one day the `Rails` team decide to change the module name, then I would have to do another commit to fix that. In addition, method name is a bit longer and cumbersome as well.

I took one step back and think about it again. That module is simply included into `ActiveRecord::Base`. From here, it reminds about my ruby lesson. Ruby method lookup path is the reverse order of module inclusion. Why Ruby does that? Well, it is because it is designed to be extensible.

Therefore, if I define another module and include it very last, my method should be run before the original method. That's cool, isn't it? Calling the original version is even easier, call `#super` will does the job. It's awesome.

{% codeblock lang:ruby %}
module ActiveRecordUuid
  module AssociationMethods
    def has_many(name, options = {}, &extension)
      options = uuid_assoc_options(:has_many, name, options)
      super
    end
    
    ........
  end
end

ActiveRecord::Base.send(:extend, ActiveRecordUuid::AssociationMethods)
{% endcodeblock %}

The question arrives to my head. What is the use of `alias_method_chain`? What is it for? Well, in the above case, I wouldn't need it, but it's still useful to overwrite the methods which are defined originally in that class.

{% codeblock lang:ruby %}
class Person
  def hello
    "Hello"
  end
end

module MyModule
  def hello
    "hello from module"
  end
end
{% endcodeblock %}

There is no use if I `include` `MyModule` into `Person` class, because Ruby will run the method inside that class, then the method inside the module. In this case, only `alias_method_chain` could help.

It's hard to see any good guidelines to do extensions when developing the gem, but here it's a better way to do the job.
