---
layout: post
title:  "How to save data using NSCoder"

---
Saving and loading data is very common in iOS app development. Sometimes Core Data is a little unwieldy, then using NSCoder becomes a better choice.

Let's make it as simple as possible, any unnecessary part which may be helpful from other respect won't appear in this blog.

Drag a text field and a button onto the scene, and don't forget to make connections between IB and view controller.

Say you type in some characters and want them to reappear if you restart the app.

Firstly, we should know where to put the data we will create. Add this method to **ViewController.swift**:

{% highlight swift %}

func dataFilePath() -> String {
    let paths = NSSearchPathForDirectoriesInDomains(.DocumentDirectory, .UserDomainMask, true) as! [String]
    return (paths[0]).stringByAppendingPathComponent("TextField.plist")
}]
{% endhighlight %}

How to save the text? add a method to do this:

{% highlight swift %}

func saveText() {
    let data = NSMutableData()
    let archiver = NSKeyedArchiver(forWritingWithMutableData: data)
    archiver.encodeObject(textField.text, forKey: "FieldText")
    archiver.finishEncoding()
    data.writeToFile(dataFilePath(), atomically: true)
}
    
{% endhighlight %}

Two steps included:

1. NSKeyedArchiver, subclass of NSCoder, encodes the text of the text field into some kind of data that could be written to a file.
2. Writing the data to a file specified *by dataFilePath()*.

We just call *saveText()* when the button is pressed for the sack of simplicity. If you want to save the text every time you change it automatically, make the view controller conforms to *UITextFieldDelegate* (don't forget to make view controller be text field's delegate) and implement *textField(_:shouldChangeCharactersInRange:replacementString:)* like this:

{% highlight swift %}

func textField(textField: UITextField, shouldChangeCharactersInRange range: NSRange, replacementString string: String) -> Bool {
    let formerText = textField.text as NSString
    textField.text = formerText.stringByReplacingCharactersInRange(range, withString: string)
    saveText()
    textField.text = formerText as String
    return true
}
    
{% endhighlight %}

Then we are going to implement loading text, add the following method:

{% highlight swift %}

func loadText() {
    let path = dataFilePath()
    if NSFileManager.defaultManager().fileExistsAtPath(path) {
        if let data = NSData(contentsOfFile: path) {
            let unarchiver = NSKeyedUnarchiver(forReadingWithData: data)
            textField.text = unarchiver.decodeObjectForKey("FieldText") as! String
            unarchiver.finishDecoding()
        }
    }
}
{% endhighlight %}

Call it at the bottom in *viewDidLoad()*.

Run the app, type some characters into the text field and tap the button to save the text. Then restart the app, what you typed before reappear in the text field, that's what we want.

**String** is one of the built-in types of Swift, we save and load it using the aforementioned method. What if you want to save and load an object of your own? For consistency, we create a class containing only a **String** property as below:

{% highlight swift %}

class CustomClass: NSObject {
    var text = ""
}
{% endhighlight %}

Then embed the view controller in a tab bar controller and create another view controller called **ViewController2**, as the second scene of the tab bar controller. **ViewController2** is almost the same as **ViewController**.

{% highlight swift %}

import UIKit

class ViewController2: UIViewController {
    @IBOutlet weak var textField: UITextField!
    var onlyText = CustomClass()

    override func viewDidLoad() {
        super.viewDidLoad()
        
        loadCustomObject()
        textField.text = onlyText.text
    }
    
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
    
    func dataFilePath() -> String {
        let paths = NSSearchPathForDirectoriesInDomains(.DocumentDirectory, .UserDomainMask, true) as! [String]
        return (paths[0]).stringByAppendingPathComponent("Object.plist")
    }
    
    func saveCustomObject() {
        onlyText.text = textField.text
        
        let data = NSMutableData()
        let archiver = NSKeyedArchiver(forWritingWithMutableData: data)
        archiver.encodeObject(onlyText, forKey: "CustomObject")
        archiver.finishEncoding()
        data.writeToFile(dataFilePath(), atomically: true)
    }
    
    func loadCustomObject() {
        let path = dataFilePath()
        if NSFileManager.defaultManager().fileExistsAtPath(path) {
            if let data = NSData(contentsOfFile: path) {
                let unarchiver = NSKeyedUnarchiver(forReadingWithData: data)
                onlyText = unarchiver.decodeObjectForKey("CustomObject") as! CustomClass
                unarchiver.finishDecoding()
            }
        }
    }

    @IBAction func saveAction(sender: AnyObject) {
        saveCustomObject()
        let alert = UIAlertController(title: "Prompt", message: "Data saved!", preferredStyle: .Alert)
        let firstAction = UIAlertAction(title: "OK", style: UIAlertActionStyle.Default, handler: nil)
        alert.addAction(firstAction)
        presentViewController(alert, animated: true, completion: nil)
    }
}
    
{% endhighlight %}

Run the app, type something and press **Save** button, it crashes! But we don't know the reason from the default prompt of Xcode, let's do it manually. Add Exception Breakpoint and repeat behavior before, you will get this:

![image]({{ site.url }}/images/2015-7-8/Crash.png)

This line of code cause the crash:

{% highlight swift %}

archiver.encodeObject(onlyText, forKey: "CustomObject")
{% endhighlight %}

And the Console says *encodeWithCoder* is an unrecognized selector for **CustomClass**. This always happens when we try to use a method but the implementation is missing.

*encodeWithCoder* is a method of **NSCoding** protocol, so let's make **CustomClass** conform to **NSCoding** and implement two required method of it like this:

{% highlight swift %}

func encodeWithCoder(aCoder: NSCoder) {
    aCoder.encodeObject(text, forKey: "Text")
}

required convenience init(coder aDecoder: NSCoder) {
    self.init()	
    text = aDecoder.decodeObjectForKey("Text") as! String
}
{% endhighlight %}

Run the app again, it works just as you expect.

The final project is [here](https://github.com/fujianjin6471/DemosForBlog/tree/master/UsingNSCoder).

