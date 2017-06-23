---
title: 'Why React Native?'
date: '2017-04-11 07:01:00'
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
---

If there was ever a  future for cross platform mobile, it's React Native. What most don't realize about RN is that even though the code is written in JavaScript, a web view is not used in the device to render the application view. What actually happens is that the *JS components* are converted into native elements by RN whenever needed. 

For example in Android, rather than jumping from activity to activity, RN will create native components in an activity (according to the JS code) when needed, and destroy them when no longer needed.

So essentially, the app will be using native components, resulting in the same performance as a native app. What's different is that the business logic (which is also written in JS) will be processed by a JS engine, running on a separate thread.

At the time I decided to gear down and seriously invest some time into learning RN, I had never used React (the front end rendering framework upon which RN is based). Here's a quick start guide on RN from my perspective. 



### setup



Setting up RN is probably harder than learning it. Follow the [installation guide](https://facebook.github.io/react-native/docs/getting-started.html) by Facebook to get everything going. Note that if you're going to be working with Android, Android Studio will, by default download the latest SDK. RN currently supports API level 23 (that's Android 6.0) so save up on bandwidth and avoid downloading the latest SDK during the Android Studio installation, if that matters to you.

After installing NodeJS, NPM, and the React Native CLI, run the following command to create a new RN project. 

```
react-native init "FirstProject"
```

An important separation here is that **React Native** is an NPM dependency of your project, and is different from the **React Native build tools** which are installed globally on your machine to debug and run React Native projects.



### first impressions



Facebook doesn't spend a lot of time maintaining the RN docs, and it's open sourced. Unfortunately, the docs can't keep up with the speed at which RN is developing. So some parts of the docs don't stick to ES6+ syntax everywhere, but you should ideally.

Developing simple apps with RN is a breeze. Especially if you're proficient in modern JS. Adding features that would have taken quite some time in native programming, are now a matter of importing a node module. So things move *fast*. You know what else moves fast? App build times, live reloading and hot reloading. There are also a lot of other tools like Expo that allow app sharing during development, remote debugging and a lot more features, for free.

The flip side of the coin includes, primarily, environmental problems. In native programming if something goes wrong, you can be fairly certain where the problem is coming from. With RN, there's always that doubt if the RN packager ran into an unexpected issue. There are also all the other issues that come with JavaScript applications on the web, which all exist here as well.



### for



If you're a web dev looking to be a mobile dev (or vice versa), this is a good way to transition. Not recommended for prototype projects that need to be done in a week or two due to the time it takes to set up and get comfortable with the framework. Great for startups, to churn out an iOS and an Android app in half the time it would have otherwise taken.

