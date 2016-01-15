---
layout: post
title:  "The least things to do to incorporate iCloud into a Core Data app"

---
I'm just wondering the least steps needed to use iCloud with Core Data to sync data among different devices. So at the very beginning, I should build a rough app using Core Data to store the data. For simplicity, I use the Core Data templete provided by Xcode, which is not recommended in many tutorials.

There are only two steps to achieve this:

**First**, set the options when adding persistent store to a coordinator. 

The code provided by the templete to add persistent store is:
{% highlight swift %}
try coordinator.addPersistentStoreWithType(NSSQLiteStoreType, configuration: nil, URL: url, options: nil)
{% endhighlight %}

Replace it by:
{% highlight swift %}
let options = [NSPersistentStoreUbiquitousContentNameKey: "iCloudAndCoredata"]
try coordinator.addPersistentStoreWithType(NSSQLiteStoreType, configuration: nil, URL: url, options: options)
{% endhighlight %}

That's all the code needed.

**Second**, turn on the iCloud, choose an account and check *iCloud Documents*:

![image]({{ site.url }}/images/2016-1-15/TurnOnIcloud.png)

![image]({{ site.url }}/images/2016-1-15/ChooseAccount.png)

![image]({{ site.url }}/images/2016-1-15/CheckIcloudDocuments.png)

All done. The demo project is [here](https://github.com/fujianjin6471/DemosForBlog/tree/master/Easy_iCloud_CoreData).

This article just makes iCloud work with CoreData. There are still a lot of things to do to make a great iCloud-sync app. For example, in the demo, if you want to see the sycn effect, you should restart the app, which is not bearable in a real app.


