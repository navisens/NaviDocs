# The Beer Test

<sup>(Don't speak [`Android`](/BEER.Android.md)? Try [`iOS`](/BEER.iOS.md) instead!)</sup>

So you cracked open a cold one...

Let's set the scene. You've been working on a project desperately in need of a robust and accurate positioning system, yet you haven't been able to find a reliable one. Lucky for you, a friend of yours tipped you off a while back on an excellent library, and in you're half-inebriated state, your outstanding judgement dictates that now's the time to give that library a shot. With half a mind ready to start, you pull out your computer and set aside your (now mysteriously half-empty) beer.

You pull out all the tools you need:
* [Android Studio](https://developer.android.com/studio/index.html)
* An Android mobile device
* An internet connection
* The magic Key of Development (which can be obtained from [here](https://navisens.com/#contact))

## 1. The Start

For starters, you begin by **starting a new Android Studio project**.

![Create new android project](https://github.com/navisens/NaviDocs/blob/resources/Images/Android/1.1.png)

We call it the **AndroidBeerTest**.

![Give the project a name](https://github.com/navisens/NaviDocs/blob/resources/Images/Android/1.2.png)

We're gonna use a minimum of **Android 5.0** for this (so we can demo the Maps too)

![Set minimum API level](https://github.com/navisens/NaviDocs/blob/resources/Images/Android/1.3.png)

Then, we want to initialize an **empty activity**.

![Add empty activity](https://github.com/navisens/NaviDocs/blob/resources/Images/Android/1.4.png)

And finally, let's just use the default settings and call this activity **MainActivity**

![Default activity name](https://github.com/navisens/NaviDocs/blob/resources/Images/Android/1.5.png)

## 2. The Setup

Awesome. Now we've started a project. This is what it should look like once we start everything up:

![Current state setup](https://github.com/navisens/NaviDocs/blob/resources/Images/Android/2.1.png)

First, we want to set up the **project gradle buildscript**. This can be found by expanding the **gradle scripts** section in the navigator on the left. This is what it should look like:

![Open the project build.gradle](https://github.com/navisens/NaviDocs/blob/resources/Images/Android/2.2.png)

We're gonna add some new maven repo links into the **repositories** tag. Copy and paste the following new links and add them after the **jcenter** tag on *line 17*. Once we're done, we can hit the **Sync Now** button on top.

```gradle
maven {
  url 'https://maven.fabric.io/public'
}
maven {
  url 'https://raw.github.com/navisens/Android-Plugin/repositories'
}
```

![Add maven links](https://github.com/navisens/NaviDocs/blob/resources/Images/Android/2.3.png)

Next, we're gonna open up the **module gradle buildscript**. This can be found next to the project gradle buildscript in the navigator on the left. Here's what it should look like:

![Open the module build.gradle](https://github.com/navisens/NaviDocs/blob/resources/Images/Android/2.4.png)

We're gonna add some new dependencies into the **dependencies** tag. Copy and paste the following new dependencies and add them at the top of the dependencies section on *line 23*. Once we're done, we can hit the **Sync Now** button on the top again.

```gradle
compile group: "com.navisens", name: "motiondnaapi", version: "1.7.3", changing: true
compile 'com.navisens:navisenscore:3.1.1'
compile 'com.navisens:navisensmaps:1.3.2'
```

![Add dependencies](https://github.com/navisens/NaviDocs/blob/resources/Images/Android/2.5.png)

## 3. The Code

Now that we're done setting up our environment, we can go straight to the **MainActivity** file that has been generated for us. We can find the file in the navigator on the left, or it should already be in one of the open tabs. This is what it should have so far:

![Current state main activity](https://github.com/navisens/NaviDocs/blob/resources/Images/Android/3.1.png)

There are two pieces of code we need to copy in. First, we want to add our **Developer Key** at the top of the file.

```java
private static final String MOTIONDNA_KEY = "/* YOUR DEV KEY HERE */";
```

And next, within the **`onCreate`** method, we will add the following code:

```java
  NavisensCore core = new NavisensCore(MOTIONDNA_KEY, this);
  NavisensMaps maps = core.init(NavisensMaps.class)
                      .useLocalOnly()
                      .showPath().hideMarkers();

  getFragmentManager().beginTransaction().add(android.R.id.content, maps).commit();
```

There will be a few errors due to not importing the required libraries, but here's what it should look like now:

![After adding code](https://github.com/navisens/NaviDocs/blob/resources/Images/Android/3.2.png)

Let's get rid of those errors. We can import them using Android Studio auto-complete shortcuts, or just copying in these import statements:

```java
import com.navisens.pojostick.navisenscore.NavisensCore;
import com.navisens.pojostick.navisensmaps.NavisensMaps;
```

![After imports](https://github.com/navisens/NaviDocs/blob/resources/Images/Android/3.3.png)

Now, we can try running our app! Yes. That's all we needed to do for a fully-functional app. Amazing isn't it? Just hook up your android device, hit that run button, and select your phone.

And while it's compiling, you take another sip of your beer.

![Run the app](https://github.com/navisens/NaviDocs/blob/resources/Images/Android/3.4.png)

Here's what it looks like running inside our phone. *Hold my beer*, we're gonna **walk around** a bit. (Yes, in *real-life*). You should notice that your marker will follow a path now.

![Walk around](https://github.com/navisens/NaviDocs/blob/resources/Images/Android/3.5.png)

And that's all! We have a working app now that tracks our location. For some more information, your heading is marked with **three** different colors. These colors represent how we think you are moving right now. While green, you should be walking or moving. If red, then you aren't moving, but you are still holding on your phone. Finally, while blue, your phone is stationary and no one is interacting with it (try setting it down on a table). The outline shows you the current movement, while the inner region is a shaded color representing the past second of movement. For the most part, it should be correct. No, we can't tell if you are drunk. If your marker isn't moving correctly, maybe that's why :P.

## 4. More Beer!

You're lonely in this empty desolate plane of white, and you wish for companionship. So you ask a friend to grab you another beer.

*Warning: The following parts require a second beer in order to successfully complete (and a friend to share that beer with). Make sure you have a second mobile device!*

We're gonna add one line of code to our app. This is a method of the **`maps`** object, so you should call it wherever you have access to that object.

```java
.enableLocationSharing()
```

Here, we added it on *line 20*, to show you that all of our other setup functions both are called on and return the `maps` reference.

![Adding multiplayer](https://github.com/navisens/NaviDocs/blob/resources/Images/Android/4.1.png)

Now, put the app on two separate mobile devices. You should now run the two apps.

For best results, make sure the two phones are at the **same location** (within a meter of each other), and are both **facing the same direction** (not facing each other).

Give your friend the second mobile device, possibly as a worthy trade for his beer. And now you can both walk around. If your companion and friend is just that second beer, consider walking with each phone separately, and moving them to different locations relative to each other. Notice that both phones should display their own location (with a large marker), along with the other phone (in a slightly smaller marker).

![Multiplayer start](https://github.com/navisens/NaviDocs/blob/resources/Images/Android/4.2.png) ![Multiplayer moved](https://github.com/navisens/NaviDocs/blob/resources/Images/Android/4.3.png)

And now you have a working app! Thanks for drinking with us!

## 5. Even Moar Beer!!!

What? You still have more beer left? Okay then...

Here are some other cool things you can try.

* Remove the call to `useLocalOnly()`, and see what happens. You should notice that now (if you have internet connection), your phone will put you on a map and attempt to synchronize your location. You'll need to walk around a block before we figure out where exactly you are.
* You don't feel like walking around to get a location lock? No fear! Just call `addControls()` method call to your initialization. Now you can set your current location instead of waiting for it to lock! Click the gear icon and tap where you are. There are a LOT of interesting notes on this behavior, and we recommend you check out other tutorials to get a more in-depth feel for how initializing your location works.
* You're curious how you walk over time? Just remove the `hideMarkers()` call, and we'll also display a bunch of cool colored dots that display stats on how we think you've been walking around. Tap on any of the dots to get more stats!

If you've made it this far, then we thank you for trying out **Navisens MotionDna**. If not, well maybe you can just grab another beer.

*Disclaimer: Navisens in no way encourages or applauds coding while inebriated ... but a little beer never hurt anyone :P.*
