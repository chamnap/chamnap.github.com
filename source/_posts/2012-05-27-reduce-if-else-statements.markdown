---
layout: post
title: "Reduce If/Else Statements"
date: 2012-05-27 23:35
comments: true
categories: [ruby, rails, refactoring]
---
In the new project, we have built recently at `Yoolk`. I really enjoyed a lot of refactoring the app.

There are things which always bother me a lot is the `if/else` statements. I see them all the time. In my views, `if/else` should be used at the low level of coding. We should not use them too much because it doesn't make the code readable.

I remembered [vorleak](http://vorleakchy.com/), my coworker, and I are moderators in a study group long time ago about [Principles of Refacoring](http://chamnap.github.com/blog/2009/09/05/principles-in-refactoring/). Two principles that really inspires me quite alot: **less code == less bugs** and **write code for human, not for machine**.

It looks simple to experienced `Rails` developers, but it's useful for novice people. Here are some tips to reduce `if/else` statements:

- Use `find_or_initialize_by`, `find_or_create_by` method

As the method name, it's a cleaner way to get/create objects without `if/else`.

{% codeblock lang:ruby %}
# A shorter version
user = User.find_or_initialize_by(params[:user_name])
# A longer version with if statement
user = User.find_by_user_name(params[:user_name])
user = User.new(:user_name => params[:user_name]) if user.nil?
{% endcodeblock %}

Be sure to check more about these methods if you didn't know on [http://api.rubyonrails.org/classes/ActiveRecord/Base.html](http://api.rubyonrails.org/classes/ActiveRecord/Base.html) in the **Dynamic attribute-based finders** section.

- Use `try` for possible nil object

Invoke `try` for object that could be nil. It's more convienient than doing a check by yourself.

{% codeblock lang:ruby %}
#Without try
@person ? @person.name : nil
#With try
@person.try(:name)
{% endcodeblock %}

However, don't confuse with the below situation.

{% codeblock lang:ruby %}
#Don't invoke try with non-existed methods
@person.try(:abcde)
#Call respond_to instead
@person.respond_to?(:abcde) ? t.abcde : ""
{% endcodeblock %}

`Try` also be called with block as well so that you can call multiple methods in a scope of try.

{% codeblock lang:ruby %}
@person.try { |p| "#{p.first_name} #{p.last_name}" }
{% endcodeblock %}

Check this document, [http://api.rubyonrails.org/classes/Object.html#method-i-try](http://api.rubyonrails.org/classes/Object.html#method-i-try) as well.

- Use `||` operator + `presence` method

The `||` is a common idiom in Ruby. However, it doesn't work well if the first operand is empty string. The `presence` method will return nil instead of "" if the object is `blank?``, otherwise it return the actual object back.

{% codeblock lang:ruby %}
host = config[:host].presence || 'localhost'
{% endcodeblock %}

- Use default value

Use default value so that you don't else clause.

{% codeblock lang:ruby %}
# should do this way, it's more readable.
subscription = 'normal'
subscription = 'premium' if condition
{% endcodeblock %}

- Keep the if/else logic in fewer places

Wrap them in a function and reuse it where it is possible. Sometimes, it 's hard to extract it into function because they are slightly different. Try to write in general context, think about its behavior, and make it fit.

If you feel you are doing too much `if/else`, go back one step why you are doing that way. Try to use the correct objects that fit to your scenarios.

Here is my coworker's version generating the last 12 months stats. He manipulates the `string` object.
{% codeblock lang:ruby %}
def last_twelve_months
  default_value = {"views"=>0, "website_clicks"=>0, "email_clicks"=>0, "billboard_clicks"=>0, "sponsor_views" =>0}
  current_month = Date.yesterday.month
  twelve_months = {}
  (1..12).each do |i|
    month = current_month - i
    if month > 0
      yearmonth = Date.yesterday.year.to_s + ("%02d" % month)
    elsif month == 0
      yearmonth = Date.yesterday.year.to_s + ("%02d" % current_month)
    else
      month = month + 13
      yearmonth = (Date.yesterday.year - 1).to_s + ("%02d" % month)
    end
    twelve_months[yearmonth] = self.yearmonths[yearmonth].blank? ? default_value : self.yearmonths[yearmonth]
  end

  Hash[twelve_months.sort_by {|k,v| k.to_i}.last(12)]
end
{% endcodeblock %}

{% codeblock lang:ruby %}
def resolve_language
  language_uuid = cookies["language_uuid"]
  @active_language = if language_uuid and @portal.languages.collect(&:uuid).include?(language_uuid)
    @portal.languages.find(language_uuid)
  else
    @portal.language
  end

  I18n.locale = @active_language.two_code
  @inactive_languages = @portal.languages.select {|language| language.uuid != @active_language.uuid}
end
{% endcodeblock %}

Here, it's my version, much shorter and less code.
{% codeblock lang:ruby %}
def last_year
  return @last_year if @last_year.present?
  
  defaults = Hash.new
  12.downto(1).each do |i|
    date = i.months.ago
    key = "#{date.year}%02d" % "#{date.month}"
    defaults[key] = { "views" => 0, "website_clicks" => 0, "email_clicks" => 0, "billboard_clicks" => 0 }
  end
  existings = yearmonths.select { |k, v| defaults.key?(k) }
  @last_year = defaults.merge(existings)
end
{% endcodeblock %}

{% codeblock lang:ruby %}
def resolve_language
  language_uuid = cookies["language_uuid"]
  @language     = @portal.has_language?(language_uuid)
  @language   ||= @portal.default_language
  
  I18n.locale = @language.two_code
  @inactive_languages = @portal.languages_except_by(@language.uuid)
end
{% endcodeblock %}

- `Polymorphism` + `Factory pattern`

I recommend you read the book from **Martin Fowler, Improving the Design of Existing Code**.
