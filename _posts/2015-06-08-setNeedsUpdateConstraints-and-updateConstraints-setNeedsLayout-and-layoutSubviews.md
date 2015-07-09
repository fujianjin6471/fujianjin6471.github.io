---
layout: post
title:  "setNeedsUpdateConstraints and updateConstraints, setNeedsLayout and layoutSubviews"

---
This topic is under *Auto Layout*. 

When should you use `setNeedsUpdateConstraints`? One aspect is when you want to change the frame of a view directly. As we know, under *Auto Layout*, just changing the frame is extremely unreliable. It may take effect for some time, but as long as `setNeedsLayout` or another method which has the same effect is called at some point you don't expect, all relevant changes you have made to frame will disappear. 

`setNeedsUpdateConstraints` gives you the power to make constraints change after frame change. What you need to do is to override `updateConstraints` of the root view. In `updateConstraints`, set constraints following the frame of the view. After you change the frame of a view, call `rootView.setNeedsUpdateConstraints()`.

When should you call a view's `setNeedsLayout`? When you want to layout the view's subviews. If you don't override the view's `layoutSubviews`, the default behavior is to make its subviews follow the relevant constraints.

The demo project is [here](https://github.com/fujianjin6471/DemosForBlog/tree/master/SetNeedsSomething), let's play with it.

![image]({{ site.url }}/images/2015-6-8/WhetherBack.gif)

Press **Button1**

{% highlight swift %}

@IBAction func setFrameOnly(sender: UIButton) {
    orangeSquare.frame.origin.x += 100
}

{% endhighlight %}


The orange square was moved, but at this very moment, its left constraint hasn't been changed, so if I press **Button3**

{% highlight swift %}

@IBAction func setNeedsLayout(sender: UIButton) {
    rootView.setNeedsLayout()
}
{% endhighlight %}

The orange square went back.

Press **Button2** 

{% highlight swift %}

@IBAction func setFrameWithUpdatingConstraints(sender: UIButton) {
    orangeSquare.frame.origin.x += 100
    rootView.setNeedsUpdateConstraints()
}
{% endhighlight %}

This time, not only did we change the orange square's position, but also called `setNeedsUpdateConstraints`. So after the position change of the orange square, `updateConstraints` was called.

{% highlight swift %}

override func updateConstraints() {
    leftConstraint.constant = orangeSquare.frame.origin.x
    super.updateConstraints()
}
{% endhighlight %}

Now, the constraint was also changed and coincided with the frame again. Press **Button3**, nothing happened, that's just what we want.

