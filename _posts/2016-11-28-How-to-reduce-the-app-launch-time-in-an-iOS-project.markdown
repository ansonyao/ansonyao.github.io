---
layout: post
title:  "How-to-reduce-the-app-launch-time-in-an-iOS-project"
date:   2016-11-28 22:35:00 -0700
categories: jekyll update
---

What is a good start time for an app? If you ask your manager, the answer must be something like the shorter the better, right? As an engineer, we need numbers. Here you go: less than 400 ms is recommended in the [WWDC video](https://developer.apple.com/videos/play/wwdc2016/406/). 

If you have been using third party management tools like cocoapods, are at an disadvantage in terms of launch time! I am using cocoapods in basically everyone of my projects because dependency management really makes life easier! However, since cocoapods use framework (aka, dynamic libraries) to import the third party libraries, it makes the app launch time longer. For detailed explainnation on why dynamic libs are bad for launch time, just check out this [WWDC video](https://developer.apple.com/videos/play/wwdc2016/406/).

It is not very hard to get around this though. If the launch time really matters to your app, just ditch frameworks and copy the source code of those libs and compile them along with your project. Ah, let's assume you did not run into that Swift version hassle first! 

Anyway, that is what I got. Feel free to email me if you have other good ideas! (yaoenxin@gmail.com)

Using framework is bad for launch time for sure, but how bad is it? Again, we are engineers. What are the numbers?

In order to get a rough idea of the numbers, I made a Swift-based demo project which tries to mimic a project where cocoapods is used to import several well-known third party libraries. 

The Podfile looks like this:

{% highlight ruby %}
pod 'Alamofire', '~> 4.2'
pod 'ObjectMapper', '~> 2.2'
pod 'SDWebImage', '~> 3.8'
pod 'SnapKit', '~> 3.0'
pod 'IQKeyboardManagerSwift', '~> 4.0'
pod 'Crashlytics', '~> 3.8'
pod 'FacebookCore', '~> 0.2'
pod 'MBProgressHUD', '~> 1.0'
pod 'GoogleMaps', '~> 2.1'
{% endhighlight %}


The launch time is measured by adding environment variable in XCode project scheme editor DYLD_PRINT_STATISTICS. 
![image](/assets/images/dyld_print.png)

For cold start and hot strat, I tried my best to minic a code start, i.e., dynamic libs are not brouhgt into memeory yet before our demo app starts to launch. So in human words, I basically restarted my cell phone before clicking on that little icon of my demo app...


# The loading time (with iPhone 6s+) is printed as follows: 

Total pre-main time: 597.73 milliseconds (100.0%)

         dylib loading time: 413.50 milliseconds (69.1%)

        rebase/binding time:  18.60 milliseconds (3.1%)

            ObjC setup time:  38.85 milliseconds (6.4%)

           initializer time: 126.70 milliseconds (21.1%)

           slowest intializers :

             libSystem.B.dylib :   7.45 milliseconds (1.2%)

   libBacktraceRecording.dylib :  13.78 milliseconds (2.3%)

          libglInterpose.dylib :  77.19 milliseconds (12.9%)

         libMTLInterpose.dylib :  12.10 milliseconds (2.0%)

# Conclusions:
# 1. The dylib loading time does consume lots of time! 69% of time is used here. 

# 2. A launch time of 600 ms is not that bad... For me, using framework is not as bad as unacceptable for a moderate project.

# 3. Keep in mind that an launched iOS app does not mean that it is in a useful state. In terms of user experience, it is always important to fetch user's data and display for them in a short time. Check [here](https://www.infoq.com/news/2015/12/facebook-ios-launch-time) on how facebook use UDP over TCP as a preflight request to shorten the launch time by avoiding the handshake in TCP. A really good hack!