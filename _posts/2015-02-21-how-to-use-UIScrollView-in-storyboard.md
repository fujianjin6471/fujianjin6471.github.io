---
layout: post
title:  "How to use UIScrollView in storyboard"

---

Let's make it from scratch. Choose a **Single View Application** template, name it anything as you like, in my case, **UIScrollViewAndUIPageControl** is used.

Drag a **Scroll View** on top of View Controller\'s default **View**.

Set constraints for **Scroll View** using the second icon at bottom right corner (assuming AutoLayout is allowed) as below:

![image]({{ site.url }}/images/2015-2-21/ConstraintsForScrollView.png)

As you see, **Scroll View** is the same size as **View**. Shouldn\'t **Scroll View** be bigger than **View**? In fact, the size of **Scroll View** and the size of its content are two different concepts. You can view **Scroll View** as a window, the scope you can see through it is often greater than itself. More on this later.

Drag a View on top of **Scroll View**. I prefer to rename it **contentView**, the name just suggests its usage. 

Next step is to set constraints for **contentView**. But this time, it's not that straightforward. 

Firstly, add four constraints as we do for **Scroll View**.

![image]({{ site.url }}/images/2015-2-21/ConstraintsForContentView.png)

What happened? Xcode gave us two warnings. 

![image]({{ site.url }}/images/2015-2-21/WarningsAfterFourContentConstraints.png)

I think information lies in **Document Outline** gives better prompts than in **Navigator**, so let's go there to have a look.

![image]({{ site.url }}/images/2015-2-21/WarningsInDocumentOutline.png)

The four constraints we added for **contentView** just means **contentView** (the view we added manually) has the same size with **Scroll View**\'s contentSize (a property of UIScrollView defined in UIKit). Actually we haven\'t specified **Scroll View**\'s contentSize yet, this is the reason why the first two warnings came out. The default size of contentSize is CGSizeZero, this explains the third warnings in **Document Outline**. 

In order to fix the problem, we should add more constraints to make the scrollable content width and height clear. One way is to specify the **contentView**\'s width and height. 

Assume that I want to scroll horizontally and the content is three times of the screen. So the **contentView** should be equal height to screen and three times width of screen.

**Ctrl-drag** from **contentView** to **Scroll View** in **Document Outline** and choose **Equal Heights**.

![image]({{ site.url }}/images/2015-2-21/EqualHeights.png)

Next, **Equal Widths**. 

![image]({{ site.url }}/images/2015-2-21/EqualWidths.png)

We will set **contentView**\'s width three times of the screen, but not now.

Now let\'s decorate **contentView** with some images for final effect. Drag three **Image View**s onto the contentView and set constraints for them in turn.

![image]({{ site.url }}/images/2015-2-21/ThreeImages.png)

Set constraints for them. Take the leftmost **Image View** for example.

![image]({{ site.url }}/images/2015-2-21/Leftmost3Edges.png)

Then **ctrl-drag** from **First** (Image View) to **contentView** in **Document Outline** and choose **Equal Widths**.

![image]({{ site.url }}/images/2015-2-21/LeftmostWidth.png)

But this time, equal isn\'t what we really want. Let\'s open **Size inspector** of **First** (Image View) and double-click **Equal Width** constraint. 

![image]({{ site.url }}/images/2015-2-21/DoubleClickEqualWidth.png)

![image]({{ site.url }}/images/2015-2-21/AfterDoubleClick.png)

Adjust it like below:

![image]({{ site.url }}/images/2015-2-21/AfterAdjustment.png)

The rule underneath the hood is:

*attribute1 == multiplier × attribute2 + constant* ([NSLayoutConstraint Class Reference](https://developer.apple.com/library/ios/documentation/AppKit/Reference/NSLayoutConstraint_Class/index.html#//apple_ref/doc/uid/TP40010628))

So what we set means the first **Image View\'s width** equals one third of **contentView\'s width**. Then do the same for the next two **Image View**s.

One last step, set **contentView**\'s width to be three times of its superview (**Scroll View**).

![image]({{ site.url }}/images/2015-2-21/ThreeTimesWidth.png)

You can only one image now, that is the reason why we put the step as the last one.

Now run the program, the effect is just what you want.

![image]({{ site.url }}/images/2015-2-21/ScrollEffect.gif)

The demo project is [here](https://github.com/fujianjin6471/UIScrollViewInStoryboard).
