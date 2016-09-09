---
layout: post
title:  "How to Create Interactive Local Notification"

---

Sometimes, when a notification appears, opening the app to deal with it is tedious, we just want to achieve our goal directly from the the notification. Fortunately, interactive local notification was introduced in iOS 8 to help.

Interactive local notification has actions we can interact with, it is based on ordinary local notification that is just to notify. Let's go through ordinary local notification first. 

# Local Notification Without Actions

## Require Notification Permission

At the very beginning, we need to ask for the permission to show notifications. This can be done in `application(_:didFinishLaunchingWithOptions:)`:

{% highlight swift %}
func requireNotificationPermission() {
    let notificationSettings = UIUserNotificationSettings(forTypes: [.Alert, .Sound, .Badge], categories: nil)
    UIApplication.sharedApplication().registerUserNotificationSettings(notificationSettings)
}
{% endhighlight %}

## Create Local Notification

After requiring the notification permission, we create a local notification, it's usually triggered by something like a button tap.

{% highlight swift %}
func createLocalNotification() {
    let newLocalNotification = UILocalNotification()
    newLocalNotification.fireDate = NSDate(timeIntervalSinceNow: 5)
    newLocalNotification.timeZone = NSTimeZone.defaultTimeZone()
    newLocalNotification.alertBody = "You can't do anything with me!"
    newLocalNotification.soundName = UILocalNotificationDefaultSoundName
    UIApplication.sharedApplication().scheduleLocalNotification(newLocalNotification)
}
{% endhighlight %}

Until now, we get a local notification whose entire functionality is to notify users. Let's move on.

# Interactive Local Notification
Now that we have an ordinary local notification, it's time to add actions to it.

## Add Actions

In `AppDelegate.swift`, add this method:
{% highlight swift %}
func categoriesContainingActions() -> Set<UIMutableUserNotificationCategory> {
    let firstAction = UIMutableUserNotificationAction()
    firstAction.identifier = "FirstAction"
    firstAction.title = "action1"
    firstAction.activationMode = .Background
    firstAction.destructive = false
    firstAction.authenticationRequired = false

    let secondAction = UIMutableUserNotificationAction()
    secondAction.identifier = "SecondAction"
    secondAction.title = "action2"
    secondAction.activationMode = .Background
    secondAction.destructive = true
    secondAction.authenticationRequired = true

    let actionsArray = [firstAction, secondAction]
    let actionsArrayMinimal = [firstAction, secondAction]

    let category = UIMutableUserNotificationCategory()
    category.identifier = "ExampleCategory"
    category.setActions(actionsArray, forContext: .Default)
    category.setActions(actionsArrayMinimal, forContext: .Minimal)

    let categories: Set = [category]

    return categories
}
{% endhighlight %}

Note, we should use `UIMutableUserNotificationAction` instead of `UIUserNotificationAction` because `UIUserNotificationAction`'s property cannot be set. 

Then, what is a **category** exactly? According to the documentation:

> A UIMutableUserNotificationCategory object encapsulates information about custom actions that your app can perform in response to a local or push notification.

We create actions and put them into categories, then return an array of categories. In our case, the array just contains one category.

Wait a second, what are `.Default` and `.Minimal`? They are set for **Alert** mode and **Banners** mode seperately:

<div style="text-align:center" markdown="1">
![image]({{ site.url }}/images/2016-9-9/BannerAndAlert.png)
</div>

`destructive` decides the appearence of the action, `authenticationRequired` decide whether password is needed when executing the action.

Now we need to change the notification settings a little to below:

{% highlight swift %}
func requireNotificationPermission() {
    let notificationSettings = UIUserNotificationSettings(forTypes: [.Alert, .Sound, .Badge], categories: categoriesContainingActions())
    UIApplication.sharedApplication().registerUserNotificationSettings(notificationSettings)
}
{% endhighlight %}

We also need to add `newLocalNotification.category = "ExampleCategory"` in `createLocalNotification()` just before scheduling.

Of course, the `alertBody` set to "You can't do anything with me!" is a little strange now, we can change it to "You can do two things with me now!" It doesn't matter O(∩_∩)O~~

## Handle Actions

At last, we should tell the app how to deal with each actions.
In `AppDelegate.swift`, add this method:
{% highlight swift %}
func application(application: UIApplication, handleActionWithIdentifier identifier: String?, forLocalNotification notification: UILocalNotification, completionHandler: () -> Void) {
    if identifier == "FirstAction" {
        print("This is the first action")
    } else if identifier == "SectionAction" {
        print("This is the second action")
    }
    completionHandler()
}
{% endhighlight %}

The effect is as below:

<div style="text-align:center" markdown="1">
![image]({{ site.url }}/images/2016-9-9/InteractiveLNEffect.gif)
</div>

The demo project is [here](https://github.com/fujianjin6471/DemosForBlog/tree/master/InteractiveLocalNotification).
