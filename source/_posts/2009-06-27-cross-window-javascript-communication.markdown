---
layout: post
title: "Cross-window Javascript communication"
date: 2009-06-27 10:00
comments: true
categories: [javascript]
---
It reminds me about uploading via iframe that I did a year ago when my colleague asked me to help with login openid in a popup window. It's a similar story with this problem. Actually, login with openid could not place in a iframe because you could the code that prevents this.
view plainprint?

{% codeblock lang:javascript %}
if(top == self) { document.write(""); } else  { top.location.href = "http://www.yahoo.com"; }
{% endcodeblock %}

Now, let's see a quick summary on this basic communication.

Communication from parent to child window, you need to a reference of the child window so that can call any function in the child window.

{% codeblock lang:javascript %}
    // Create a new popup window  
    var popupWin = window.open(url, "popupWin");  
      
    // To call functions defined in the popup:  
    popupWin.doSomething();  
{% endcodeblock %}

Communication from child to parent window, you need to use this way:

{% codeblock lang:javascript %}
    window.opener.doSomethingOnParent();  
{% endcodeblock %}

Here is the problem, the parent window needs to know when the uploading (in iframe) or logging in (in popup window) is done. The only way that the parent window can notified by the child window after finish processing. Usually, for uploading and logging in with openid, the action in your controller would render a view back. The trick is here on the onload of the body, you could notify the parent window.

{% codeblock lang:ruby %}
    def login
      @status = "something"
    end
{% endcodeblock %}
    
{% codeblock lang:html %}
    <html><head></head>
    <body onload="window.opener.handleOpenIDResponse('" + @status + "');window.close();">
    </body>
    </html>
{% endcodeblock %}

That would solve the problem, and you could send any information back through your view.
