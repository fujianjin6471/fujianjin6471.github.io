---
layout: post
title:  "How to use custom presentation"

---

When `UIAlertController` is not enough to show what you want but you still need the modal style, it is up to `UIPresentationController` and its siblings.

Let's begin with a single view application. In our case, the existing view controller will be the *presenting view controller*, responsible for presenting another view controller. Besides, we need a *presented view controller*, a *transitioning delegate*, a *presentation controller* and an *animation controller*. Sounds a little complicated? Let's go through them one by one.

# Presented View Controller

Presented view controller is the controller presented by presenting view controller. It's just an ordinary view controller. The only additional thing to do is to set its `modalPresentationStyle` be `UIModalPresentationCustom` before presentation. And **don't forget to set Storyboard ID if you use storyboard to configure presented view controller**. I set it "**PresentedVC**".

# Transitioning Delegate

Presented view controller has a property `transitioningDelegate`. It's responsible for **providing presentation controller and animation controller**, we set it before the presentation.

# Presentation Controller

Presentation controller is responsible for **defining the frame of the presented view** and **customizing the dimming view** during the presentation. We create a class `PresentationController` which is a subclass of `UIPresentationController` for it.

## Defining the Frame of the Presented View

Both **size** and **position** information of the presented view should be provided. Override these two methods:

{% highlight swift %}
override func frameOfPresentedViewInContainerView() -> CGRect {
    var presentedViewFrame = CGRectZero
    let containerBounds = containerView!.bounds
    presentedViewFrame.size = sizeForChildContentContainer(presentedViewController, withParentContainerSize: containerBounds.size)
    presentedViewFrame.origin.x = (containerBounds.size.width - presentedViewFrame.size.width) / 2
    presentedViewFrame.origin.y = (containerBounds.size.height - presentedViewFrame.size.height) / 2
    return presentedViewFrame
}

override func sizeForChildContentContainer(container: UIContentContainer, withParentContainerSize parentSize: CGSize) -> CGSize {
    return CGSizeMake(parentSize.width / 1.5, parentSize.height / 2.0)
}
{% endhighlight %}

Of course you can adjust the frame as you need.

## Customizing the Dimming View

This part contains creating and animating the dimming view. First, we need to create the dimming view and set it up:

{% highlight swift %}
var dimmingView: UIView = UIView()

override init(presentedViewController: UIViewController, presentingViewController: UIViewController) {
    super.init(presentedViewController: presentedViewController, presentingViewController: presentingViewController)
    setupDimmingView()
}

func setupDimmingView() {
    dimmingView.backgroundColor = UIColor(white: 0, alpha: 0.5)
    let tap = UITapGestureRecognizer(target: self, action: #selector(dimmingViewTapped(_:)))
    dimmingView.addGestureRecognizer(tap)
}

func dimmingViewTapped(gesture: UIGestureRecognizer) {
    if (gesture.state == UIGestureRecognizerState.Ended) {
        presentingViewController.dismissViewControllerAnimated(true, completion: nil)
    }
}
{% endhighlight %}

Then implement animating the dimming view for presentation and dismissal:

{% highlight swift %}
override func presentationTransitionWillBegin() {
    dimmingView.frame = self.containerView!.bounds
    dimmingView.alpha = 0.0
    containerView!.insertSubview(dimmingView, atIndex:0)
    let coordinator = presentedViewController.transitionCoordinator()
    if (coordinator != nil) {
        coordinator!.animateAlongsideTransition({
            (context:UIViewControllerTransitionCoordinatorContext!) -> Void in
            self.dimmingView.alpha = 1.0
            }, completion:nil)
    } else {
        dimmingView.alpha = 1.0
    }
}

override func dismissalTransitionWillBegin() {
    let coordinator = presentedViewController.transitionCoordinator()
    if (coordinator != nil) {
        coordinator!.animateAlongsideTransition({
            (context:UIViewControllerTransitionCoordinatorContext!) -> Void in
            self.dimmingView.alpha = 0.0
            }, completion:nil)
    } else {
        dimmingView.alpha = 0.0
    }
}
{% endhighlight %}

## Deal With Rotation

{% highlight swift %}
override func containerViewWillLayoutSubviews() {
    dimmingView.frame = containerView!.bounds
    presentedView()!.frame = frameOfPresentedViewInContainerView()
}
{% endhighlight %}

Without this method, when you rotate the device, just see it by yourself...

# Animation Controller

Animation controller is also called *animator object*, it is responsible for **creating the animations for transitioning a view controller on or off screen in a fixed amount of time**. Usually, we create a presentation animator object and a dismissal animator object respectively in one file. Both animator object need to implement `transitionDuration(_:)` to set a fixed time and `animateTransition(_:)` to control the animation of transition.

{% highlight swift %}
class PresentationAnimatedTransitioning: NSObject, UIViewControllerAnimatedTransitioning {
    func transitionDuration(transitionContext: UIViewControllerContextTransitioning?) -> NSTimeInterval {
        return 0.5
    }
    
    func animateTransition(transitionContext: UIViewControllerContextTransitioning) {
        let toVC = transitionContext.viewControllerForKey(UITransitionContextToViewControllerKey)
        let toView = toVC?.view
        let containerView = transitionContext.containerView()
        
        containerView!.addSubview(toView!)
        
        let finalFrame = transitionContext.finalFrameForViewController(toVC!)
        var initialFrame = finalFrame
        initialFrame.origin.x = containerView!.frame.width
        
        toView?.frame = initialFrame
        
        UIView.animateWithDuration(transitionDuration(transitionContext), animations: {
            toView?.frame = finalFrame
            }, completion: { finished in
                transitionContext.completeTransition(true)
        })
    }
}

class DismissalAnimatedTransitioning: NSObject, UIViewControllerAnimatedTransitioning {
    func transitionDuration(transitionContext: UIViewControllerContextTransitioning?) -> NSTimeInterval {
        return 0.5
    }
    
    func animateTransition(transitionContext: UIViewControllerContextTransitioning) {
        let fromVC = transitionContext.viewControllerForKey(UITransitionContextFromViewControllerKey)
        let fromView = fromVC?.view
        let containerView = transitionContext.containerView()
        
        let initialFrame = transitionContext.initialFrameForViewController(fromVC!)
        var finalFrame = initialFrame
        finalFrame.origin.x = containerView!.frame.width
        
        fromView?.frame = initialFrame
        
        UIView.animateWithDuration(transitionDuration(transitionContext), animations: {
            fromView?.frame = finalFrame
            }, completion: { finished in
                fromView?.removeFromSuperview()
                transitionContext.completeTransition(true)
        })
    }
}
{% endhighlight %}

There are several points need to mention:

1. `transitionContext` is the context object containing information about the transition. For examples, the presenting view controller and the presented view controller, the container view.
2. In the transition of presentation, we get the ending frame rectangle for presented view controller. Why do we know that? Because we set it in presentation controller.
3. In the transition of dismissal, we get the starting frame rectangle for presented view controller in the same way as before.
4. The container view is always an ancestor of the presented view controller’s view. We add presented view controller's view to it before animation of presentation and remove presented view controller's view from it after animation of dismissal.

# Back to Transitioning Delegate

Now that we have `PresentationController`, `PresentationAnimatedTransitioning` and `DismissalAnimatedTransitioning`, the transitioning delegate is very straightforward:

{% highlight swift %}
import UIKit

class TransitioningDelegate: NSObject, UIViewControllerTransitioningDelegate {
    func presentationControllerForPresentedViewController(presented: UIViewController, presentingViewController presenting: UIViewController, sourceViewController source: UIViewController) -> UIPresentationController? {
        let presentationController = PresentationController(presentedViewController: presented, presentingViewController: presenting)
        
        return presentationController
    }
    
    func animationControllerForPresentedController(presented: UIViewController, presentingController presenting: UIViewController, sourceController source: UIViewController) -> UIViewControllerAnimatedTransitioning? {
        let animationController = PresentationAnimatedTransitioning()
        return animationController
    }
    
    func animationControllerForDismissedController(dismissed: UIViewController) -> UIViewControllerAnimatedTransitioning? {
        let animationController = DismissalAnimatedTransitioning()
        return animationController
    }
}
{% endhighlight %}

# Time to Present

We have all the components ready for the presentation, time to present now:

{% highlight swift %}
import UIKit

class ViewController: UIViewController {
    let customTransitioningDelegate = TransitioningDelegate()
    override func viewDidLoad() {
        super.viewDidLoad()
    }

    @IBAction func presentAction(sender: UIButton) {
        let presentedVC = storyboard?.instantiateViewControllerWithIdentifier("PresentedVC") as! PresentedVC
        presentedVC.transitioningDelegate = customTransitioningDelegate
        presentedVC.modalPresentationStyle = .Custom
        presentViewController(presentedVC, animated: true, completion: nil)
    }
}
{% endhighlight %}

The effect is as below:

<div style="text-align:center" markdown="1">
![image]({{ site.url }}/images/2016-8-11/CustomPresentationEffect.gif)
</div>

The demo project is [here](https://github.com/fujianjin6471/DemosForBlog/tree/master/CustomPresentation).
