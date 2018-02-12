# Setting up MotionDNA Continuous Service Mode on Android

There are now 2 ways to run our SDK in the background on Android:

* ForegroundService: is the “proper” way to run a location app with the icon in the top corner signaling a service to the user
* BackgroundService: is an alternative for when your app cannot support a foreground service (e.g. for legacy reasons). 

Enabling the background service will allow our location SDK to continue running similar to having a foreground service enabled. Our recommendation is to use a foreground service whenever possible.

In order to use the service mode the following entry must be added to your application's `AndroidManifest.xml` file. 

```gradle
<service
    android:name="com.navisens.motiondnaapi.MotionDnaApplicationService"
    android:enabled="true"
    android:stopWithTask="true"
    >
```

Note: the `stopWithTask` attribute ensures that if the user terminates the app in the task bar that it will be recognized as a user directed action and not restart the service. If this is set to `false` then the service will restart even when the user kills the app in this manner.

Run the static method `MotionDnaApplication.setContinuousService(true)` or  `MotionDnaApplication.enableContinuousService()` prior to instantiation of MotionDnaApplication in your activity.

Sample Code:
```java
private MotionDnaApplication motionDnaApplication_;

protected void onCreate() {
  MotionDnaApplication.enableContinuousService();
  motionDnaApplication_ = new MotionDnaApplication(YourActivity.this);
	// ...
}

protected void onStart(Bundle savedInstanceState) {
  String devkey = “Your-Dev-Key”;
	motionDnaApplication_.runMotionDna(devkey);
}
```

When you instantiate `MotionDnaApplication` try to place some distance between it and the first API call to runMotionDna. Android needs time to bind the initialized service to the app for use. It is recommended to instantiate MotionDnaApplication in `onCreate` of your activity and make the first call to `runMotionDna` in the `onStart()` method.

Understand that if the service has restarted from background app termination then upon restart of your app your `receiveMotionDna` callback may start making updates immediately. Be prepared to check that any objects (particularly UI elements) that are updated through this listener are instantiated before the `receiveMotionDna` callback can update them following instantiation of `MotionDnaApplication` or check for `null` before updating any objects in the `receiveMotionDna` callback.
