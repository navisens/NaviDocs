# NaviDocs/Android #
-----

This is the central Android API documentation for the Navisens MotionDnaSDK.

## Installation ##

If you haven't already done so or haven't build an Android app before, check out the [Quick Start Guide](/BEER.Android.md), as it is designed to be a very straight-forward demo of an easy app you can build with our SDK and tools. Otherwise, if you already have some experience building Android apps, here's what you need to do to set up the Navisens MotionDnaSDK.

Add the following dependencies to your application buildscript:

```gradle
dependencies {
    implementation group: "com.navisens", name: "motiondnaapi", version: "1.7.0", changing: true
    // ...
}
```

Replace the version with whichever version of the SDK you wish to use.

# API #
-----
To setup the MotionDnaSDK, there are two objects you must create.

First, you need create a new class which implements `MotionDnaInterface`. This will serve as the object which receives all callbacks from our SDK, serving new data through the callback functions.

Next, you need to instantiate a new instance of `MotionDnaApplication`, passing in as an argument an instantiated `MotionDnaInterface` type from above.

It is important to note that `MotionDnaApplication` serves as the main controller of the SDK, and is used to alter the state of the SDK, while `MotionDnaInterface` is used solely as an event listener.

Below, we will organize the API into several sections. First, the **Setup** and **Control** functions are called through `MotionDnaApplication`. **Setup** involves tasks that must be completed for our SDK to function. **Control** includes any functions that alter the behavior of our SDK, along with retrieving some basic device information. We also have the **Callbacks**, which are configured in `MotionDnaInterface`. **Callbacks** must all be implemented, but don't necessarily need to do anything. Some of the callbacks are invoked with an event object of type `MotionDna`. In the **Getters** section, we'll explain what information you can extract from these event objects.
 * [Setup](#setup)
 * [Control](#control)
 * [Callbacks](#callbacks)
 * [Getters](#getters)

-----
## Setup ##

These are functions that must be called in order to configure the SDK completely.

* [`MotionDnaApplication(MotionDnaInterface)`](#motiondnaapplicationmotiondnainterface)
* [`void runMotionDna(String devkey)`](#void-runmotiondnastring-devkey)

-----
#### `MotionDnaApplication(MotionDnaInterface)`
This binds your `MotionDnaInterface` to the SDK, so you can receive event information.

**Params**
The parameter is an instance of the custom class you created which implements `MotionDnaInterface`.

Make sure your `MotionDnaInterface` implements all the required methods. There are two required methods (shown below) that further bind the interface to your application's activity, so MotionDna is able to retrieve the necessary permissions and capabilities. They should be implemented as follows:

```java
@Override
    public Context getAppContext() {
        return getApplicationContext();
}

@Override
    public PackageManager getPkgManager() {
        return getPackageManager();
}
```

#### `void runMotionDna(String devkey)`
This functions starts up the SDK. You must pass in a valid developer's key in order for the SDK to function. IF the key has expired or there are other errors, you may receive those errors through the [`reportError()`](#void-reporterrorerrorcode-errorcode-string-errordescription) callback route.

**Params**
The paramater is your developer key. If this is missing, or incorrect, the SDK will cease to function.

-----
## Reducing Android SDK Size ##


Our SDK internally handles 3 cpu architectures to provide as much hardware support as possible:
- arm 64bit
- arm 32bit
- x86

If you don’t require support for all CPU architectures, you can strip out certain CPU architectures thus reducing the SDK size.
You can do so by adding the following lines of code to your `build.gradle` file.

Example for supporting arm 32bit cpu architecture (note: all phones that have arm 64bit will support arm 32bit):
```gradle
buildTypes {
    release {
        ndk {
            abiFilters "armeabi-v7a"
        }
    }
```

For supporting arm 64 bit cpu architecture (this will not work on arm 32bit cpus):
```gradle
buildTypes {
    release {
        ndk {
            abiFilters "arm64-v8a"
    	}
    }
```

For supporting arm 64 bit & arm 32bit cpu architectures:
```gradle
buildTypes {
    release {
        ndk {
            abiFilters "arm64-v8a", "armeabi-v7a"
	}
    }
```


-----
## Control ##

There are many ways to control the SDK. Here, we have split control methods into several sections.

* Device Information
	* [`String MotionDnaApplication.checkSDKVersion()`](#string-motiondnaapplicationchecksdkversion)
	* [`String getDeviceID()`](#string-getdeviceid)
	* [`String getDeviceModel()`](#string-getdevicemodel)
* SDK Control
	* [`void resume()`](#void-resume)
	* [`void pause()`](#void-pause)
	* [`void stop()`](#void-stop)
	* [`void resetLocalEstimation()`](#void-resetlocalestimation)
	* [`void resetLocalHeading()`](#void-resetlocalheading)
	
* SDK Settings
	* [`void setARModeEnabled(boolean mode)`](#void-setarmodeenabledboolean-mode)
	* [`void setBackpropagationEnabled(boolean state)`](#void-setbackpropagationenabledboolean-state)
	* [`void setBackpropagationBufferSize(int size)`](#setbackpropagationbuffersizeint-size)
	* [`void setBinaryFileLoggingEnabled(boolean state)`](#void-setbinaryfileloggingenabledboolean-state)
	* [`void setCallbackUpdateRateInMs(double rate)`](#void-setcallbackupdaterateinmsdouble-rate)
	* [`void setNetworkUpdateRateInMs(double rate)`](#void-setnetworkupdaterateinmsdouble-rate)
	* [`void setPowerMode(PowerConsumptionMode mode)`](#void-setpowermodepowerconsumptionmode-mode)
* Global Location Initialization
	* [`void setLocationNavisens()`](#void-setlocationnavisens)
	* [`void setLocationGPSOnly()`](#void-setlocationgpsonly)
	* [`void setLocationLatitudeLongitude(double lat, double lng)`](#void-setlocationlatitudelongitudedouble-lat-double-lng)
	* [`void setLocationLatitudeLongitudeAndHeadingInDegrees(double lat, double lng, double angle)`](#void-setlocationlatitudelongitudeandheadingindegreesdouble-lat-double-lng-double-angle)
	* [`void inputGlobalCoordinates(double lat, double lon, double timestamp)`](#void-inputglobalcoordinatesdouble-lat-double-lon-double-timestamp)
	* [`void inputGlobalCoordinates(double lat, double lon, double timestamp, double angleThreshold)`](#void-inputglobalcoordinatesdouble-lat-double-lon-double-timestamp-double-anglethreshold)
	* [`void setHeadingMagInDegrees()`](#void-setheadingmagindegrees)
	* [`void setHeadingInDegrees(double heading)`](#void-setheadingindegreesdouble-heading)
	* [`void setCartesianOffsetInMetersXY(double x, double y)`](#void-setcartesianoffsetinmetersxydouble-x-double-y)
	* [`void setLocalHeadingOffsetInDegrees(double heading)`](#void-setlocalheadingoffsetindegreesdouble-heading)
	* [`void setLocalHeading(double heading)`](#void-setlocalheadingdouble-heading)
	* [`void setCartesianPositionXY(double x, double y)`](#void-setcartesianpositionxydouble-x-double-y)
* Location Sharing
	* [`void startUDP(...)`](#void-startudp)
	* [`void stopUDP()`](#void-stopudp)
	* [`void setUDPRoom(String room)`](#void-setudproomstring-room)
	* [`void sendUDPPacket(String msg)`](#void-sendudppacketstring-msg)
	* [`void sendUDPQueryRooms(String[] rooms)`](#void-sendudpqueryroomsstring-rooms)

-----
#### `String MotionDnaApplication.checkSDKVersion()`
Get the SDK version used to build the application.

**Returns**
A string representing the current SDK version used in the app.

#### `String getDeviceID()`
Get a unique id for the current device.

**Returns**
A string id of the current device, which will always be the same.

#### `String getDeviceModel()`
Get the model of the device represented as a string.

**Returns**
A string representing the device model.

-----
#### `void resume()`
Resume estimation of current location, if it was paused in the past.

#### `void pause()`
Pause estimation of the current location. Estimation can be resumed at a later time by calling `resume()`.

#### `void stop()`
Completely shut down all estimation in our SDK, and release any resources that are in-use.

#### `void resetLocalEstimation()`
Resets the cartesian coordinates back to the origin `(0, 0)`, and sets the heading to `0` degrees.

#### `void resetLocalHeading()`
Resets just the heading to `0` degrees.

-----
#### `void setARModeEnabled(boolean mode)`
Enables AR mode. AR mode publishes orientation quaternion at a higher rate.

**Params**
Enable or disable AR mode.

#### `void setBackpropagationEnabled(boolean state)`
When setLocationNavisens is enabled and setBackpropagationEnabled is called, once Navisens has initialized you will not only get the current position, but also a set of latitude longitude coordinates which lead back to the start position (where the SDK/App was started). This is useful to determine which building and even where inside a building the person started, or where the person exited a vehicle (e.g. the vehicle parking spot or the location of a drop-off).

The historical trajectory of points will be ordered as follows: Point(t-n), Point(t-2), Point(t-1), Point(t-0)
So you will receive all the points from the past with the appropriate timestamps, until you get to your current position.

**Params**
Enable or disable back propagation.

#### `void setBackpropagationBufferSize(int size)`

If the user wants to see everything that happened before Navisens found an initial position,
he can adjust the amount of the trajectory to see before the initial position was set automatically.

To correlate the back propagation buffer in time you can measure it based on the POWER_MODE
you set: PERFORMANCE/LOW/MEDIUM.
Back propagation buffer is set to 4000 by default:
In low power:
4000 / 16: 250 seconds
Medium:
4000 / 8: 500 seconds
Performance:
4000 / 4: 1000 seconds

**Params**
Back propagation buffer size.

#### `void setBinaryFileLoggingEnabled(boolean state)`
Allow our SDK to record data and use it to enhance our estimation system.

**Params**
Enable or disable file logging.

##### Tutorial: Downloading Logs
> Downloading these logs is a very good way of providing debug information, should there be an issue with our SDK. On Android, this is very easily done. First, make sure you have the "Android Debug Bridge" or `adb` Android Platform Tool downloaded. Once you have it set up correctly, plug in your Android device to a computer. On the computer, open a command terminal and run the following command:
> 
> `adb backup -f myAndroidBackup.ab navisens.motiondna`
> 
> Select "Backup My Data" and this will begin an unencrypted bakcup of the navisens app package onto your computer.
> 
> Enter the following command to decompress the navisens app backup
>
> `python -c "import zlib,sys;sys.stdout.write(zlib.decompress(sys.stdin.read()))" > myAndroidBackup.tar; tar xvf myAndroidBackup.tar`
>
> This will give you access the the app storage file structure. Logs can be found under the "files/Logs" or "f/Logs" folder and subfolder

#### `void setCallbackUpdateRateInMs(double rate)`
Tell our SDK how often to provide estimation results. Note that there is a limit on how fast our SDK can provide results, but usually setting a slower update rate improves results. Setting the rate to `0ms` will output estimation results at our maximum rate.

**Params**
The number of milliseconds `ms` between each update. Setting this to 5000 will publish estimation results every 5 seconds, while setting this to 0 will publish results at the fastest rate our SDK is capable of running estimations (though not consistently timed).

#### `void setNetworkUpdateRateInMs(double rate)`
Tell our SDK how often to publish our location to your servers. This allows your device to receive MotionDna information on other devices on the same network.

**Params**
The number of milliseconds `ms` between each network update. Setting this to 5000 will send network estimation results every 5 seconds, while setting this to 0 will send results at the fastest rate our SDK is capable of running estimations (though not consistently timed).

#### `void setPowerMode(PowerConsumptionMode mode)`
Set the power consumption mode to trade off accuracy of predictions for power saving.

**Params**
The `PowerConsumptionMode` is one of the following:

* `SUPER_LOW_CONSUMPTION`: Batches samples for 2 minutes and processes them queued up and internal estimation runs at 6Hz.
* `LOW_CONSUMPTION`: Internal Estimation at 6.25Hz
* `MEDIUM_CONSUMPTION`: Internal Estimation at 12.5Hz
* `PERFORMANCE`: Internal Estimation at 25Hz

-----
#### `void setLocationNavisens()`
Use our internal algorithm to automatically compute your location and heading by fusing inertial estimation with global location information. This is designed for outdoor use and will not compute a position when indoors. Solving location requires the user to be walking outdoors. Depending on the quality of the global location, this may only require as little as 10 meters of walking outdoors.

While we are computing the user's location, you can use the `locationStatus` of [`getLocation()`](#location-getlocation) to determine when an accurate location has been determined.

#### `void setLocationGPSOnly()`
Wait for the next GPS reading, and set the latitude and longitude to that reading.

Given the limitations of GPS, it is best to use this only if you can guarantee the user has accurate GPS readings, and is outside, or you do not need a very accurate position.

#### `void setLocationLatitudeLongitude(double lat, double lng)`
Manually set only the global latitude and longitude.

Use this if you have other sources of information (for example, user-defined address), and need readings more accurate than GPS can provide.

**Params**
The latitude `lat` and longitude `long`.

#### `void setLocationLatitudeLongitudeAndHeadingInDegrees(double lat, double lng, double angle)`
Manually sets the global latitude, longitude, and heading. This enables receiving a latitude and longitude instead of cartesian coordinates.

Use this if you have other sources of information (for example, user-defined address), and need readings more accurate than GPS can provide.

**Params**
The latitude `lat` and longitude `long`, and heading `head` in degrees.

#### `void setHeadingMagInDegrees()`
Wait for the next magnetic compass reading, and set the heading to that reading.

Given the limitations of the magnetic compass, don't use this unless you are sure the device supports readings at high accuracy.

#### `void setHeadingInDegrees(double heading)`
Manually set the initial global heading

It is advised if you are using global heading, to manually set this, or allow the user to set this.

**Params**
The `heading`, as a rotation in degrees.

#### `void inputGlobalCoordinates(double lat, double lon, double timestamp)`
This method allows you to feed in geodetic coordinates from an external positioning system (e.g. beacons/wifi) and running our internal algorithms to find an initial start point. The method expects: latitude, longitude and timestamp (in seconds since boot time).

**Params**
The latitude and longitude coordinates with the associated timestamp.

#### `void inputGlobalCoordinates(double lat, double lon, double timestamp, double angleThreshold)`
Same as the above method except with an angular threshold input in degrees (the default is 7.5 degrees). The angleThreshold specifies the minimum required accuracy for Navisens to initialize the position and heading. The lower the value the more accurate the initialization and longer the initialization period. The higher the value the faster the initialization and the lower the accuracy as a tradeoff for speed.

**Params**
The latitude and longitude coordinates and timestamp with the desired angular threshold in degrees.

#### `void setCartesianOffsetInMetersXY(double x, double y)`
Adds an offset to both x and y axes. So if you are running the SDK in the local XY frame, you can start at custom XY positions.

**Params**
The `x` and `y` offset in meters.

#### `void setLocalHeadingOffsetInDegrees(double heading)`
Sets the local heading offset in degrees. The heading offsets you input get accumulated. You can reset the offset by calling `resetLocalHeading`.

**Params**
The `heading`, as a counterclockwise rotation in degrees (in +- 180 frame).

#### `void setCartesianPositionXY(double x, double y)`
Sets the Cartesian position in XY to the exact x and y passed in.

**Params**
The x and y coordinates to set.

#### `void setLocalHeading(double heading)`
Sets the local heading the exact heading value passed in.

**Params**
The `heading`, as a counterclockwise rotation in degrees (in +- 180 frame).

##### Tutorial: Running the SDK in the local frame.
> To run our SDK without any global reference, its simple, you only need to call `runMotionDna`, without any global input call. Since the system doesn't receive any user input or activation of the Navisens automatic initialization input (e.g. `setLocationNavisens`), it assumes the estimation is being run in the local XY frame.
> When receiving user user motionDna, you can access the `localLocation` and `localHeading` attributes.

-----
#### `void startUDP(...)`
Connect to your own server and specify a room. Any other device connected to the same room and also under the same developer will receive any udp packets this device sends.

This device will automatically begin broadcasting the `MotionDna` to your servers, so other devices will receive positional data. Using the [`void sendUDPPacket(String msg)`](#void-sendudppacketstring-msg) call will allow sending arbitrary strings of data, although it is recommended to use only safe and secure characters if communicating with devices of differing OS.

All messages and data are encrypted and sent securely; the server is unable to decode anything - only clients can read and interpret messages.

There are multiple versions of this method as follows:
* `void startUDP()`
* `void startUDP(String room)`
* `void startUDP(String room, String host, String port)`
* `void startUDPHostAndPort(String host, String port)`

For each version, if `host` and `port` is missing, a default public server will be used. Note that the public server has a limited amount of space, and may not always be available for use, so it is best to only use it for testing and instead use your own server for production.

If `room` is not provided, the default room will be used. Also note that the default room has a maximum number of device connections provisioned for it, so this room should only be used for testing and not any formal usage.

Recommendation:
Use `startUDPHostAndPort(String host, String port)` for internal testing, and only use `startUDP(String room, String host, String port)` for deployment release build.

#### `void stopUDP()`
Stop broadcasting positional data to your servers.

#### `void setUDPRoom(String room)`
Change the current 

#### `void sendUDPPacket(String msg)`
Send a string of data that will be broadcast to all other devices also in the same server room.

Data is encrypted and sent securely. It is recommended that data use only safe and secure characters to reduce chances of failure.

To receive the messages from other devices, see [`receiveNetworkData()`](#void-receivenetworkdatanetworkcode-opcode-map-payload)

#### `void sendUDPQueryRooms(String[] rooms)`
Query the server for the current capacity of the listed rooms. Note that the server was designed so that you cannot retrieve a list of rooms without modifying the server. This is to encourage privacy through obscurity.

To receive the results of a query, see [`receiveNetworkData()`](#void-receivenetworkdatanetworkcode-opcode-map-payload)

-----

## Callbacks ##

Callbacks are received from the implemented `MotionDnaInterface` class, and are each passed relevant event information in each listener method. All callback methods must be implemented, though they don't necessarily need to do anything.

* [`void receiveMotionDna(MotionDna motionDna)`](#void-receivemotiondnamotiondna-motiondna)
* [`void receiveNetworkData(MotionDna motionDna)`](#void-receivenetworkdatamotiondna-motiondna)
* [`void receiveNetworkData(NetworkCode opcode, Map<> payload)`](#void-receivenetworkdatanetworkcode-opcode-map-payload)
* [`void reportError(ErrorCode errorCode, String errorDescription)`](#void-reporterrorerrorcode-errorcode-string-errordescription)
* [`Context getAppContext()`](#context-getappcontext)
* [`PackageManager getPkgManager()`](#packagemanager-getpkgmanager)

-----
#### `void receiveMotionDna(MotionDna motionDna)`
This event receives the estimation results using a `MotionDna` object. Check out the [Getters](#getters) section to learn how to read data out of this object.

#### `void receiveNetworkData(MotionDna motionDna)`
This event receives estimation results from other devices in the server room. In order to receive anything, make sure you call `startUDP` to connect to a room. Again, it provides access to a `MotionDna` object, which can be unpacked the same way as above.

If you aren't receiving anything, then the room may be full, or there may be an error in your connection. See the `reportError` event below for more information.

#### `void receiveNetworkData(NetworkCode opcode, Map<> payload)`
This event receives arbitrary data from the server room. You must have called `startUDP` already to connect to the room.

**Params**
An `opcode` it passed in which identifies the type of data being passed. There are four valid opcodes:

* `RAW_NETWORK_DATA`
This event is received whenever any other device uses `sendUDPPacket` to broadcast a message.

The `payload` contains two items. The first item has key `ID`, and is a String containing the device ID of the sending device. The second item has key `payload` is a String containing the message that was sent.

* `ROOM_CAPACITY_STATUS`
This event is received whenever the current device makes a request for room status using the `sendUDPQueryRooms` method.

The `payload` contains many items. Each item has key equal to the room name queried, and the value is an Integer which is the number of connections inside a room.

We understand that there is no way to determine all available rooms. This is intended behavior in order to maintain privacy. The server does not know who owns which rooms, so there is also a matter of security here.

There is no way of getting the maximum number of connections allowed per room. If you exceed the number of rooms, or the number of connections, a different opcode is received to notify the application. In the future, we may add a way of querying the max number of connections, if it is useful enough. Alternatively, the server is open-source, so you may also write your own server which returns more information.

* `EXCEEDED_ROOM_CONNECTION_CAPACITY`
This event is received whenever an application attempts to create a new room, but no remaining rooms are available. Note that this event is received exactly once for each request to connect to a room using `startUDP()` or `setUDPRoom()`. Since the connection uses UDP, it is recommended that the application has some way of recovering from this invalid state even if a packet is dropped.

* `EXCEEDED_SERVER_ROOM_CAPACITY`
This event is received whenever an application attempts to broadcast to a room, but the server room is already full. Note that the room will not connect, and thus the next time another update is sent to the room, this error event will be received again. This event will thus be received at the same rate as the rate set by `setNetworkCallbackRateInMs()`. It is recommended that the application has some way of recovering from this invalid state.

#### `void reportError(ErrorCode errorCode, String errorDescription)`
Report any errors of the estimation or internal SDK. There are five possible errors as follows:
* `ERROR_SENSOR_TIMING`
* `ERROR_AUTHENTICATION_FAILED`
* `ERROR_SENSOR_MISSING`
* `ERROR_SDK_EXPIRED`
* `ERROR_PERMISSION`

The errors are self-explanatory, and the error description can be printed to gain further information on what caused the error. The most important error that should be handled is the permissions error, as this is the only error that is really relevant to the user. The other errors may be handled by simply shutting down the SDK.

#### `Context getAppContext()`
This callback is used by the SDK to get the application context of the current activity. Make sure this is valid, so the SDK is able to access the sensors on the device.

#### `PackageManager getPkgManager()`
This callback is used by the SDK to get the package manager of the current activity. Make sure this is valid, so the SDK will be able to access the device's file directory.

-----

## Getters ##

These are methods of the `MotionDna` object, and are used to get data from these event objects returned from the callbacks `receiveMotionDna()` and `receiveNetworkData()`.

* [`Attitude getAttitude()`](#attitude-getattitude)
* [`Location getLocation()`](#location-getlocation)
* [`Motion getMotion()`](#motion-getmotion)
* [`String getID()`](#string-getid)
* [`String getDeviceName()`](#string-getdevicename)
* [`MotionStatistics getMotionStatistics()`](#motionstatistics-getmotionstatistics)
* [`OrientationQuaternion getOrientationQuaternion()`](#orientationquaternion-getorientationquaternion)
* [`Timestamp getTimestamp()`](#timestamp-gettimestamp)

#### `Attitude getAttitude()`
Gets the attitude of the current device. Attitude contains the following attributes:
* `double roll`
* `double pitch`
* `double yaw`

These are self-explanatory.

#### `Location getLocation()`
Gets the location of the current device. The location contains the following attributes:
* `LocationStatus locationStatus`
	* `UNINITIALIZED` --> When the SDK is running in cartesian mode (XYZ frame only).
	* `NAVISENS_INITIALIZING` --> After `setLocationNavisens` has been called, the SDK is automatically initializing.
	* `NAVISENS_INITIALIZED` --> Whenever a user has input a location (through `setLocationLatitudeLongitude` or other) or 					    when the SDK has found a location automatically (after `setLocationNavisens` has been called).

* `XYZ localLocation`, units in meters.
* `GlobalLocation globalLocation`, units in degrees (latitude/longitude).
* `double heading`
* `double localHeading`
* `XY uncertainty`
* `VerticalMotionStatus verticalMotionStatus`
* `int floor`
* `double absoluteAltitude`
* `double absoluteAltitudeUncertainty`

The `locationStatus` can be used to determine whether or not our internal estimations have finished solving for your location.

The `localLocation` provides a location in meters on an imaginary cartesian world where the start location is at the origin.

The `globalLocation` on the other hand provides a latitude, longitude, and altitude.

The `uncertainty` is the uncertainty in the local location estimation.

The `heading` and `localHeading` are self-explanatory, and provide the global and local heading values respectively.

The `verticalMotionStatus` is the vertical motion status type.  The options include:
- `VERTICAL_STATUS_LEVEL_GROUND` when on flat/level ground or outdoors (will always report this outdoors)
- `VERTICAL_STATUS_ESCALATOR_UP` when riding escalator up
- `VERTICAL_STATUS_ESCALATOR_DOWN` when riding escalator down
- `VERTICAL_STATUS_ELEVATOR_UP` when riding elevator up
- `VERTICAL_STATUS_ELEVATOR_DOWN` when riding elevator down
- `VERTICAL_STATUS_STAIRS_UP` when walking up stairs
- `VERTICAL_STATUS_STAIRS_DOWN` when walking down stairs

The `floor` is the relative floor with respect to the initial location.  For example: if you set the location on floor 4, that will be the reference floor (i.e. floor 0).  Then, if you go up one floor, Navisens will report floor 1.  This function will always report 0 if there is no barometer.

The `absoluteAltitude` is the height of current position above mean sea level (or more precisely above the reference  WGS84 geoid)

The `absoluteAltitudeUncertainty` is the altitude uncertainty, which takes into consideration the average deviations from different phone models barometric measurements, the availability of recent weather station measurements, and the usefulness of the data provided given the distance between the weather station and the user.

##### Tutorial: Setting Floor Number
> 
> There are three functions for configuring the user floor number. Note that these functions are not listed under the **Control** section.
> 
> - **`void addFloorNumberAndHeight(int floor, double height)`**
>   - Appends a floor number with its respective height in meters to a vector that we use internally and correlate it to the current internal altitude measurements and output the proper floor based on the floors and heights you have inputted in our core.
>   - Then the `.getLocation().floor` outputted will be calculated using the floor numbers and heights you have inputted into our core.
>   - Note: Please enter the floor heights linearly starting from floor 0, then 1, 2, etc.
> - **`void setAverageFloorHeight(double floorHeight)`**
>   - Sets the average floor height we use to calculate the floor transitions internally, our default value is 4.5. When an averageFloorHeight is inputted this will override our 4.5m default value and use the value you have entered to measure the floor fluctuations. If all floors have the same height you can simply set the average floor height. 
> - **`void setFloorNumber(int floor)`**
>   - This method allows you to enter the current floor you are on, which overrides our internal estimate. We start measuring floor fluctuations from the floor you have inputted using the barometric data we receive from the device.
> 
> Known limitations:
> 
> If the floor height differs significantly from our default values the floor fluctuations will start reporting inaccuracies due to our default value, if you tweak this default value based on a known floor height of the building you are testing in you will be receiving better results.
> 
> Behavior:
> 
> If you initialize with setLocationNavisens, Navisens will manage global initialization of your start position and will also override the floor number and set to zero when we detect that the user is outdoors at street level. 

#### `Motion getMotion()`
Gets some more motion statistics, which provide the step frequency and the motion type. The motion type can be `STATIONARY`, `FIDGETING`, or `FORWARD`.

#### `String getID()`
Gets the unique id of the device.

#### `String getDeviceName()`
Gets the name of the device.

#### `MotionStatistics getMotionStatistics()`
Gets some motion statistics. These report the aggregate percentage of time spent in each motion type like the `getMotion()` method provides.

#### `OrientationQuaternion getOrientationQuaternion()`
Gets the look direction of the device. This can be useful for rendering applications. If you are dealing with augmented reality, make sure to call `setARModeEnabled(true)` to enable AR mode so the quaternion is broadcast in realtime.

#### `Timestamp getTimestamp()`
This timestamp represents the time at which the packet arrived, the time frame is with respect to when the phone last booted up.
Based on your configuration (e.g. PowerConsumptionMode.PERFORMANCE/MEDIUM_CONSUMPTION/LOW_CONSUMPTION), 
you will receive MotionDna events at different intervals:
- PERFORMANCE: 0.04s intervals
- MEDIUM_CONSUMPTION: 0.08s intervals
- LOW_CONSUMPTION: 0.16s intervals

