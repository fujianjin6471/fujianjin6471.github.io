---
layout: post
title:  "How to add UIPageControl to a scroll view in storyboard"

---
The starting point is a view controller with a scroll view, you can access it [here](https://github.com/fujianjin6471/DemosForBlog/tree/master/UIScrollViewInStoryboard).

At the very beginning, drag a **Page Control** on top the root view, the hierarchy in the Document Outline should look like this:

![image]({{ site.url }}/images/2015-4-23/HierarchyInDO.png)

Then, set the constraints of the page control as below:

![image]({{ site.url }}/images/2015-4-23/ConstraintsOfPageControl.png)

Next, add two properties to your **ViewController** class and connect them to scroll view and page control respectively.

{% highlight swift %}
@IBOutlet weak var scrollView: UIScrollView!
@IBOutlet weak var pageControl: UIPageControl!
{% endhighlight %}

Run the app.

![image]({{ site.url }}/images/2015-4-23/InitialEffect.gif)

You will see when you tap at the right part of the page control, the shiny dot move towards the right, vice versa, but nothing happen to the scroll view. And when you sweep the scroll view, the shiny dot of page control keeps quiet. These are the two tasks we are going to finish.

Firstly, let's make scroll view respond to taps on page control.

Ctrl-drag from **Page Control** to the code editor to make an action:

![image]({{ site.url }}/images/2015-4-23/ActionOfPageControl.png)

code here:

{% highlight swift %}
@IBAction func pageTurn(sender: UIPageControl) {
    let viewSize = scrollView.frame.size
    let rect = CGRectMake(CGFloat(sender.currentPage) * viewSize.width, 0, viewSize.width, viewSize.height)
    scrollView.scrollRectToVisible(rect, animated: true)
}
{% endhighlight %}

The code inside the action is very straightforward. Every time when we tap the page control, this action is triggered. Then we ask the scroll view to slide to the part which correspond to the current page (the shiny dot, remember?) of the page control.

Run the app.

![image]({{ site.url }}/images/2015-4-23/PageTurnEffect.gif)

This time when you tap at the right part of the page control, the scroll view moves!

Then comes the reverse part. When we sweep the scroll view, the view controller will be notified and asks the page control to synchronize with the scroll view's position (in fact, its contentView's position). How to implement this? Delegate is a good choice. 

Make **Scroll View** be **View Controller**'s delegate in the IB and add a method in the class

{% highlight swift %}
func scrollViewDidEndDecelerating(scrollView: UIScrollView) {
    let offset = scrollView.contentOffset
    let bounds = scrollView.frame
    pageControl.currentPage = Int(offset.x / bounds.size.width)
}
{% endhighlight %}
    

*- scrollViewDidEndDecelerating:* will be called when the scroll view has ended decelerating the scrolling movement. Inside of it, we get the contentView's offset to calculate the corresponding page of page control and set the latter to be **Page Control**'s current page.

Run the app.

![image]({{ site.url }}/images/2015-4-23/TwoEffects.gif)

When you sweep the scroll view, the page control synchronizes with it, great!

Play with it for a moment, you will notice there's improvement being able to take place.

Just add another method like this and call it in *viewDidLoad*:

{% highlight swift %}
func someImprovement() {
    scrollView.pagingEnabled = true
    scrollView.bounces = false
    scrollView.showsHorizontalScrollIndicator = false
}
{% endhighlight %}

Run the app for the last time.

![image]({{ site.url }}/images/2015-4-23/FinalEffect.gif)

That's just what you want, right?

The demo project is [here](https://github.com/fujianjin6471/DemosForBlog/tree/master/ScrollViewAndPageControl).