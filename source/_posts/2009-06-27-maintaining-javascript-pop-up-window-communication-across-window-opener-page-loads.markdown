---
layout: post
title: "Maintaining Javascript Pop-Up Window Communication Across Window Opener Page Loads"
date: 2009-06-27 18:05
comments: true
categories: [javascript]
---
I came across a blog post that talks how to maintain a reference to javascript popup window while the parent window has been navigated away. This scenario doesn't want to reload the child popup window. I just quoted out from 1 Pixel Out. There is a really nick trick.

In the main window:
{% codeblock lang:javascript %}
var popupWin = null;
  
function openPopup() {
   var url = "popup.htm";
   popupWin = open( "", "popupWin", "width=500,height=400" );
   if( !popupWin || popupWin.closed || !popupWin.doSomething ) {
      popupWin = window.open( url, "popupWin", "width=500,height=400" );
   } else {
      popupWin.focus();
   }
}

function doSomething() {
   openPopup();
   popupWin.doSomething();
}
{% endcodeblock %}

In the popup:
{% codeblock lang:javascript %}
self.focus();
  
function doSomething() {
   alert("I'm doing something");
}
{% endcodeblock %}


[http://www.bennadel.com/blog/89-Maintaining-Javascript-Pop-Up-Window-Communication-Across-Window-Opener-Page-Loads.htm](http://www.bennadel.com/blog/89-Maintaining-Javascript-Pop-Up-Window-Communication-Across-Window-Opener-Page-Loads.htm),
[http://www.1pixelout.net/2005/04/19/cross-window-javascript-communication/](http://www.1pixelout.net/2005/04/19/cross-window-javascript-communication/),
[http://www.1pixelout.net/2006/12/15/cross-window-javascript-communication-20/](http://www.1pixelout.net/2006/12/15/cross-window-javascript-communication-20/),
[http://www.1pixelout.net/wp-content/downloads/popups20.zip](http://www.1pixelout.net/wp-content/downloads/popups20.zip)
