---
layout: post
title:  "How to implement inline date picker"

---
Inline date picker is an elegant way to change date, which is introduced in iOS 7. This is how it looks like in the Calendar app:

<div style="text-align:center" markdown="1">
![image]({{ site.url }}/images/2016-2-3/Calendar.png)
</div>

Let's make it from scratch. Create a new Project in Xcode, and choose the **iOS\Application\Single View Application** templete. After creation of the project, open **Main.storyboard**, delete the initial view controller, drag a Table View Controller into the storyboard and make it the initial view controller. Then delete **ViewController.swift** in Navigator, go to **File\New\File...**, choose the **iOS\Source\Cocoa Touch Class** templete, name the class **TableViewController** and make it a subclass of **UITableViewController**. Then back to **Main.storyboard**, select the Table View Controller, change its class to **TableViewController** using the Identity Inspector (3rd tab). These are the preparatory work.

Our table view will display a list of events including their titles and time. When selecting one event, and inline date picker will appear allowing you to change the date. First, let's define a Event class. Go to **File\New\Files...**, choose the **iOS\Source\Swift File** templete and name it **Event.swift**. We just need two properties, a title and a time, followed by an initializer:

{% highlight swift %}
import Foundation
class Event {
    let title: String
    var time: NSDate
    init(title: String, time: NSDate) {
        self.title = title
        self.time = time
    }
}
{% endhighlight %}

Next, select the table view, set **Prototype Cells** to be 2 since we need two kinds of cells, one for displaying info of an event and one for displaying the date picker. Choose the style for the cell **Right Detail** and **Custom**, set the identifiers for them to be *EventCell* and *DatePickerCell* respectively. The second cell should be big enough to show the date picker properly, so set its row height (about 200 points) using the Size Inspector. Drag a Date Picker into the second cell and make them the same size using constraints. 

Before showing the date picker when we select a row, we should display a list of events correctly. Since we will display the time of an event, an **NSDate** object, we need a date formatter to obtain a string.

{% highlight swift %}
var dateFormatter = NSDateFormatter()
func setDateFormatter() { // called in viewDidLoad()
    dateFormatter.dateStyle = .ShortStyle
}
{% endhighlight %}

We need an array property to hold some events.

{% highlight swift %}
var events = [Event]()
func createEvents() { // called in viewDidLoad()
    let event1 = Event(title: "Swift 1.0", time: dateFormatter.dateFromString("6/2/14")!)
    let event2 = Event(title: "Swift 2.0", time: dateFormatter.dateFromString("9/16/15")!)
    let event3 = Event(title: "Swift Open Source", time: dateFormatter.dateFromString("12/4/15")!)
    
    events.append(event1)
    events.append(event2)
    events.append(event3)
}
{% endhighlight %}


Then the table view data source:

{% highlight swift %}
override func numberOfSectionsInTableView(tableView: UITableView) -> Int {
    return 1
}

override func tableView(tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
    return events.count
}

override func tableView(tableView: UITableView, cellForRowAtIndexPath indexPath: NSIndexPath) -> UITableViewCell {
    let cell = tableView.dequeueReusableCellWithIdentifier("EventCell")!
    let event = events[indexPath.row]
    cell.textLabel!.text = event.title
    cell.detailTextLabel!.text = dateFormatter.stringFromDate(event.time)
    return cell
}
{% endhighlight %}

Run the app, the table view displays three events just as we expect.

<div style="text-align:center" markdown="1">
![image]({{ site.url }}/images/2016-2-3/ShowEvents.png)
</div>

But no matter which row you select by tapping, nothing happen. Now it's time to implement showing inline date picker when you select a row. Though this is the toughest part, we have prepared anything for it, let's continue.

When we tap a row, `tableView:didSelectRowAtIndexPath:` is called. There are **three cases**:

1. There is no date picker shown, we tap a row, then a date picker is shown just under it.
2. A date picker is shown, we tap the row just above it, then the date picker is hidden.
3. A date picker is shown, we tap a row that is not just above it, then the date picker is hidden and another date picker under the tapped row is shown. And there are **two subcases**:
    * the tapped is above the shown date picker
    * the tapped is under the shown date picker

Since we need to know whether a date picker is shown, and where it is, add a property to achieve these tasks:
{% highlight swift %}
var datePickerIndexPath: NSIndexPath?
{% endhighlight %}

If the date picker is hidden, `datePickerIndexPath` is `nil`, else it contains the location information of the date picker. The logic of three cases is translated as below:
{% highlight swift %}
override func tableView(tableView: UITableView, didSelectRowAtIndexPath indexPath: NSIndexPath) {
    tableView.beginUpdates() // because there are more than one action below
    if datePickerIndexPath != nil && datePickerIndexPath!.row - 1 == indexPath.row { // case 2
        tableView.deleteRowsAtIndexPaths([datePickerIndexPath!], withRowAnimation: .Fade)
        datePickerIndexPath = nil
    } else { // case 1、3
        if datePickerIndexPath != nil { // case 3
            tableView.deleteRowsAtIndexPaths([datePickerIndexPath!], withRowAnimation: .Fade)
        }
        datePickerIndexPath = calculateDatePickerIndexPath(indexPath)
        tableView.insertRowsAtIndexPaths([datePickerIndexPath!], withRowAnimation: .Fade)
    }
    tableView.deselectRowAtIndexPath(indexPath, animated: true)
    tableView.endUpdates()
}

func calculateDatePickerIndexPath(indexPathSelected: NSIndexPath) -> NSIndexPath {
    if datePickerIndexPath != nil && datePickerIndexPath!.row  < indexPathSelected.row { // case 3.2
        return NSIndexPath(forRow: indexPathSelected.row, inSection: 0)
    } else { // case 1、3.1
        return NSIndexPath(forRow: indexPathSelected.row + 1, inSection: 0)
    }
}
{% endhighlight %}

If the table view wants to display a date picker, a new row should be inserted, change `tableView:numberOfRowsInSection:` a little:
{% highlight swift %}
override func tableView(tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
    var rows = events.count
    if datePickerIndexPath != nil {
        rows++
    }
    return rows
}
{% endhighlight %}

And now there are two types of cells to display, `tableView:cellForRowAtIndexPath:` should also be changed:
{% highlight swift %}
override func tableView(tableView: UITableView, cellForRowAtIndexPath indexPath: NSIndexPath) -> UITableViewCell {
    var cell: UITableViewCell
    if datePickerIndexPath != nil && datePickerIndexPath!.row == indexPath.row {
        cell = tableView.dequeueReusableCellWithIdentifier("DatePickerCell")!
        let datePicker = cell.viewWithTag(1) as! UIDatePicker // set the tag of Date Picker to be 1 in the Attributes Inspector
        let event = events[indexPath.row - 1]
        datePicker.setDate(event.time, animated: true)
    } else {
        cell = tableView.dequeueReusableCellWithIdentifier("EventCell")!
        let event = events[indexPath.row]
        cell.textLabel!.text = event.title
        cell.detailTextLabel!.text = dateFormatter.stringFromDate(event.time)
    }
    return cell
}
{% endhighlight %}

The height of date picker cell should be enough to show the date picker, so let's override `tableView:heightForRowAtIndexPath:`
{% highlight swift %}
override func tableView(tableView: UITableView, heightForRowAtIndexPath indexPath: NSIndexPath) -> CGFloat {
    var rowHeight = tableView.rowHeight
    if datePickerIndexPath != nil && datePickerIndexPath!.row == indexPath.row {
        let cell = tableView.dequeueReusableCellWithIdentifier("DatePickerCell")!
        rowHeight = cell.frame.height
    }
    return rowHeight
}
{% endhighlight %}

If you run the app now, the date picker will be shown and hidden as you expect. But the relevant event cell won't change the date shown on the right as you change the date of the date picker, so let's fix this. Create an action method for the Date Picker when the value is changed:

<div style="text-align:center" markdown="1">
![image]({{ site.url }}/images/2016-2-3/ChangeDateAction.png)
</div>

{% highlight swift %}
@IBAction func changeDate(sender: UIDatePicker) {
    let parentIndexPath = NSIndexPath(forRow: datePickerIndexPath!.row - 1, inSection: 0)
    // change model
    let event = events[parentIndexPath.row]
    event.time = sender.date
    // change view
    let eventCell = tableView.cellForRowAtIndexPath(parentIndexPath)!
    eventCell.detailTextLabel!.text = dateFormatter.stringFromDate(sender.date)
}
{% endhighlight %}

All done. The final effect is shown below:

<div style="text-align:center" markdown="1">
![image]({{ site.url }}/images/2016-2-3/FinalEffect.gif)
</div>

The demo project is [here](https://github.com/fujianjin6471/DemosForBlog/tree/master/InlineDatePicker).
