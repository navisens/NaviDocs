# Getting Started on iOS #

## Dependencies ##
------------------

The can be included using the Cocoapods dependency manager. You can find installation instructions for cocoapods [here](https://cocoapods.org).

Create a `Podfile` in the base project folder with the following pods, and install them with ```pod install``` in the terminal from the project folder:

```pod
source 'https://github.com/CocoaPods/Specs.git'
platform :ios, '12.0'

target 'ProjectName' do
  use_frameworks!
  pod 'MotionDnaSDK'
end
```

## Permissions ##
-----------------
MotionDnaSDK will request the appropriate permissions for location and motion sensor updates once initialized. Additionally, if you need MotionDnaSDK to continue in the background, continue to the [Background Support](#background-support) section.

## Startup ##
-------------
### Instantiation ###
To setup the MotionDnaSDK, there are two objects you must create.

First, you need a class that impements the `MotionDnaSDKDelegate` protocol. This will receive all estimation updates from our SDK via these implemented callback methods.

Next, you need a new instance of `MotionDnaSDK` using the `MotionDnaSDK(delegate:)` contructor, and passing a `MotionDnaSDKDelegate` conforming object as the receiving delegate.

### Configuration ###
Before starting the MotionDnaSDK you have the choice to pass in a series of options for how it will run. These options can be passed in using a Dictionary of type [String:Any] with the string key as the option name and the value as its parameter. Valid configuration parameter type can be booleans, strings, integers, or doubles. However each option only uses a single type. You can find a detailed list of configuration options on in the [API Documentation here](API.iOS.md).


### Execution ###
Start the estimation by calling `start(withDeveloperKey: String, configurations: [String : Any])` on your MotionDnaSDK object and passing your developer key along with the prepared configuration dictionary as specified above. If you do not already have an SDK you can request one [here](https://navisens.com/#contact).

If you wish to start MotionDnaSDK without specifying configuration options you can call `start(developerKey: String)` instead. This will start estimation using a standard set of configurations expected to be most common for developers. These defaults can also be found in the [API Documentation](API.iOS.md).


## Background Support ##
------------------------
Using the SDK call above, you application will continue to acquire sensor and gps data for estimation only while the application remains in the foreground. In order to continue sensor acquisition when the application is backgrounded, you should select the *Location Updates* and *Background Fetch* options from the available background modes for your applications target. Then simple add the `enableBackgroundEstimation()` method call following startup of the MotionDnaSDK.
