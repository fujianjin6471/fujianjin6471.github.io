---
layout: post
title:  "An example of when should setNeedsDisplay be called"

---
`setNeedsDisplay` is meant to ask the system to call `drawRect:`, and there are great answers on StackOverflow like [this](http://stackoverflow.com/questions/14506968/setneedslayout-and-setneedsdisplay). But an [example](https://github.com/fujianjin6471/SetNeedsDisplay) may be helpful.

Code in `drawRect:` just draw a line between two points.

With `setNeedsDisplay`, the effect is 

![image]({{ site.url }}/images/2015-6-11/SetNeedsDisplay.gif)

Without `setNeedsDisplay`, the effect is 

![image]({{ site.url }}/images/2015-6-11/WithoutSetNeedsDisplay.gif)
