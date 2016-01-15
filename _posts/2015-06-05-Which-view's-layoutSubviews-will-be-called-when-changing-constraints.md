---
layout: post
title:  "Which view's layoutSubviews will be called when changing constraints"

---
Under *Auto Layout*, we are told to forget the **frame** and keep **constraints** in mind. When we change a constraint and it doesn't coincide with the frame influenced by it, the `layoutSubviews` will be called. 

The documentation of `layoutSubviews` says:

> the default implementation uses any constraints you have set to determine the size and position of any subviews.

But, which view's `layoutSubviews` will be called? Let's have a look. (The demo project is [here](https://github.com/fujianjin6471/DemosForBlog/tree/master/WhichLayoutSubviews), it's necessary because what follows depend on the constraints set in the project)

First view is the root view of viewController, second view is the subview of first view, third view is the subview of second view.

Firstly, change the width constraint of third view:

{% highlight swift %}

thirdWidthConstraint.constant += 1
{% endhighlight %}

In the console, we see

* **second layoutSubviews**
* **third layoutSubviews**

Since this step will change the width of third view, so `layoutSubviews` of third view's super view (second view) will be called. More about this condition later.

Secondly, change the constraint between the bottom of second view and the bottom of third view:

{% highlight swift %}

secondThirdBottomConstraint.constant += 1
{% endhighlight %}

In the console, we see

* **first layoutSubviews**

Since this step will only change second view's height, so only `layoutSubviews` of second view's super view (first view) will be called.


Thirdly, change the constraint between the top of second view and the top of third view:

{% highlight swift %}

secondThirdTopConstraint.constant += 1
{% endhighlight %}

In the console, we see

* **first layoutSubviews**
* **second layoutSubviews**

This time, the height of second view and the relevant position between second and third view have changed, so both first and second view's `layoutSubviews` are called.

Lastly, change the constraint between the left of first view and the left of second view:

{% highlight swift %}

secondLeftConstraint.constant += 1
{% endhighlight %}

In the console, we see

* **first layoutSubviews**
* **second layoutSubviews**

This time, the relevant position between first and second view changes, along with third view's position in the screen, so both first and second view's `layoutSubviews` are called.

What can we conclude from above? **When we change a constraint, a view's `layoutSubviews` will be called if any of its subview whose frame is influenced by the constraint**.

However, there is still a doubt in the way, which is reflected by the first condition we considered before. When we change the width constraint of the third view, only the the third view's width is changed, so as we expect, only `layoutSubviews` of third view's super view (second view) will be called. But the third view's `layoutSubviews` is also called! I have done some more experiments, I'm not sure I get the answer, but I make a guess: **When we change a constraint and it influences a  view which doesn't have any subviews, both the view and its super view's `layoutSubviews` will be called**. If someone knows why, please tell me, thanks in advance.




