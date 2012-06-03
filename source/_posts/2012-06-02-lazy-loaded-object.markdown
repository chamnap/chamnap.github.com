---
layout: post
title: "Lazy-loaded object"
date: 2012-06-02 23:26
comments: true
categories: [rails, ruby, gem]
---
Last week, we decided to remove `sunspot` gem from this new version App. Therefore, I rolled out a simple and small `solr` library.

I went through the [ActiveRelation Walkthrough](http://railscasts.com/episodes/239-activerecord-relation-walkthrough) episode from RailsCasts long time ago, but now I have a chance to do something similar. I want the search response object lazy to load activerecord objects. I don't want to call `#results` method the same as `sunspot` does. Actually, it's a nice trick and simple to do it.

{% codeblock lang:ruby %}
require 'json'

class Solr::Response
	attr_reader :response

  def initialize(clazz, raw_response)
  	@clazz    = clazz
  	@response = JSON.parse(raw_response)
  end

  def docs
  	response["response"]["docs"]
  end
  
  def total
  	response["response"]["numFound"].to_i
  end
  
  def inspect
    to_a.inspect
  end

  def to_a
    return @records if loaded?

  	uuids    = docs.collect { |doc| doc['uuid'] }
    @records = load_objects(@clazz, uuids)
  end

  def loaded?
    defined?(@records)
  end
  
  private
  def load_objects(clazz, uuids)
    return [] if uuids.blank?

    records = clazz.unscoped.where(:uuid => uuids).to_a
    return [] if records.blank?

    uuids.inject([]) do |result, uuid|
      result << records.find { |record| record.uuid == uuid }
    end
  end

  def method_missing(*args, &block)
    to_a.send(*args, &block)
  end
end

{% endcodeblock %}

After sending the request to Solr, it will initialize the response object by passing solr response and the class to retreive the results.

The trick is to overwrite `#inspect` method so that when in the console, you will see the objects back. `#to_a` method is responsible loading the objects. 

The question is that when to load? I want the caller be able to iterate through the collection by using standard Ruby enumerable methods such as `each`, `inject`, ...... My solution is to overwrite `method_missing` by sending any undefined methods to `#to_a` method.

{% codeblock lang:ruby %}
@listings = Solr::Listing.text_search(params[:q], .....)
@listings.class     #Solr::Response
@listings.total     #1367, total founds from solr
@listings.count     #20, +count+ method on array
@listings[0]        #Listing class
@listings.collect(&:name)
{% endcodeblock %}

Everything works fine except when it renders the view. It shows this error `[....] is not an ActiveModel-compatible object that returns a valid partial path.`

{% codeblock lang:ruby %}
<%= render @listings %>
{% endcodeblock %}

After digging through google, it simply means it doesn't know the partial path to render because it is my new object. Therefore, I just one more method in `Solr::Response` class. It's pretty simple, actually. `ActiveModel` does the same thing, [http://apidock.com/rails/v3.2.3/ActiveModel/Conversion/to_partial_path](http://apidock.com/rails/v3.2.3/ActiveModel/Conversion/to_partial_path).

{% codeblock lang:ruby %}
  def to_partial_path
    @clazz._to_partial_path
  end
{% endcodeblock %}
