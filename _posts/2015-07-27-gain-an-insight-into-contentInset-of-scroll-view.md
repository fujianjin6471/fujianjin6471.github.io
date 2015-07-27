---
layout: post
title:  "Gain an insight into contentInset of scroll view"

---
The **contentInset** property bring some perplexity to developers new to scroll view. This article visualizes the effect of **contentInset** under different circumstances to help understand the concept.

Imagine a scroll view as a photo frame and its content view (maybe not an UIView object but just the range of content) a photo. If the photo is bigger than the frame, we can only see part of the photo at a moment. 

Say we have a scroll view of 100 point both in width and height. Now let's go see effect of **contentInset** under different circumstances one by one.

**Circumstance 1:**

{% highlight swift %}
scrollView.contentSize = CGSizeMake(100, 100)
scrollView.contentInset = UIEdgeInsets(top: 10, left: 20, bottom: 30, right: 40)
{% endhighlight %}

![image]({{ site.url }}/images/2015-7-27/1.png)

Positive component (top, left, bottom or right) of **contentInset** generates <u>a specified range stretching from the edge of content view</u>.  The positive component of **contentInset** is marked white and can be dragged into scroll view.

**Circumstance 2:**

{% highlight swift %}
scrollView.contentSize = CGSizeMake(160, 140)
{% endhighlight %}

![image]({{ site.url }}/images/2015-7-27/2.png)

If we don't specify **contentInset**, the default value is *UIEdgeInsetsZero*.

**Circumstance 3:**

{% highlight swift %}
scrollView.contentSize = CGSizeMake(160, 140)
scrollView.contentInset = UIEdgeInsets(top: 10, left: 10, bottom: 10, right: 10)
{% endhighlight %}

![image]({{ site.url }}/images/2015-7-27/3.png)

**Circumstance 4:**

{% highlight swift %}
scrollView.contentSize = CGSizeMake(100, 100)
scrollView.contentInset = UIEdgeInsets(top: -10, left: -20, bottom: -30, right: -40)
{% endhighlight %}

![image]({{ site.url }}/images/2015-7-27/4.png)

Negative component (top, left, bottom or right) of **contentInset** generates <u>a specified range shrink from the edge of content view</u>. The negative component of **contentInset** is marked gray and cannot be dragged into scroll view, indicating that some part of content view will be sheltered.

**Circumstance 5:**

{% highlight swift %}
scrollView.contentSize = CGSizeMake(150, 150)
scrollView.contentInset = UIEdgeInsets(top: -10, left: -10, bottom: -10, right: -10)
{% endhighlight %}

![image]({{ site.url }}/images/2015-7-27/5.png)

**Circumstance 6:**

{% highlight swift %}
scrollView.contentSize = CGSizeMake(150, 150)
scrollView.contentInset = UIEdgeInsets(top: 10, left: -10, bottom: -10, right: 10)
{% endhighlight %}

![image]({{ site.url }}/images/2015-7-27/6.png)

In summary, we can seperate the process of setting **contentInset** into two parts:

First, decorate the content view with something like a band:

![image]({{ site.url }}/images/2015-7-27/7.png)

If a component of **contentInset** is positive, then the relative side is legal, that is to say it can be dragged into scroll view. If a component of **contentInset** is negative, then the relative side is illegal, in other words, it cannot be dragged into scroll view.

Second, the top left corner of the rectangle inside the band overlaps the origin of scroll view .
