---
layout: post
title: "active_record_uuid 0.1.0 is released"
date: 2012-06-20 19:34
comments: true
categories: [gem, ruby, uuid]
---

For the past two weeks, I have been working on the [active_record_uuid gem](https://rubygems.org/gems/active_record_uuid). It's a nice experience because I have learnt quite a lot even though it's a small gem. This gem provides `uuid` support to active_record model. It allows you to store `uuid` as `binary`, `base64`, `hexdigest`, and `string`. Actually, this gem is well-tested (55 examples). Fork me or create issue on GitHub if you see a bug.

The good thing about this gem is that you query the data by using a plain `uuid` string while you store as `binary`, `base64`, `hexdigest`, or `string`.

{% codeblock lang:ruby %}
post = PostBinary.create(:text => "Binary uuid1")
# INSERT INTO `post_binaries` (`created_at`, `text`, `updated_at`, `uuid`) VALUES ('2012-06-20 17:32:47', 'Binary uuid1', '2012-06-20 17:32:47', x'4748f690bac311e18e440026b90faf3c')

post.uuid # "4748f690-bac3-11e1-8e44-0026b90faf3c"

# it works as usual for finding records
PostBinary.find_by_uuid(post.uuid)
PostBinary.where(:uuid => post.uuid)
PostBinary.find(post)
PostBinary.find(post.uuid)
PostBinary.find([post.uuid])
post.comments.create(:text => "Comment 1")

# access the value that stored in db
post.reload
post.attributes_before_type_cast["uuid"]["value"]
{% endcodeblock %}

Be sure to check out other options to use at [https://github.com/chamnap/active_record_uuid/blob/master/README.md](https://github.com/chamnap/active_record_uuid/blob/master/README.md).
