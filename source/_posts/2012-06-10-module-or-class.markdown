---
layout: post
title: "Module or Class?"
date: 2012-06-10 23:15
comments: true
categories: [ruby, solid, rails]
---
Just see the title, you would probably could answer this question very well. A module cannot be instantiated, is used to mixin, while a class can be instantiated, and so on. However, this blog post is not a tutorial at all. There is something else you could learn from it.

Last week, I paired with my boss, @jensendarren. He asked me a question about my code which I have never thought before. Why don't you make `ListingConverter` as a module and use it as a `mixin` to the `Listing` class? Actually, he asked me the right thing because sometimes I write a module, some other times I write a class and using `delegate`. Here is my code.

{% codeblock lang:ruby %}
class Listing < ActiveRecord::Base
  delegate :to_solr, :to => converter
  
  def converter
    ListingConverter.new(self)
  end
end
{% endcodeblock %}

He probably wants something like this.

{% codeblock lang:ruby %}
class Listing < ActiveRecord::Base
  include ListingConverter
end

module ListingConverter
  def to_solr
  end
end
{% endcodeblock %}

At that time, I couldn't answer him very well. I have read [Rails Anti-Pattern book](http://www.amazon.com/Rails-AntiPatterns-Refactoring-Addison-Wesley-Professional/dp/0321604814) some months ago and I followed that book because it's very convincing to me.
To be honest I never thought about using as a `module` or a `class`. I just understand that the author is trying to break responsibilities into multiple classes, [Single Responsibilities Principle (SRP)](http://en.wikipedia.org/wiki/Single_responsibility_principle). Each class should have very specific use case and few reasons to change.

Thinking about SRP reminds me about blog post, [SOLID Design Principles](http://blog.rubybestpractices.com/posts/gregory/055-issue-23-solid-design.html) from [Gregory Brown](http://blog.rubybestpractices.com/about/gregory.html) which I read it some months ago as well. For me, it's an excellent blog post because it changes me quite a lot. I would recommend you go through it, at least **SRP**. My response will take it as a reference.

Actually, these two above code achieve the same result, but there is case where we should use one rather than the other. In this case, I would say a `class` wins over a `module` in terms of efficiency.

My `ListingConverter` class contains only 1 public method, `#to_solr` and almost 20 private methods. It is responsible for converting into solr json format. 

- If `ListingConverter` is a module, `Listing` would contain unneccessary methods,  and if we have 50 mixined into, the `Listing` instance would become bigger and bigger objects, 200 methods. Imagine this case, what if each module has name collision? Then, it might be difficult to track down the case and find out what's wrong in those 50 mixins. Usually, a `Listing` instance would not use `to_solr` method most of the time, but those additional methods from each module are always there which is not optimal at all. The thing will be worse when we load 5000 `Listing` instance at a time to do reporting for example because the memory would go up steadily.

- Make `ListingConverter` as a class is more about **Single Responsibility**. This class just does only one thing, convert `activerecord` into `solr`. It would be better to treat `Listing` class like a big entity contains many small entities. It should forward the message requested to its contained objects. Finally, use `delegate` from `Rails` to make interaction a bit easier. What [delegate](https://github.com/rails/rails/blob/8ba491acc31bf08cf63a83ea0a3c314c52cd020f/activesupport/lib/active_support/core_ext/module/delegation.rb#L104) does is that it defines methods that are passed in and forwards those methods to the `:to` object. You could do mannually as below.

{% codeblock lang:ruby %}
def to_solr
  ListingConverter.new(self).to_solr
end
{% endcodeblock %}

It only instantiates `ListingConverter` only when we call `#to_solr` method which is more efficient. At the end of the day, the `Listing` class contains only 1 additional public method.
