---
layout: post
title:  "Ways to add a Tap Gesture Recognizer"

---
Say you are using a **UITextField**, you want to tap another place to hide the keyboard. This article provides two methods to implement the simple functionality.

First, let's realize it mainly through IB:

![image]({{ site.url }}/images/2015-4-4/DragATapGestureRecognizer.png)

![image]({{ site.url }}/images/2015-4-4/CtrlDragRecognizer.png)

Make it an action, add only one line of code as follow:

![image]({{ site.url }}/images/2015-4-4/OnlyOneLine.png)

All done, so easy, right?

![image]({{ site.url }}/images/2015-4-4/EffectOne.gif)

Then, let's realize it through pure code:

You just need to add two piece of code:

Add ![image]({{ site.url }}/images/2015-4-4/CreateGestureRecognizer.png) in *viewDidLoad:* 

and add a method ![image]({{ site.url }}/images/2015-4-4/HideKeyboard.png)

The effect is exactly the same as before. While there are of course other ways to hide keyboard, but that's not the point in this article.

IB or pure code, which one do you prefer?

The demo project is [here](https://github.com/fujianjin6471/GestureRecognizer).