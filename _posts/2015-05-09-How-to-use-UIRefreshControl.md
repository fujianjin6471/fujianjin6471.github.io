---
layout: post
title:  "How to use UIRefreshControl"

---
There are some third-party libraries being able to provide pull-down-refresh effect, Apple introduces its own in iOS 6.0. It's UIRefreshControl, our topic today.

We can use UIRefreshControl either in UITableViewController or just in UITableView. 

The former is more straightforward because a table view controller has a property called refreshControl. Let's begin with this situation.

Drag a table view controller from the object library and make it the initial view controller. Then add a new file through ⌘N -> iOS -> Source -> Cocoa Touch Class -> Subclass of UITableViewController, name it anything as you like, in my case, just TableViewController. After creating the new file, don't forget to relate it to the table view controller. 

Time for some code now, in TableViewController.swift

{% highlight swift %}

override func viewDidLoad() {
    super.viewDidLoad()
    refreshControl = UIRefreshControl()
    configureRefreshControl()
}
    
func configureRefreshControl() {
    refreshControl?.addTarget(self, action: "refreshEffect", forControlEvents: .ValueChanged)
}
    
func refreshEffect() {
    println("Refresh!")
    refreshControl?.endRefreshing()
}
{% endhighlight %}

Two warnings exist, just ignore them, they have nothing to do with our topic today.

Run the app.

Pull down the table view, string "Refresh!" appear in console as we expect.

What if I want to use refresh control just in a table view, not in a table view controller? Just a little more. Let's go back to the very first view controller we have forgotten╮(╯▽╰)╭

Make View Controller the initial view controller. Drag a table view onto View Controller, then ctrl-drag it to the code part to create an IBOutlet, just name it tableView. After that, add a property below IBOutlet.

{% highlight swift %}

@IBOutlet weak var tableView: UITableView!
let refreshControl = UIRefreshControl()
{% endhighlight %}

The rest is almost the same as before.

{% highlight swift %}

override func viewDidLoad() {
    super.viewDidLoad()
    configureRefreshControl()
}

func configureRefreshControl() {
    refreshControl.addTarget(self, action: "refreshEffect", forControlEvents: .ValueChanged)
    tableView.insertSubview(refreshControl, atIndex: 0)
}
    
func refreshEffect() {
    println("Refresh!")
    refreshControl.endRefreshing()
}
{% endhighlight %}

Don't forget to set constraints for table view to make it the same size as its super view, or you will see the activity indicator not in the center of the screen after you pull down the table view.

Run the app, the effect is like below:

![image]({{ site.url }}/images/2015-5-9/PullDownEffect.gif)

Of course what we have done is simple, but you can do whatever you want to after pulling down the table view.

The demo project is [here](https://github.com/fujianjin6471/UIRefreshControl).
