---
layout: post
title:  "Thread Safety in Swfit"
date:   2018-06-11 21:07:05 -0700
categories: jekyll update
---

I have been running into some thread safety problems recently. In Objective-C, properties can be atomic and the immutable structures are thread safe while mutable structures are usually not thread safe [Ref: Apple Docs](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/Multithreading/Introduction/Introduction.html#//apple_ref/doc/uid/10000057i-CH1-SW1).
But I did not really find official docs for the Swift on thread safety.

Here is a summary afer I looked into the thread safety in Swift.
1. Swift Array and Dictionaries are not thread safe. Do not modify the data source of a tableView in a background thread. Otherwise,
there is a risk that your app may crash. Just keep in mind not to access the same array or dictionary from different threads. 
If you really have to, create a serial dispatch queue and make sure all operations on the object happens in serial. 
2. Swift objects are not thread safe. It is actually unsafe to read and write to same variable in multiple threads. There is proposal but Swift does not support @atomic yet.
2. It is possible to create thread safe Arrays and Dictionaries using locks or semaphores. There are a bunch of tutorials on this.
3. In most scenarios, just keep in mind to update the models in the view controller in the main thread will be sufficient.

