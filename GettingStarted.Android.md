# Getting Started on Android

## Setup Dependencies ##

Here is what you need to set up the Navisens MotionDnaSDK.

Add the following dependency to your application buildscript:

```gradle
dependencies {
    implementation group: "com.navisens", name: "motiondnaapi", version: "2.0.0", changing: true
    // ...
}
```

Replace the version with whichever version of the SDK you wish to use.

# Configure SDK #
-----
To setup the MotionDnaSDK, there are two objects you must create.

First, you need a new class which implements `MotionDnaSDKListener`. This will receive all callbacks from our SDK, serving new data through the implemented callback functions.

Next, you need to instantiate a new instance of `MotionDnaSDK`, passing in as an argument an instantiated `MotionDnaSDKListener` type from above as well as a application context to make sensor requests.

The `MotionDnaSDK` instance serves as the main controller of the SDK, and is used to alter the state of the SDK, while `MotionDnaSDKListener` is used solely as an event listener for estimation results and status updates.