---
layout: post
title: NSLayoutConstraint
---

## 

{% highlight objc %}
[containerView layoutIfNeeded];
[UIView animateWithDuration:1.0 animations:^{
  // Make all constraint changes here
  [containerView layoutIfNeeded];
}];
{% endhighlight %}

## References

- [Animating Autolayout Constraints (useyourloaf)](http://useyourloaf.com/blog/animating-autolayout-constraints/)
