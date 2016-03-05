---
layout: post
title: NSLayoutConstraint
tags: 
 - ios
---

# How do I animated autolayout constraints again?

{% highlight objc %}
[containerView layoutIfNeeded];
[UIView animateWithDuration:1.0 animations:^{
  // Make all constraint changes here
  [containerView layoutIfNeeded];
}];
{% endhighlight %}

## More references

- [Animating Autolayout Constraints (useyourloaf)](http://useyourloaf.com/blog/animating-autolayout-constraints/)
