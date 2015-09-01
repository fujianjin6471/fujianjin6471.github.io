---
layout: post
title:  "Why does constructing an object with a metatype need a required initializer"

---
Look at a simple class like this:
    
{% highlight swift %}
class SomeClass {
    class func generate() -> SomeClass {
        return self()
    }
}
{% endhighlight %}

`self` is different in instance method and class method, you can see it through the auto completion of Xcode. In instance method, `self`'s type is `SomeClass` while in class method, it's type is `SomeClass.Type`.

The compiler complains **constructing an object of class type 'SomeClass' with a metatype value must use a 'required' initializer**

How to understand this? If I write a subclass like this:

{% highlight swift %}
class SubClass: SomeClass {
    let aNumber: Int
    init(aNumber: Int) {
        self.aNumber = aNumber
        super.init()
    }
}
{% endhighlight %}

It will inherit `SomeClass`'s class method `generate()` but won't inherit its default initializer `init()`. So `SubClass.generate()` actually calls `SubClass()`, but `SubClass` doesn't have an initializer `init()`! Of course this is what the compiler want to prevent.

Then let's move to a similar problem, it really took me some time to understand it.

{% highlight swift %}
class someClass {
}
var anotherClass = someClass.self
var anotherObject = anotherClass()
{% endhighlight %}

The compiler still complains **constructing an object of class type 'SomeClass' with a metatype value must use a 'required' initializer**. 

It's true that `anotherClass` is a **metatype value**, but what bad result will be caused?

Consider the case where we also have a subclass:

{% highlight swift %}
class SomeClass {
}

class Subclass : SomeClass {
}
{% endhighlight %}

If you store the class type in a variable:

{% highlight swift %}
var anotherClass = SomeClass.self
{% endhighlight %}

The variable `anotherClass` is of type `SomeClass.Type`.

You can later assign this variable a subclass:

{% highlight swift %}
anotherClass = SubClass.self
{% endhighlight %}

This is valid because `SubClass.Type` is a `SomeClass.Type`. At this point, `anotherClass()` would fail if the initializer is not implemented in the subclass. This is what the compiler is protecting against.

So, why does constructing an object of class type `SomeClass` with a metatype value must use a 'required' initializer?

Let's say `SubClass` is a subclass of `SomeClass`. Since `SubClass` is still a `SomeClass`, it's legal for `SubClass` to construct an object of class type `SomeClass` like `SomeClass` do but there's no guarantee that `SubClass` has implemented the initializer which is used by `SomeClass` to do so. And the guarantee is the required initializer.

Thanks to [Aaron Brager](http://stackoverflow.com/users/1445366/aaron-brager) who answered my [question](http://stackoverflow.com/questions/32163124/why-must-constructing-an-object-of-class-type-someclass-with-a-metatype-value) on SOF and [this question](http://stackoverflow.com/questions/28144475/implementing-nscopying-in-swift-with-subclasses) also helped a lot to understand this problem.
