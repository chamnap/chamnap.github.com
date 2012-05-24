---
layout: post
title: "How useful is monit?"
date: 2011-10-19 16:09
comments: true
categories: [ubuntu, ruby, gems]
---
From here assuming that you know what monit is, but if you don’t, just go to see [Monit](http://mmonit.com/monit/). It’s basically a monitoring utility in unix system. You could monitor almost anything in your server. It’ll automatically restart the service if it stops and send you an email.

Generally, some people don't like it very much because it send too many emails when your server is in trouble, and those emails are annoying. A part of my job is the servers' maintainer and deployer. My job is to keep the servers fast and stable. By now, we have around 10 servers using Amazon Web Service. There are many services need to be run properly, otherwise my work’s inbox will contain some new emails. There are some ruby scripts as well need to run on the production server. The problem is those scripts are a bit heavy, it took more than 50% of the cpu to process, and if we more than 2 simultaneously, we’ll see some complaints from customers saying that "our websites are performing slowly. Why?"

Generally, we often lose our server because something is wrong on the server. We are really don't know why. It happens on one server after another. Until I decide to use monit to monitor many services as possible. It seems when the ubuntu server is heavily processing, the sshd service is down. We never know it was down. We just can’t remote our server. The solution is to reboot, but next few days it happens again. Usually, when something weird happens during the startup, the sshd service is never up. How can we remote to those servers? Clearly, there are no way if that service is down.

I see the answer when I add monit on all servers, and I run some scripts on the server as usual. The next morning came up, two emails from monit saying "sshd is down" and "sshd is up". (I didn't notice about it until the next few days.) That's good, isn't it? I don't have to reboot the server. When monit detects there is no such service, it simply restart that service. By now, I don't really understand why sshd is always down when the server is heavily busy. I tried to upgrade to the latest AWS ami already, but the problem still exists. Maybe the server is too overloaded. The solution is to expand more servers or refactor those ruby processes.

Be sure to monit sshd service to all your server.

{% codeblock lang:sh %}
check process sshd with pidfile /var/run/sshd.pid
    group system
    start program  "/etc/init.d/ssh start"
    stop program  "/etc/init.d/ssh stop"
    if failed port 1234 protocol ssh then restart
    if 5 restarts within 5 cycles then timeout
{% endcodeblock %}

You can checkout [God](https://github.com/mojombo/god) as well if you want nicer syntax and more readable code.
