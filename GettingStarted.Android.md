# Getting Started on Android

## Dependencies ##

To include MotionDnaSDK in your project, add the following dependency to your application buildscript (build.gradle):

```gradle
dependencies {
    implementation group: "com.navisens", name: "motiondnaapi", version: "2.0.0", changing: true
    // ...
}
```

## Permissions ##

MotionDnaSDK needs the following permissions added to the `AndroidManifest.xml` to allow them to be used and/or requested in app as needed.

``` xml
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
    <uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
    <uses-permission android:name="android.permission.CHANGE_WIFI_STATE" />
    <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
    <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
```

Some of these permissions will need to be requested from the main app activity and approved by the user. By calling `getRequiredPermissions()` you can retrieve a list of permissions that are needed to run the MotionDnaSDK. This request should be made through the `requestPermissions()` method on the Activity or ActivityCompat that the SDK is running in.

Additionally, if you need MotionDnaSDK to continue in the background, continue to the [Background Support](#background-support) section.

## Startup ##
-----

### Instantiation ###
To setup the MotionDnaSDK, there are two objects you must create.

First, you need a class that implements `MotionDnaSDKListener` interface. This will receive all callbacks from our SDK, serving new data through the implemented callback functions.

Next, you need to instantiate a new instance of `MotionDnaSDK`, passing in as an argument an instantiated `MotionDnaSDKListener` type from above as well as a application context to make sensor requests.

The `MotionDnaSDK` instance serves as the main controller of the SDK, and is used to alter the state of the SDK, while `MotionDnaSDKListener` is used solely as an event listener for estimation results and status updates.

### Configuration ###
Before starting the MotionDnaSDK you have the choice to pass in a series of options for how it will run. These options can be passed in using a `HashMap<String,Object>` with the string key as the option name and the value as its parameter. Valid configuration parameter types include booleans, strings, integers, or doubles. However each option only uses a single type. You can find a detailed list of configuration options in the [API Documentation here](API.Android.md).


### Execution ###
Start the estimation by calling `start(String developerKey, HashMap<String, Object> configurations)` on your MotionDnaSDK object and passing your developer key along with the prepared configuration dictionary as specified above. If you do not already have an SDK you can request one [here](https://navisens.com/#contact).

If you wish to start MotionDnaSDK without specifying configuration options you can call `start(String developerKey)` instead. This will start estimation using a standard set of configurations expected to be most common for developers. These defaults can also be found in the [API Documentation](API.Android.md).



## Background Support ##
------------------------
Using the SDK call above, you application will usually continue to estimate in the background for a period of time. However, the OS reserves the right to shut it down if memory or CPU usage gets too high, and there is no guarantee that sensor input from the gyroscope and accelerometer will be continuous, resulting in potential errors in the estimation.

To mitigate this we execute the `startForegroundService()` method just prior to starting the SDK with `start()`. This, coupled with the addition of
```xml
<uses-permission android:name="android.permission.FOREGROUND_SERVICE" />
```
and
```xml
<application>
...
  <service android:name="com.navisens.motiondnaapi.MotionDnaForegroundService"
      android:foregroundServiceType="location"
      />
</application>
```
 to the application `AndroidManifest.xml` creates a permanent message in the notification tray while the app is running and indicates to the OS that the app should still maintain priority on resources and sensor data that is comparable to any app that is running in the forground. If you are already using a forground service this method should not be necessary.