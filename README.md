# NaviDocs
How does Navisens work? Tutorials, quick starts, and extensive documentation on all our newest and latest features!

*Not sure where to start? Check out the **Quickstart** for [Android](/BEER.Android.md) and [iOS](/BEER.iOS.md)!*

## Quick Links

**SDK Release notes**: [Android-SDK](https://github.com/navisens/Android-SDK/releases) | [iOS-SDK](https://github.com/navisens/iOS-SDK/releases)

**Quickstart**: The quick and easy way for beginners of mobile app development, and those unfamiliar with our Plugins system!

<sup>[Android](/BEER.Android.md) | [iOS](/BEER.iOS.md)</sup>

**Documentation**: The nitty-gritty of our SDK, with extensive details and examples.

<sup>[Android](/API.Android.md) | [iOS](/API.iOS.md)</sup>

**Plugins**: Our Plugins system for quickly testing out ideas or new ways of using our SDK. And all of it completley open source!

<sup>[Android](https://github.com/navisens/Android-Plugin) | [iOS](https://github.com/navisens/iOS-Plugin)</sup>

[**Tutorials**](/Tutorials): Some of our more advanced guides and workshops for getting the most out of MotionDna.

*Android:*
* **Recommended**: [Using and Making Plugins (for groups)](/Tutorials/navihelpme.Android.md)
* [Creating New NavisensPlugins](/Tutorials/creating-navisensplugins.Android.md)
* [Publishing NavisensPlugins](/Tutorials/publishing-navisensplugins.Android.md)

*iOS:*
* [Creating New NavisensPlugins](/Tutorials/creating-navisensplugins.iOS.md)
* [Publishing NavisensPlugins](/Tutorials/publishing-navisensplugins.Android.md)

-----

### Navisens Mini Tutorial/Quickstart (aka “The Beer Test”)
The purpose of this tutorial is to become familiar with the Navisens SDK in the time it takes you to drink approximately one standard size beer 🍺.

This tutorial will give you a quick overview of Navisens, the types of applications our technology supports, explain a few navigation concepts, and get right into building an app!

To get the most out of this mini tutorial, you should have some previous app development and beer drinking experience.

Short on beer? Skip the fine print and get straight to the code! Click [here if you want to develop for Android](/BEER.Android.md), or [here if you want to develop for iOS](/BEER.iOS.md).

### What is Navisens motionDNA?
Navisens motionDNA is a location platform designed to operate in all environments: indoors, outdoors, and underground, with zero setup and much more accurately than existing location systems. 

Navisens achieves this without forcing you to install any sensors or hardware such as beacons or WiFi Access Points (APs), without requiring you to collect data from the environment in advance (aka “fingerprinting”), and without having any type of calibration or set-up routine. All you need is a smartphone and the Navisens motionDNA SDK!

You can think of motionDNA as a modern GPS which works in urban environments and is designed to support modern use-cases and applications such as Augmented Reality (AR). Navisens motionDNA is available as a cross-platform Software Development Kit (SDK) for iOS and Android with the exact same capabilities and functionality on both platforms. To use the location capabilities of Navisens, you simply integrate the motionDNA SDK into your iOS or Android app.

For a quick test: [download our demo app](https://itunes.apple.com/us/app/navisens-indoor-location/id1224813390) and simply walk around your office or building, and watch the position update in real-time as a history of your previous positions is also visualized.

### How does Navisens motionDNA work?
Rather than relying on infrastructure such as beacons and WiFi APs which require a lot of overhead to install, calibrate, and maintain, Navisens leverages the accelerometers and gyroscopes built into mobile devices (these sensors are called “inertial sensors”). 

Navisens processes the data from the inertial sensors in a novel way so that an accurate position can be calculated, without requiring the placement of sensors in the environment. Navisens motionDNA continues tracking whether the phone is in your hand or in your pocket, and provides a low-latency position and orientation of the device, making it suitable both for AR applications and as a GPS “blue dot” replacement for regular applications.

### What are the limitations?
To provide as best performance as possible without relying on external hardware, Navisens is optimized for certain applications and use cases. Currently Navisens is optimized for pedestrian applications - these are applications based on tracking mobile devices carried by humans. Navisens is not currently optimized for drones, mobile robots, or other vehicles.

### Typical applications
The Navisens motionDNA SDK has been implemented in all types of applications by Fortune 100 companies to small startups. 

Shopping malls/retail: guiding users to stores and items within stores
Smart building/office: guiding staff and visitors to meeting rooms on large campuses
Industrial/facilities management: guiding workers to locate equipment and machinery inside large facilities and warehouses

### Known issues
Some smartphones can have problems with the inertial sensors. For iOS, the iPhone 5/5S has known sensor hardware issues (this was covered by the media as “sensor-gate”).

For Android, some devices may have issues due to hardware, and some due to software. In some cases, we have seen devices performing poorly (e.g. a Samsung Galaxy S4) perform much better after a firmware update. It’s best to always have the latest firmware update installed on your Android phone.

### Quick background of basic location concepts
Navisens supports two modes of positioning: relative positioning and global positioning. This tutorial provide code examples of both.

The simplest way to use the motionDNA SDK is in relative positioning mode. This mode provides location in a local Cartesian coordinate frame which provides positions in meters relative to a start position. By default the start position is set to (0, 0), however the start position and heading can be set to any arbitrary value, e.g. (35.00, 10.50). The important thing is to remember is that all position updates are relative to this start position. In this mode of operation, motionDNA has zero reliance on any global sensors. Our demo app on the [iOS App Store](https://itunes.apple.com/us/app/navisens-indoor-location/id1224813390) demonstrates this mode of operation.

The global positioning mode is designed as a direct replacement for GPS, and provides location coordinates in a global frame with positions provided as (latitude, longitude). To operate in this mode, a global (latitude, longitude) position must be set as a start position. There are several different ways this can be performed. It could be as simple as setting the start position to the GPS location provided by the phone, or by setting the start position based on a (latitude, longitude) which is associated with a beacon. The preferred method is to use a special feature of the motionDNA SDK whereby the start position is automatically determined by fusing information from the motionDNA relative trajectory along with global sensor information (GPS), after taking several measurements while the user is walking (requires the user to walk around 1-2 blocks outdoors). 

### Helping out Navisens!
To help us out, you can provide us with a dataset you have recorded which we will be able to analyze in our labs. To do this please follow these instructions:
* Choose a start point, and when you complete the dataset, make sure to return as close as possible to the start point so that the beginning and end position is the same.
* When walking, please follow a consistent pattern of walking rather than mixing walking and stopping or other motions. This helps us analyze the data. If you do need to stop, it’s nicer if it’s a distinct stop for several seconds before you start walking again.
* Walking a longer distance is preferred.
* Walking a path which can be measured relative to a reference point helps, e.g. walking the entire length of a building and then turn around and walk back the same path.
* Take a screenshot of the results you see in the app and send it to us
* If it’s easier, you can use our demo app on the iOS and Android app store. We recently updated the app with a tutorial and UI refresh to make it easier to set a start position on Google Maps. We also have a dataset upload button so you can easily send us the dataset. You can find the iOS app here https://itunes.apple.com/us/app/navisens-maps/id1155641105?mt=8 and the Android app here https://play.google.com/store/apps/details?id=navisens.motiondna
* Don’t forget to return back to the start point before you upload the dataset to us.
* Also if you include a general description (if you can remember) of what you performed, that can also help.
