---
layout: post
title: iOS View Controller Transitions
tag: ios
---

## TIL

- an UIView animation block will be skipped if there are no animatable properties changing in the block. In my case, animation completion blocks were firing almost immediately.

- if not already displayed, invoking ```transitionContext.completeTransition(true)``` will cause the destination view to display

- [Render View Into Image Faster](http://stackoverflow.com/questions/19066717/how-to-render-view-into-image-faster)

## Examples

- [The Right Way](http://netsplit.com/custom-ios-segues-transitions-and-animations-the-right-way)

### [objc.io Issue 5 - View Controller Transitions](https://www.objc.io/issues/5-ios7/view-controller-transitions/)

{% highlight swift %}
    func animateTransition(transitionContext: UIViewControllerContextTransitioning) {

        let sourceVC = transitionContext.viewControllerForKey(UITransitionContextFromViewControllerKey)!
        
        let destinationVC = transitionContext.viewControllerForKey(UITransitionContextToViewControllerKey)!
        let destinationView = destinationVC.view
        
        let container = transitionContext.containerView()!
        container.addSubview(destinationView)

        destinationVC.view.alpha = 0
        
        UIView.animateWithDuration(transitionDuration(transitionContext), animations: { () -> Void in
            sourceVC.view.transform = CGAffineTransformMakeScale(0.1, 0.1);
            destinationVC.view.alpha = 1
        }) { (Success) -> Void in
            sourceVC.view.transform = CGAffineTransformIdentity
            transitionContext.completeTransition(!transitionContext.transitionWasCancelled())
        }
        
    }
	{% endhighlight %}

### [Google Inbox Style Expanding Cell]()

{% highlight swift %}
    func animateTransition2(transitionContext: UIViewControllerContextTransitioning) {
        let sourceVC = transitionContext.viewControllerForKey(UITransitionContextFromViewControllerKey)!
        let sourceView = sourceVC.view
        
        let destinationVC = transitionContext.viewControllerForKey(UITransitionContextToViewControllerKey)!
        let destinationView = destinationVC.view
        
        let container = transitionContext.containerView()!
        
        self.selectedCellFrame = CGRectMake(self.selectedCellFrame.origin.x, self.selectedCellFrame.origin.y + self.selectedCellFrame.height, self.selectedCellFrame.width, self.selectedCellFrame.height)
        
        var snapShot = UIImage()
        let bounds = CGRectMake(0, 0, sourceView.bounds.size.width, sourceView.bounds.size.height)
        
        UIGraphicsBeginImageContextWithOptions(sourceView.bounds.size, true, 0)
        
        if self.operation == UINavigationControllerOperation.Push {
            sourceView.drawViewHierarchyInRect(bounds, afterScreenUpdates: false)
            
            snapShot = UIGraphicsGetImageFromCurrentImageContext()
            UIGraphicsEndImageContext()
            
            let tempImageRef = snapShot.CGImage!
            let imageSize = snapShot.size
            let imageScale = snapShot.scale

            let midPoint = bounds.height / 2
            let selectedFrame = self.selectedCellFrame.origin.y - (self.selectedCellFrame.height / 2)
            
            let padding = self.selectedCellFrame.height * imageScale
            
            var topHeight: CGFloat = 0.0
            var bottomHeight: CGFloat = 0.0
            
            if selectedFrame < midPoint {
                topHeight = self.selectedCellFrame.origin.y * imageScale
                bottomHeight = (imageSize.height - self.selectedCellFrame.origin.y) * imageScale
            } else {
                topHeight = (self.selectedCellFrame.origin.y * imageScale) - padding
                bottomHeight = ((imageSize.height - self.selectedCellFrame.origin.y) * imageScale) + padding
            }
            
            let topImageRect = CGRectMake(0, 0, imageSize.width * imageScale, topHeight)
            
            let bottomImageRect = CGRectMake(0, topHeight, imageSize.width * imageScale, bottomHeight)

            let topImageRef = CGImageCreateWithImageInRect(tempImageRef, topImageRect)!
            let bottomImageRef = CGImageCreateWithImageInRect(tempImageRef, bottomImageRect)
            
            self.imageViewTop = UIImageView(image: UIImage(CGImage: topImageRef, scale: snapShot.scale, orientation: UIImageOrientation.Up))
            
            if (bottomImageRef != nil) {
                self.imageViewBottom = UIImageView(image: UIImage(CGImage: bottomImageRef!, scale: snapShot.scale, orientation: UIImageOrientation.Up))
            }
        }
        
        var startFrameTop = self.imageViewTop!.frame
        var endFrameTop = startFrameTop
        var startFrameBottom = self.imageViewBottom!.frame
        var endFrameBottom = startFrameBottom
        
        if self.operation == UINavigationControllerOperation.Pop {
            startFrameTop.origin.y = -startFrameTop.size.height
            endFrameTop.origin.y = 0
            startFrameBottom.origin.y = startFrameTop.height + startFrameBottom.height
            endFrameBottom.origin.y = startFrameTop.height
        } else {
            startFrameTop.origin.y = 0
            endFrameTop.origin.y = -startFrameTop.size.height
            startFrameBottom.origin.y = startFrameTop.size.height
            endFrameBottom.origin.y = startFrameTop.height + startFrameBottom.height
        }
        
        self.imageViewTop!.frame = startFrameTop
        self.imageViewBottom!.frame = startFrameBottom
        
        // hide the destination view
        destinationView.alpha = 0
        
        // add the destination view (hidden)
        container.addSubview(destinationView)
        
        // add the fake facade views (match current)
        container.addSubview(self.imageViewTop!)
        container.addSubview(self.imageViewBottom!)
        
        if self.operation == UINavigationControllerOperation.Pop {
            
            UIView.animateWithDuration(transitionDuration(transitionContext), animations: { () -> Void in
                
                // come together!
                self.imageViewTop!.frame = endFrameTop
                self.imageViewBottom!.frame = endFrameBottom
                
                // fade away
                sourceView.alpha = 0

            }, completion: { (finish) -> Void in
                
                // appear behind facade
                destinationView.alpha = 1

                // remove from view (previous line destination view should show up)
                self.imageViewTop!.removeFromSuperview()
                self.imageViewBottom!.removeFromSuperview()

                transitionContext.completeTransition(true)
            })
            
        } else {
            
            // hide the source view (the facade currently covers it up)
            sourceView.alpha = 0
            
            // separate out
            UIView.animateWithDuration(transitionDuration(transitionContext), animations: { () -> Void in
                self.imageViewTop!.frame = endFrameTop
                self.imageViewBottom!.frame = endFrameBottom
                
                // fade the underlying view in
                destinationView.alpha = 1
            
            }, completion: { (finish) -> Void in
                self.imageViewTop!.removeFromSuperview()
                self.imageViewBottom!.removeFromSuperview()
                
                transitionContext.completeTransition(true)
            })
        }
    }
{% endhighlight %}
