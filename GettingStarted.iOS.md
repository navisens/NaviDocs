# Getting Started on iOS

## Setup Dependencies ##

Here is what you need to set up the Navisens MotionDnaSDK.

The SDK is easily installed using the Cocoapods dependency manager. You can find installation instructions for cocoapods [here](https://cocoapods.org).

Create a `Podfile` with the following pods, and install them with ```pod install``` in the terminal from the project folder:

```pod
source 'https://github.com/CocoaPods/Specs.git'
platform :ios, '12.0'

target 'ProjectName' do
  use_frameworks!
  pod 'MotionDnaSDK'
end
```

# Configure SDK #
-----
To setup the MotionDnaSDK, there are two objects you must create.

To setup the MotionDnaSDK, you must implement the `MotionDnaSDKDelegate` protocol.  This will receive all callbacks from our SDK, serving new data through the implemented callback functions.

Next, you need to instantiate a new instance of `MotionDnaSDK`, passing in as an argument an instantiated `MotionDnaSDKDelegate` type from above.

The `MotionDnaSDK` instance serves as the main controller of the SDK, and is used to alter the state of the SDK, while `MotionDnaSDKDelegate` is used solely as an event listener for estimation results and status updates.