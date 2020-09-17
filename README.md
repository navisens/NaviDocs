## Introduction to Navisens

### What is Navisens motionDNA?
[Navisens](https://www.navisens.com) motionDNA is a location SDK for iOS, Android, and web platforms which provides an accurate location in any environment without installing sensors or collecting data. Navisens uses the motion sensors built into devices and can work indoors and automatically integrate with GPS outdoors.

The Navisens SDK provides a 3D location in global (latitude, longitude, altitude) and local (x, y, z) coordinates. The SDK also provides contextual information about the motion the user is performing, the mode of transport, whether the user is indoors or outdoors, and the orientation of the device. 

### Background
Navisens is a unique SDK which has been built from years of research in robotics and development as a startup. [Read our blog post to learn about the background of Navisens](https://www.linkedin.com/pulse/why-i-founded-navisens-ashod-donikian)

### The Navisens basics
The Navisens SDK is a cross-platform native SDK for iOS and Android with the exact same functionality and capabilities for both platforms. [A web version of the Navisens SDK is coming soon](https://forms.gle/P8gWA6EobyzYF6ZYA) which will allow running web apps directly inside the browser without requiring an app download.

The Navisens SDK provides the following information in real-time:
1. Location: accurate location indoors, underground, or outdoors, in global (latitude, longitude, altitude) or local (x, y, z) coordinates. The Navisens SDK provides the developer full control, allowing the host app to set and override the current location at any time. Navisens can also automatically integrate with GPS to set a reference position and periodically perform location updates.
2. Orientation: the complete orientation of the device represented as Euler angles and a quaternion. 
3. Motion and statistical information: contextual information about the motion the user is performing and their current activity and location. This includes information about the motion the user is performing (walking, fidgeting while sitting or standing, or stationary with the device placed down on a fixed surface), the mode of transport (pedestrian or vehicle), and the environment (indoors or outdoors). The Navisens SDK also provides the time spent and distance moved in each of these states, and a statistical breakdown of the percentage of time spent in each of these states.

### Requirements
- Hardware: Navisens requires an accelerometer and gyroscope. Be default, Navisens uses the sensors internal to your device and will report if a sensor is missing. A barometer is optional but required for 3D location. A magnetometer is optional but provides additional information to our sensor fusion algorithms.
- Operating system: minimum iOS 12, Android minimum SDK version 21

### FAQ
- **Does Navisens provide an app?** The Navisens SDK is designed to be integrated into an app. We provide examples to help you get familiar with our SDK and we provide support to help you integrate our SDK into your app, however we do not provide a customer-facing app.
- **Do I need a map?** The Navisens SDK functions independent of a map and continually provides location coordinates indoors and outdoors with or without a map. It’s up to the host app to visualize the location provided by the Navisens SDK, and this may require a map for the user experience. We provide examples demonstrating how to integrate our SDK with popular map platforms and we have partnerships in place with companies which specialize in creating custom maps. Please contact us for more information regarding our map partnerships.
- **Does Navisens support my custom device?** Navisens currently supports smartphones and select smartglasses via our API which supports external inertial sensor data input. We currently do not provide support for custom hardware and applications such as drones, mobile robots, or other vehicles.

## Navisens SDK Documentation

**Release Notes**
- [Android](https://github.com/navisens/Android-SDK/releases)
- [iOS](https://github.com/navisens/iOS-SDK/releases)

**API Reference**
- [Android](/API.Android.md)
- [iOS](/API.iOS.md)

**Getting Started Guide**
- [Android](/GettingStarted.Android.md)
- [iOS](/GettingStarted.iOS.md)

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

## Demo Apps
- [Demo app for iOS](https://itunes.apple.com/us/app/navisens-maps/id1155641105?mt=8)
- [Demo app for Android](https://play.google.com/store/apps/details?id=navisens.motiondna)
