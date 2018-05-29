---
layout: post
title:  "Hit Test Sequence in iOS"
date:   2016-04-24 21:07:44 -0700
categories: jekyll update
---

Recently, I was confused by the code by a coworker when the code has lots of `bringSubView(toFront: UIView)` and `layer.zIndex = 2` to change the view hierarchy dynamically. Sometimes I found I triggered a button which is behind the top view! Not cool, not cool. I searched around but did not find a good material to tell me what happened in the hit test sequence in iOS. So I did a little experiment here [github link](https://github.com/ansonyao/demoHitTestSequence/) to figure it out.

The touch event in the iOS is first handled by the UIApplication and then passed to the window, view, and subviews sequencially. The views will run a hit-test to determine whether the event is inside its range such that it knows whether it should response to it or pass the click. However, a confusing issue is about the sequence of the subviews which will response first to the hit-test.
After my experiment, I have the following conclusions.

# 1. Subview runs hit test earlier than its parents. 
This is obviously but we need to keep it in mind for following discuss.

# 2. For subviews under the same parent, the subview which was added last runs the hittest first. 
You probably already know this if you worked on an iOS project. 

# 3. For subviews under the same parent, `bringSubView(toFront: UIView)` WILL change the hit test sequence. Because it basically will change the sequence of subviews in the array of `parentView.subViews`
Intuitive for me

# 4. For subviews under the same parent, change `layer.zPosition` will NOT change the hit test sequence. But it does change which view appears on top
Confusing in my mind! Need to keep in mind if you want to use zPosition to change which subview shows up.

