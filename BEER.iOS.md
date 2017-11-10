# The Beer Test

<sub>(Don't speak [`iOS`](BEER.iOS.md)? Try [`Android`](BEER.Android.md) instead!)</sub>

So you cracked open a cold one...

Let's set the scene. You've been working on a project desperately in need of a robust and accurate positioning system, yet you haven't been able to find a reliable one. Lucky for you, a friend of yours tipped you off a while back on an excellent library, and in you're half-inebriated state, your outstanding judgement dictates that now's the time to give that library a shot. With half a mind ready to start, you pull out your computer and set aside your (now mysteriously half-empty) beer.

You pull out all the tools you need:
* [XCode](https://developer.apple.com/xcode/)
* [CocoaPods](https://cocoapods.org)
* An iOS mobile device
* An internet connection
* The magic Key of Development (which can be obtained from *here(link pending)*)

## 1. The Start

For starters, you begin by **creating a new Xcode workspace**.

![Create new xcode project](https://github.com/navisens/NaviDocs/blob/resources/Images/iOS/1.1.png)

We'll choose the **Single View App** for this.

![Use single view app](https://github.com/navisens/NaviDocs/blob/resources/Images/iOS/1.2.png)

And let's call it the **iOSBeerTest**.

![Give the project a name](https://github.com/navisens/NaviDocs/blob/resources/Images/iOS/1.3.png)

## 2. The Setup

Awesome. Now we've created a project. This is what it should look like once we start everything up:

![Current state setup](https://github.com/navisens/NaviDocs/blob/resources/Images/iOS/2.1.png)

We need to create a new file inside the main directory. Just right click and select **new file**.

![Create new file](https://github.com/navisens/NaviDocs/blob/resources/Images/iOS/2.2.png)

We're just gonna choose an **empty** template for the file, so we can put whatever we want in it.

![Use empty template](https://github.com/navisens/NaviDocs/blob/resources/Images/iOS/2.3.png)

We need to name this file `Podfile`. We'll be using this to setup the CocoaPods environment.

![Name it podfile](https://github.com/navisens/NaviDocs/blob/resources/Images/iOS/2.4.png)

We need to add some dependencies to the `Podfile`. Copy and paste the following new dependencies in. This sets up our CocoaPods environment for iOS 9.1, and attaches the environment to the target `iOSBeerTest`. We then import our core SDK, along with the plugins required for this project.

```podfile
source 'https://github.com/CocoaPods/Specs.git'
platform :ios, '9.1'

target 'iOSBeerTest' do
  use_frameworks!
  pod 'MotionDnaSDK', :git => 'https://github.com/navisens/iOS-SDK.git'
  pod ‘NavisensPlugins/NavisensCore’, :git => 'https://github.com/navisens/iOS-Plugin.git', :branch => 'repositories'
  pod ‘NavisensPlugins/NavisensMaps’, :git => 'https://github.com/navisens/iOS-Plugin.git', :branch => 'repositories'
end
```

![Add dependencies](https://github.com/navisens/NaviDocs/blob/resources/Images/iOS/2.5.png)

At this point, we can **close Xcode**. We'll open it up later again.

Now we'll need to open up a terminal to install the new environment. First change the working directory to wherever you saved `iOSBeerTest` using the command `cd`. Make sure you're in the right folder with the `ls` command. If you are, you should see printed below a file named `Podfile`. We'll install the pods next.

![Pre install pods](https://github.com/navisens/NaviDocs/blob/resources/Images/iOS/2.6.png)

To install the pods, run the following command:

```bash
pod install
```

![Install pods](https://github.com/navisens/NaviDocs/blob/resources/Images/iOS/2.7.png)

Finally, as stated in the installation text, we need to open up the `xcworkspace` instead of the `xcodeproj` file. To do that, we can just run:

```bash
open iOSBeerTest.xcworkspace
```

![Post install pods](https://github.com/navisens/NaviDocs/blob/resources/Images/iOS/2.8.png)

## 3. The Configuration

After we open up the new Xcode workspace, we can verify we have everything setup successfully. Just check the left side and you should see two modules. One is our `iOSBeerTest`. The second one called `Pods` is created by CocoaPods in order to store our dependencies. Go ahead and click on `iOSBeerTest` so we can configure the new workspace.

![Current state](https://github.com/navisens/NaviDocs/blob/resources/Images/iOS/3.1.png)

First, we need to enable background location services. To do that, we go to the **Capabilities** tab, scroll down to the bottom and look for **Background Modes**, and flip the switch on the right hand side to **On**. Some options should then appear below. We want to enable the **Location updates** option by ticking the checkbox.

![Enable background location updates](https://github.com/navisens/NaviDocs/blob/resources/Images/iOS/3.2.png)

Next, we need to tell our user why we're asking them for their location, in order to ensure application security. Go to the **Info** tab, and click the **+** button on any of already-existing information messages. This should cause a new option to appear. For that option, we want to search for the **Privacy - Location When in Use Usage Description.** message. Alternatively, just copy the following key instead and that will work too:

```xcode
NSLocationWhenInUseUsageDescription
```

![Location when in use usage description](https://github.com/navisens/NaviDocs/blob/resources/Images/iOS/3.3.png)

Fill in the value on the right with whatever message you want. This message will appear whenever we ask the user for permission to access their location.

![Location message](https://github.com/navisens/NaviDocs/blob/resources/Images/iOS/3.4.png)

Next, we want to disable bitcode. To do that, go to the **Build Settings** tab, and type into the search bar **bitcode**. You should see an option appear below called **Enable Bitcode**. We're gonna go ahead and set that to **No**.

![Disable bitcode](https://github.com/navisens/NaviDocs/blob/resources/Images/iOS/3.5.png)

That's it for setting up the environment!

## 4. The Code

Now we can get to the application creation part! First of all, we need something to display our maps app. To do that, we need to go to our storyboard file **Main.storyboard**. Right now, it should appear empty.

![Current state storyboard](https://github.com/navisens/NaviDocs/blob/resources/Images/iOS/4.1.png)

Go on to the bottom right section. Here, we can find a list of other components to add. We want to search for the **Container View**.

![Search container view](https://github.com/navisens/NaviDocs/blob/resources/Images/iOS/4.2.png)

We want to add this view to our main view. To do that, just click and drag the **Container View** into the app area.

![Add container view](https://github.com/navisens/NaviDocs/blob/resources/Images/iOS/4.3.png)

We want this container view to expand and fill the entire viewing area. We also want it to work on any size screen, since we don't know what phone users will be using. So we need to add some constraints to our container view. First we go to the bottom and click the **Add Constraints** icon. This will show a small modal dialog. In here, we want to set all four constraint values to **0**. This will force the container view to expand and fill the area. Once we're done, just click **Add 4 constraints**. Make sure you are adding all four of them!

![Add 4 constraints](https://github.com/navisens/NaviDocs/blob/resources/Images/iOS/4.4.png)

Now we have to import the container view into our code. To do that, first we want to open up **split view**. Go to the top right hand corner, and click the second view mode. This will open up the **ViewController** file on the right hand side, while our **storyboard** will remain on the left side. Collapse any panels if you don't have enough room to see what's going on. If the right panel isn't a **ViewController**, you can click the top and manually select the desired file. Once we have both views open, we want to **hold Ctrl** key, and drag the **container view** into the code.

![Ctrl and drag](https://github.com/navisens/NaviDocs/blob/resources/Images/iOS/4.5.png)

A dialog will appear asking you for a name. We want to name this `mapContainer`, so we can access it later in the code.

![Rename mapContainer](https://github.com/navisens/NaviDocs/blob/resources/Images/iOS/4.6.png)

Now go back to **single view** by clicking the first button on the top right. We're gonna switch over to the `ViewController.swift` file now. In here, you should see that an **outlet** has automagically been created linking your `mapContainer` from the **storyboard**!

![Switch to view controller](https://github.com/navisens/NaviDocs/blob/resources/Images/iOS/4.7.png)

Here, we finally get to write some code! First, we want to import the `NavisensCore` and `NavisensMaps` plugins at the top. We'll create some variables to reference the `core` and the `maps`. Then we can add code to initialize the two variables. Just copy and paste the snippets below:

Add this to the top:

```swift
import NavisensCore
import NavisensMaps
```

Add this inside the class:

```swift
  let DEVKEY = "/* YOUR DEV KEY HERE */"
  private var core: NavisensCore?
  private var maps: NavisensMaps?
```

Add this at the end of `viewDidLoad()`

```swift
    core = NavisensCore(DEVKEY)
    maps = core!.add(NavisensMaps.self)?
      .useLocalOnly()
      .showPath().hideMarkers()
      .addTo(mapContainer)
```

![All the code](https://github.com/navisens/NaviDocs/blob/resources/Images/iOS/4.8.png)

Now, we can try running our app! Yes. That's all we needed to do for a fully-functional app. Amazing isn't it? Just hook up your iOS device, select your phone, and hit that run button.

And while it's compiling, you take another sip of your beer.

![Run the app](https://github.com/navisens/NaviDocs/blob/resources/Images/iOS/4.9.png)

## 5. The Demo

Here's what it looks like running inside our phone. It's gonna ask for confirmation to use location services first, so we should allow that.

![Location permissions](https://github.com/navisens/NaviDocs/blob/resources/Images/iOS/5.1.png)

Once we grant permissions, we'll see our current location at the origin.

![Our location](https://github.com/navisens/NaviDocs/blob/resources/Images/iOS/5.2.png)

*Hold my beer*, we're gonna **walk around** a bit. (Yes, in *real-life*). You should notice that your marker will follow a path now.

![Walk around](https://github.com/navisens/NaviDocs/blob/resources/Images/iOS/5.3.png)

And that's all! We have a working app now that tracks our location. For some more information, your heading is marked with **three** different colors. These colors represent how we think you are moving right now. While green, you should be walking or moving. If red, then you aren't moving, but you are still holding on your phone. Finally, while blue, your phone is stationary and no one is interacting with it (try setting it down on a table). The outline shows you the current movement, while the inner region is a shaded color representing the past second of movement. For the most part, it should be correct. No, we can't tell if you are drunk. If your marker isn't moving correctly, maybe that's why :P.

## 6. More Beer!

You're lonely in this empty desolate plane of white, and you wish for companionship. So you ask a friend to grab you another beer.

*Warning: The following parts require a second beer in order to successfully complete (and a friend to share that beer with). Make sure you have a second mobile device!*

We're gonna add one line of code to our app. This is a method of the **`maps`** object, so you should call it wherever you have access to that object.

```swift
.enableLocationSharing()
```

We can add it in between any of the other setup functions since these functions both are called on and return the `maps` reference.

Now, put the app on two separate mobile devices. You should now run the two apps.

For best results, make sure the two phones are at the **same location** (within a meter of each other), and are both **facing the same direction** (not facing each other).

Give your friend the second mobile device, possibly as a worthy trade for his beer. And now you can both walk around. If your companion and friend is just that second beer, consider walking with each phone separately, and moving them to different locations relative to each other. Notice that both phones should display their own location (with a large marker), along with the other phone (in a slightly smaller marker).

And now you have a working app! Thanks for drinking with us!

## 7. Even Moar Beer!!!

What? You still have more beer left? Okay then...

Here are some other cool things you can try.

* Remove the call to `useLocalOnly()`, and see what happens. You should notice that now (if you have internet connection), your phone will put you on a map and attempt to synchronize your location. You'll need to walk around a block before we figure out where exactly you are.
* You don't feel like walking around to get a location lock? No fear! Just call `addControls()` method call to your initialization. Now you can set your current location instead of waiting for it to lock! Click the gear icon and tap where you are. There are a LOT of interesting notes on this behavior, and we recommend you check out other tutorials to get a more in-depth feel for how initializing your location works.
* You're curious how you walk over time? Just remove the `hideMarkers()` call, and we'll also display a bunch of cool colored dots that display stats on how we think you've been walking around. Tap on any of the dots to get more stats!

If you've made it this far, then we thank you for trying out **Navisens MotionDna**. If not, well maybe you can just grab another beer.

*Disclaimer: Navisens in no way encourages or applauds coding while inebriated ... but a little beer never hurt anyone :P.*
