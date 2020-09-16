# Navisens SDK Documentation

How does Navisens work? Extensive documentation on all our newest and latest features!

**Android:**
- [Release Notes](https://github.com/navisens/Android-SDK/releases)
- [Getting Started](/GettingStarted.Android.md)
- [API](/API.Android.md)

**iOS:**
- [Release Notes](https://github.com/navisens/iOS-SDK/releases)
- [Getting Started](/GettingStarted.iOS.md)
- [API](/API.iOS.md)

## Quick Start Examples
Hello world examples below contain thoroughly documented code examples along with common use case code snippets in the project readme.



**Android:**
* [Hello World (Java)](https://github.com/navisens/android-app-helloworld)
* [Hello World (Kotlin)](https://github.com/navisens/android-app-helloworld-kotlin)
* [Map Manual Location Setting (Java)](https://github.com/navisens/android-app-map-manual-location)

**iOS:**
* [Hello World (Objective-C)](https://github.com/navisens/iOS-app-helloworld)
* [Hello World (Swift)](https://github.com/navisens/iOS-app-helloworld-swift)
* [Hello World (SwiftUI)](https://github.com/navisens/ios-app-helloworld-swiftui)
* [Google Maps Integration (Swift)](https://github.com/navisens/ios-googlemaps-motiondna-integration)

-----

### Navisens prerequisites

Our SDK works on the majority of smartphones (both iOS and Android). You can view below the required sensors and operating system versions to ensure proper functioning of our SDK, we also have a method callback which reports any hardware problems: [reportError](https://github.com/navisens/NaviDocs/blob/master/API.Android.md#void-reporterrorerrorcode-errorcode-string-errordescription).

#### Hardware

We require at least an accelerometer and a gyroscope for our inertial estimation.

##### Barometer

If the phone is	equipped with a	barometer our estimation will be in 3D, which means we will report the altitude.

#### Software

##### iOS

We require at least iOS 12.0.

##### Android

Minimum SDK is 21.

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

### Quick background of basic location concepts
Navisens supports two modes of positioning: relative positioning and global positioning. This tutorial provide code examples of both.

The simplest way to use the motionDNA SDK is in relative positioning mode. This mode provides location in a local Cartesian coordinate frame which provides positions in meters relative to a start position. By default the start position is set to (0, 0), however the start position and heading can be set to any arbitrary value, e.g. (35.00, 10.50). The important thing is to remember is that all position updates are relative to this start position. In this mode of operation, motionDNA has zero reliance on any global sensors. Our demo app on the [iOS App Store](https://itunes.apple.com/us/app/navisens-indoor-location/id1224813390) demonstrates this mode of operation.

The global positioning mode is designed as a direct replacement for GPS, and provides location coordinates in a global frame with positions provided as (latitude, longitude). To operate in this mode, a global (latitude, longitude) position must be set as a start position. There are several different ways this can be performed. It could be as simple as setting the start position to the GPS location provided by the phone, or by setting the start position based on a (latitude, longitude) which is associated with a beacon. The preferred method is to use a special feature of the motionDNA SDK whereby the start position is automatically determined by fusing information from the motionDNA relative trajectory along with global sensor information (GPS), after taking several measurements while the user is walking (requires the user to walk around 1-2 blocks outdoors).

### Cartesian Coordinate System

The Navisens SDK is constantly estimating in two frames: cartesian and geodesic.

Here I'll be talking about our cartesian coordinate system. When a user is walking, every
estimate outputs a distance and a heading delta with respect to the previous estimate.
As we keep receiving these distances and headings we integrate them, which give us xy coordinates.

When the SDK starts, the user is facing 90 degrees in our local frame of reference. The angle
orientation grows counter clockwise.
Assuming the user is 90 degrees and walks straight the Y axis will grow, then when the user
turns right the X axis will grow.

The system can also be referred to a right handed coordinate system.
