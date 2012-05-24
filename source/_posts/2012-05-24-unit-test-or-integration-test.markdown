---
layout: post
title: "Unit Test or Integration Test?"
date: 2012-05-24 00:39
comments: true
categories: [rspec, rails]
---
I were often asked when to do unit test or integration test, so I decided to write this blog post to clarify:

##Unit Test

- By definition

    - it doesn't talk to database
    - it doesn't communicate across the network
    - it doesn't touch the file system
- Test a single module in isolation. We often use `stub` or `mock` its dependencies. When its tests fail, we know exactly because of itself.
- A `stub` object is a `fake object` and not part of the test. It can be replaced by any other objects.
- A `mock` object is also a `fake object`. A `mock` object contains behavior and it is part of what we want to test (collaborator).

{% codeblock lang:ruby %}
  describe Statement do
    it "logs a message on generate()" do
      customer = stub('customer')
      customer.stub(:name).and_return('Aslak')

      logger   = mock('logger')
      statement = Statement.new(customer, logger)

      logger.should_receive(:log).with(/Statement generated for Aslak/)

      statement.generate
    end
  end
{% endcodeblock %}

- The above test would fail if the `logger` doesn't call `#log` with the specified parameter. customer is simply a fake object with no behavior in the test, and it cannot make the test fail. Remember this test is about the logging a message.
- Kind of `White-box testing`. It tests internal workings of an application. You dictate the software that it should do this and do that.
- Generally, `private methods` are already tested indirectly by public methods. You should not test it explicitly. However, if you feel that private method is crucious to make your class work correctly, consider give it a better name and promote it as public method.
- Unit tests alone are not enough to make sure the application work correctly.
- Much faster than integration test.
- In Rails, there are functional tests: `model spec`, `controller spec` (it touches database).

##Integration Test

- It tests interaction between components to make sure these components work nicely with each other.
- Kind of `Acceptance Testing`, it focuses on what the user see and how the user interacts with the system. It can be called `Black-box testing` where we don't care how it is done. We care only the outcome.
- Much slower than unit test.
- In Rails, it would be request spec and capybara.

{% codeblock lang:ruby %}
  describe "POST /tasks" do
    it "creates task" do
      visit tasks_path
      fill_in "Name", :with => "washing clothes"
      click_button "Add"

      page.should have_content("Successfully added task.")
      page.should have_content("washing clothes")
    end
  end
{% endcodeblock %}

- The above test send the actual request to `/tasks`, fill out the form, and submit. It expects the content we filled out to be display in the page. If it's a unit test (in this case, functional test aka. controller spec), we would test it differently. We won't send the actual request, and we would assert it should receive `#save` on a `@task` object.
- It's better than unit test because it mimics real user behaviors and it tests the entire stack of the application.
- Should we still write controller spec? The answer is `yes` for sad path and leave the happy path in the integration tests. Doing this make your tests a bit faster. [Check this blog](http://solnic.eu/2012/02/02/yes-you-should-write-controller-tests.html)

##Where to get started?

- Should we write unit test first or integration test first? If you haven't watched [this episode](http://railscasts.com/episodes/275-how-i-test) from `RailsCasts`, watch it. He followed the outside-in development, starting from request spec, controller spec, and model spec.
