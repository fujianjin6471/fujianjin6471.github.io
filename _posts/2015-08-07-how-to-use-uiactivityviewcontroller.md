---
layout: post
title:  "How to use UIActivityViewController"

---
`UIActivityViewController` provides convenient access to various services such as sending items via email or SMS, copying items to the pasteboard, posting content to social media sites. Let's make a demo to show how to use it.

For simplicity, just use a button to trigger the services of `UIActivityViewController`.

{% highlight swift %}
@IBAction func services(sender: AnyObject) {
    let activityViewController = UIActivityViewController(activityItems: ["firstItem","secondItem"], applicationActivities: nil)
    presentViewController(activityViewController, animated: true, completion: nil)
}
{% endhighlight %}

Run the app, tap the button

![image]({{ site.url }}/images/2015-8-7/ServicesForString.png)

Choose **Mail**, we can see the contents are just the two items we used to initialize the `UIActivityViewController`.

![image]({{ site.url }}/images/2015-8-7/SendingViaMail.png)

Well, above is the least we should know to use `UIActivityViewController`, and of course there's much more about this topic.

If you don't want **Mail** to show up in the activity sheet, specify it by setting `UIActivityViewController`'s property `excludedActivityTypes`, like this:

{% highlight swift %}
activityViewController.excludedActivityTypes = [UIActivityTypeMail]
{% endhighlight %}

Run the app to see the effect:

![image]({{ site.url }}/images/2015-8-7/WithoutMail.png)

If you want to forbid more, just seperate them with comma. You can see all built-in activity types [here](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIActivity_Class/index.html#//apple_ref/doc/constant_group/Built_in_Activity_Types).

It is evident that you want to use your own class as the item, then you should make it conform to `<UIActivityItemSource>`. An example is below:

{% highlight swift %}
import UIKit
class CustomClass: NSObject, UIActivityItemSource {
    func activityViewControllerPlaceholderItem(activityViewController: UIActivityViewController) -> AnyObject {
        return "I am a String"
    }
    
    func activityViewController(activityViewController: UIActivityViewController, itemForActivityType activityType: String) -> AnyObject? {
        if activityType == UIActivityTypeMessage {
            return "message"
        }
        else if activityType == UIActivityTypeMail {
            return "mail"
        }
        else {
            return "others"
        }
    }
}
{% endhighlight %}

Correspondingly, initialization of `UIActivityViewController` changes to

{% highlight swift %}
let activityViewController = UIActivityViewController(activityItems: [CustomClass()], applicationActivities: nil)
{% endhighlight %}

You may wonder what the method `activityViewControllerPlaceholderItem(_:)` is used for. 

This method provides a "promise" to `UIActivityViewController` what the `itemForActivityType` will be . The class type of the return value is used to figure out what activities to offer in the activity sheet ([reference](https://devforums.apple.com/message/881829#881829)). In the example above, I return a `String`, so the activity sheet is identical to the very first 
screenshot. If I cheat `UIActivityViewController` like this:

{% highlight swift %}
func activityViewControllerPlaceholderItem(activityViewController: UIActivityViewController) -> AnyObject {
    return UIImage()
}
{% endhighlight %}

The activity sheet will be as below:

![image]({{ site.url }}/images/2015-8-7/ForImage.png)

The control flow in `activityViewController(_:itemForActivityType:)` is not necessary, you can just return something directly if you don't need to customize your item according to different activity type.

For deeper understanding and best practice of `UIActivityViewController`, I highly recommand [this blog](http://nshipster.com/uiactivityviewcontroller/).

The demo project is [here](https://github.com/fujianjin6471/DemosForBlog/tree/master/UIActivityViewController).
