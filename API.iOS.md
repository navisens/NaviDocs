# NaviDocs/iOS #
-----

This is the central iOS API documentation for the Navisens MotionDnaSDK.

## Installation ##

If you haven't already done so or haven't build an iOS app before, check out the [Quick Start Guide](/BEER.iOS.md), as it is designed to be a very straight-forward demo of an easy app you can build with our SDK and tools. Otherwise, if you already have some experience building iOS apps, here's what you need to do to set up the Navisens MotionDnaSDK.

Creating a `Podfile` with the following pods, and install them:

```pod
source 'https://github.com/CocoaPods/Specs.git'
platform :ios, '9.1'

target 'ProjectName' do
  use_frameworks!
  pod 'MotionDnaSDK'
end
```

# API #
-----
To setup the MotionDnaSDK, you must implement the `MotionDnaSDK` protocol. This serves as both a controller for the SDK, and an event listener to receive updates on estimation results from our SDK.

Below, we will organize the API into several sections. First, the **Setup** and **Control** functions are called through `MotionDnaSDK`. **Setup** involves tasks that must be completed for our SDK to function. **Control** includes any functions that alter the behavior of our SDK, along with retrieving some basic device information. We also have the **Callbacks**, which are also implemented through the `MotionDnaSDK` protocol. **Callbacks** must all be implemented, but don't necessarily need to do anything. Some of the callbacks are invoked with an event object of type `MotionDna`. In the **Getters** section, we'll explain what information you can extract from these event objects.
 * [Setup](#setup)
 * [Control](#control)
 * [Callbacks](#callbacks)
 * [Getters](#getters)

-----
## Setup ##

These are functions that must be called in order to configure the SDK completely.

* [`runMotionDna(_ devkey: String, receiver: MotionDnaSDK)`](#runmotiondna_-devkey-string-receiver-motiondnasdk)
* [`runMotionDnaWithoutMotionManager(ID: String)`](#runmotiondnawithoutmotionmanagerid-string)

-----

#### `runMotionDna(_ devkey: String, receiver: MotionDnaSDK)`
This functions starts up the SDK. You must pass in a valid developer's key in order for the SDK to function. If the key has expired or there are other errors, you may receive those errors through the [`reportError()`](#reporterror_-errorcode-errorcode-withmessage-string) callback route.

**Params**
The first paramater is your developer key. If this is missing, or incorrect, the SDK will cease to function. The second parameter is the callback route. You may provide `self` as a valid object.


#### `runMotionDnaWithoutMotionManager(ID: String)`
This function behaves like the `runMotionDna()` method above and similarly requires a developer key. However it does not start up the phone sensors allowing sensor input from other sources.

#### `inputMotion(withTimestamp: Double, roll: Double, pitch: Double, yaw: Double, accX: Double, accY: Double, accZ: Double)` 
This method must be calls at regular intervals with sensor data if you have started the SDK with `void runManualMotionDna()`.

**Params**
Timestamp units must be in seconds. Accelerometer x, y, and z should be in G's and Gyro roll, pitch, and yaw unit should be in radians/sec. 

-----
## Control ##

There are many ways to control the SDK. Here, we have split control methods into several sections.

* Device Information
	* [`MotionDnaSDK.checkSDKVersion() -> String`](#motiondnasdkchecksdkversion---string)
	* [`getDeviceID() -> String`](#getdeviceid---string)
	* [`getDeviceName() -> String`](#getdevicemodel---string)
* SDK Control
	* [`resume()`](#resume)
	* [`pause()`](#pause)
	* [`stop()`](#stop)
	* [`resetLocalEstimation()`](#resetlocalestimation)
	* [`resetLocalHeading()`](#resetlocalheading)
* SDK Settings
	* [`setARModeEnabled(_ state: Bool)`](#setarmodeenabled_-state-bool)
	* [`setBackgroundModeEnabled(_ state: Bool)`](#setbackgroundmodeenabled_-state-bool)
	* [`enableBackgroundSensors()`](#enablebackgroundsensors)
	* [`setBackpropagationEnabled(_ state: Bool)`](#setbackpropagationenabled_-state-bool)
	* [`setBackpropagationBufferSize(_ size: Int)`](#setbackpropagationbuffersize_-size-int)
	* [`setCorrectionBackpropagationEnabled(_ state: Bool)`](#setcorrectionbackpropagationenabled_-state-bool)
	* [`setBinaryFileLoggingEnabled(_ state: Bool)`](#setbinaryfileloggingenabled_-state-bool)
	* [`setCallbackUpdateRateInMs(_ rate: Double)`](#setcallbackupdaterateinms_-rate-double)
	* [`setNetworkUpdateRateInMs(_ rate: Double)`](#setnetworkupdaterateinms_-rate-double)
	* [`setPowerMode(_ mode: PowerConsumptionMode)`](#setpowermode_-mode-powerconsumptionmode)
	* [`setExternalPositioningState(_ mode: ExternalPositioningState)`](#setexternalpositioningstate_-mode-externalpositioningstate)
* Global Location Initialization
	* [`setLocationNavisens()`](#setlocationnavisens)
	* [`setLocationGPSOnly()`](#setlocationgpsonly)
	* [`setLocationLatitude(_ latitude: Double, longitude: Double)`](#setlocationlatitude_-latitude-double-longitude-double)
	* [`setLocationLatitude(_ latitude: Double, longitude: Double, andHeadingInDegrees heading: Double)`](#setlocationlatitude_-latitude-double=longitude-double-andheadingindegrees-heading-double)
	* [`inputGlobalCoordinatesLat(_ latitude: Double, longitude: Double, AndTimestamp timestamp: Double)`](#inputglobalcoordinateslat_-latitude-double-longitude-double-andtimestamp-timestamp-double)
	* [`inputGlobalCoordinatesLat(_ latitude: Double, longitude: Double, timestamp: Double, AndAngleThreshold angleThreshold: Double)`](#inputglobalcoordinates_-latitude-double-longitude-double-timestamp-double-andanglethreshold-anglethreshold-double)
	* [`setHeadingMagInDegrees()`](#setheadingmagindegrees)
	* [`setHeadingInDegrees(_ heading: Double)`](#setheadingindegrees_-heading-double)
	* [`setCartesianOffsetInMetersX(_ x: Double, Y y: Double)`](#setcartesianoffsetinmetersx_-x-double-y-y-double)
	* [`setLocalHeadingOffsetInDegrees(_ heading: Double)`](#setlocalheadingoffsetindegrees_-heading-double)
	* [`setLocalHeading(_ heading: Double)`](#setlocalheading_-heading-double)
	* [`setCartesianPositionX(_ x: Double, Y y: Double)`](#setcartesianpositionx_-x-double-y-y-double)
* Location Sharing
	* [`startUDP(...)`](#startudp)
	* [`stopUDP()`](#stopudp)
	* [`setUDPRoom(_ room: String)`](#setudproom_-room-string)
	* [`sendUDPPacket(_ msg: String)`](#sendudppacket_-msg-string)
	* [`sendUDPQueryRooms(_ rooms: NSMutableArray)`](#sendudpqueryrooms_-rooms-nsmutablearray)

-----
#### `MotionDnaSDK.checkSDKVersion() -> String`
Get the SDK version used to build the application.

**Returns**
A string representing the current SDK version used in the app.

#### `getDeviceID() -> String`
Get a unique id for the current device.

**Returns**
A string id of the current device, which will always be the same.

#### `getDeviceName() -> String`
Get the model of the device represented as a string.

**Returns**
A string representing the device model.

-----
#### `resume()`
Resume estimation of current location, if it was paused in the past.

#### `pause()`
Pause estimation of the current location. Estimation can be resumed at a later time by calling `resume()`.

#### `stop()`
Completely shut down all estimation in our SDK, and release any resources that are in-use.

#### `resetLocalEstimation()`
Resets the cartesian coordinates back to the origin `(0, 0)`, and sets the heading to `0` degrees.

#### `resetLocalHeading()`
Resets just the heading to 0 degrees.

-----
#### `setARModeEnabled(_ state: Bool)`
Enables AR mode. AR mode publishes orientation quaternion at a higher rate.

**Params**
Enable or disable AR mode.

#### `setBackgroundModeEnabled(_ state: Bool)`
Allow our SDK to run while your app is backgrounded by turning the CLLocationManager on. If you wish to use this, you must enable the `Location updates` `Background Mode` and enable `Background Modes` if it is not on.

**Attention for any device iOS 11 and above, please follow the instructions for the enableBackgroundSensor method
to prevent sensors from stopping in the background.**

**Params**
Enable or disable background location updates.

#### [Deprecated] `enableBackgroundSensors()`

This function is no longer needed for MotionDna to run in background. From v1.8.0 it does nothing. It will be removed in next major release.

~~If you are using iOS 11+, then you should also go to your `AppDelegate` file and find the `applicationDidEnterBackground` method and add this method into it.
This method essentially restarts our internal CMMotionManager to ensure sensors updates keep occuring in background, this is an **Apple** bug that has already been submitted.~~

```swift
  MotionDnaSDK.enableBackgroundSensors()
```

#### `setBackpropagationEnabled(_ state: Bool)`

When setLocationNavisens is enabled and setBackpropagationEnabled is called, once Navisens has initialized you will not only get the current position, but also a set of latitude longitude coordinates which lead back to the start position (where the SDK/App was started). This is useful to determine which building and even where inside a building the person started, or where the person exited a vehicle (e.g. the vehicle parking  spot or the location of a drop-off).

The historical trajectory of points will be ordered as follows: Point(t-n), Point(t-2), Point(t-1), Point(t-0)
So you will receive all the points from the past with the appropriate timestamps, until you get to your current position.

```swift
  MotionDnaSDK.enableBackpropagation()
```

**Params**

Enable or disable location back propagation.

#### `setBackpropagationBufferSize(_ size: Int)`

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

#### `setCorrectionBackpropagationEnabled(_ state: Bool)`

When setLocationNavisens is enabled and setCorrectionBackpropagationEnabled is called, after Navisens has initialized you will get the current position, and with certain corrections you will receive a series of latitude longitude coordinates representing a revised estimation on a recent portion of the traversed path.

The corrected trajectory of points will be ordered by timestamp from oldest to most recent, until you get to your current revised position.

**Params**

Enable or disable correction back propagation.

#### `setBinaryFileLoggingEnabled(_ state: Bool)`
Allow our SDK to record data and use it to enhance our estimation system.

**Params**
Enable or disable file logging.

##### Tutorial: Downloading Logs
> Downloading these logs is a very good way of providing debug information, should there be an issue with our SDK. For iOS, first make sure you have Xcode set up. Open it up, and make sure your device is plugged in into your computer.
> 
> 1. Go to the top and click on the **Window** tab.
> 2. Open up the **Devices and Simulators** window from the menu.
> 3. Select the **Devices** tab at the top.
> 4. Under **Installed Apps**, select the relevant app.
> 5. Click on the *Gear Icon* to open up settings.
> 6. Click on **Download Container**. This may take a while.
> 7. After it is done, right click on the container and click **Show Package Contents**.

#### `setCallbackUpdateRateInMs(_ rate: Double)`
Tell our SDK how often to provide estimation results. Note that there is a limit on how fast our SDK can provide results, but usually setting a slower update rate improves results. Setting the rate to `0ms` will output estimation results at our maximum rate.

**Params**
The number of milliseconds `ms` between each update. Setting this to 5000 will publish estimation results every 5 seconds, while setting this to 0 will publish results at the fastest rate our SDK is capable of running estimations (though not consistently timed).

#### `setNetworkUpdateRateInMs(_ rate: Double)`
Tell our SDK how often to publish our location to your servers. This allows your device to receive MotionDna information on other devices on the same network.

**Params**
The number of milliseconds `ms` between each network update. Setting this to 5000 will send network estimation results every 5 seconds, while setting this to 0 will send results at the fastest rate our SDK is capable of running estimations (though not consistently timed).

#### `setPowerMode(_ mode: PowerConsumptionMode)`
Set the power consumption mode to trade off accuracy of predictions for power saving.

**Params**
The `PowerConsumptionMode` is one of the following:

* `LOW_CONSUMPTION`: Internal Estimation at 6.25Hz
* `MEDIUM_CONSUMPTION`: Internal Estimation at 12.5Hz
* `PERFORMANCE`: Internal Estimation at 25Hz


#### `setExternalPositioningState(_ mode: ExternalPositioningState)`
This method is essentially a wrapper for Apple's CLLocationManager, which allows us to control from the API & the core sides
Apple's GPS.
You are required to at least set the externational positioning state to LOW_ACCURACY if you want your application to function in the background.
And set it to HIGH_ACCURACY if you are using setLocationNavisens (need not worry thanks to our MotionDna
technology we decide when to switch GPS between LOW and HIGH accuracies internally to prevent battery drainage).


**Params**
The `ExternalPositioningState` is one of the following:

* `OFF`: GPS is off (application will not function in background)
* `LOW_ACCURACY`: GPS set to 1 kilometer accuracy (kCLLocationAccuracyKilometer) (application will function in background)
* `HIGH_ACCURACY`: GPS set to best accuracy (kCLLocationAccuracyBest) & no distance filter (kCLDistanceFilterNone) (application will function in background)

-----
#### `setLocationNavisens()`
Use our internal algorithm to automatically compute your location and heading by fusing inertial estimation with global location information. This is designed for outdoor use and will not compute a position when indoors. Solving location requires the user to be walking outdoors. Depending on the quality of the global location, this may only require as little as 10 meters of walking outdoors.

While we are computing the user's location, you can use the `locationStatus` of [`getLocation()`](#getlocation---location) to determine when an accurate location has been determined.

#### `setLocationGPSOnly()`
Wait for the next GPS reading, and set the latitude and longitude to that reading.

Given the limitations of GPS, it is best to use this only if you can guarantee the user has accurate GPS readings, and is outside, or you do not need a very accurate position.

#### `setLocationLatitude(_ latitude: Double, longitude: Double)`
Manually set only the global latitude and longitude.

Use this if you have other sources of information (for example, user-defined address), and need readings more accurate than GPS can provide.

**Params**
The latitude `lat` and longitude `long`.

#### `setLocationLatitude(_ latitude: Double, longitude: Double, andHeadingInDegrees heading: Double)`
Manually sets the global latitude, longitude, and heading. This enables receiving a latitude and longitude instead of cartesian coordinates.

Use this if you have other sources of information (for example, user-defined address), and need readings more accurate than GPS can provide.

**Params**
The latitude `lat` and longitude `long`, and heading `head` in degrees.

#### `setHeadingMagInDegrees()`
Wait for the next magnetic compass reading, and set the heading to that reading.

Given the limitations of the magnetic compass, don't use this unless you are sure the device supports readings at high accuracy.

#### `setHeadingInDegrees(_ heading: Double)`
Manually set the initial global heading

It is advised if you are using global heading, to manually set this, or allow the user to set this.

**Params**
The `heading`, as a rotation in degrees.

#### `inputGlobalCoordinatesLat(_ latitude: Double, longitude: Double, andTimestamp timestamp: Double)`
This method allows you to feed in geodetic coordinates from an external positioning system (e.g. beacons/wifi) and running our internal algorithms to find an initial start point. The method expects: latitude, longitude and timestamp (in seconds since boot time).

**Params**
The latitude and longitude coordinates with the associated timestamp.

#### `inputGlobalCoordinates(_ latitude: Double, longitude: Double, timestamp: Double, AndAngleThreshold angleThreshold: Double)`
Same as the above method except with an angular threshold input in degrees (the default is 7.5 degrees). The angleThreshold specifies the minimum required accuracy for Navisens to initialize the position and heading. The lower the value the more accurate the initialization and longer the initialization period. The higher the value the faster the initialization and the lower the accuracy as a tradeoff for speed.

**Params**
The latitude and longitude coordinates and timestamp with the desired angular threshold in degrees.

#### `setCartesianOffsetInMetersX(_ x: Double, Y y: Double)`
Adds an offset to both x and y axes. So if you are running the SDK in the local XY frame, you can start at custom XY positions.

**Params**
The `x` and `y` offset in meters.

#### `setLocalHeadingOffsetInDegrees(_ heading: Double)`
Sets the local heading offset in degrees. The heading offsets you input get accumulated. You can reset the offset by calling `resetLocalHeading`.

**Params**
The `heading`, as a counterclockwise rotation in degrees.

#### `setCartesianPositionX(_ x: Double, Y y: Double)`
Sets the Cartesian position in XY to the exact x and y passed in.

**Params**
The x and y coordinates to set.

#### `setLocalHeading(_ heading: Double)`
Sets the local heading the exact heading value passed in.

**Params**
The `heading`, as a counterclockwise rotation in degrees (in +- 180 frame).


##### Tutorial: Running the SDK in the local frame.
> To run our SDK without any global reference, its simple, you only need to call `runMotionDna`, without any global input call. Since the system doesn't receive any user input or activation of the Navisens automatic initialization input (e.g. `setLocationNavisens`), it assumes the estimation is being run in the local XY frame.
> When receiving user user motionDna, you can access the `localLocation` and `localHeading` attributes.

-----
#### `startUDP(...)`
Connect to your own server and specify a room. Any other device connected to the same room and also under the same developer will receive any udp packets this device sends.

This device will automatically begin broadcasting the `MotionDna` to your servers, so other devices will receive positional data. Using the [`sendUDPPacket(_ msg: String)`](#sendudppacket_-msg-string) call will allow sending arbitrary strings of data, although it is recommended to use only safe and secure characters if communicating with devices of differing OS.

All messages and data are encrypted and sent securely; the server is unable to decode anything - only clients can read and interpret messages.

There are multiple versions of this method as follows:
* `startUDP()`
* `startUDPRoom(_ room: String)`
* `startUDPRoom(_ room: String, atHost host: String, andPort port: String)`
* `startUDPHost(_ host: String, andPort port: String)`

For each version, if `host` and `port` is missing, a default public server will be used. Note that the public server has a limited amount of space, and may not always be available for use, so it is best to only use it for testing and instead use your own server for production.

If `room` is not provided, the default room will be used. Also note that the default room has a maximum number of device connections provisioned for it, so this room should only be used for testing and not any formal usage.

Recommendation:
Use `startUDPHost(_ host: String, andPort port: String)` for internal testing, and only use `startUDPRoom(_ room: String, atHost host: String, andPort port: String)` for deployment release build.

#### `stopUDP()`
Stop broadcasting positional data to your servers.

#### `setUDPRoom(_ room: String)`
Change the current 

#### `sendUDPPacket(_ msg: String)`
Send a string of data that will be broadcast to all other devices also in the same server room.

Data is encrypted and sent securely. It is recommended that data use only safe and secure characters to reduce chances of failure.

To receive the messages from other devices, see [`receiveNetworkData()`](#receivenetworkdata_-networkcode-networkcode-withpayload-map-dictionary)

#### `sendUDPQueryRooms(_ rooms: NSMutableArray)`
Query the server for the current capacity of the listed rooms. Note that the server was designed so that you cannot retrieve a list of rooms without modifying the server. This is to encourage privacy through obscurity.

To receive the results of a query, see [`receiveNetworkData()`](#receivenetworkdata_-networkcode-networkcode-withpayload-map-dictionary)

-----

## Callbacks ##

Callbacks are received from the implemented `MotionDnaInterface` class, and are each passed relevant event information in each listener method. While not all callbacks must be implemented, it is recommended that each case is handled as needed.

* [`receive(_ motionDna: MotionDna)`](#receive_-motiondna-motiondna)
* [`receiveNetworkData(_ motionDna: MotionDna)`](#receivenetworkdata_-motiondna-motiondna)
* [`receiveNetworkData(_ networkCode: NetworkCode, withPayload map: Dictionary<>)`](#receivenetworkdata_-networkcode-networkcode-withpayload-map-dictionary)
* [`reportError(_ errorCode: ErrorCode, withMessage s: String)`](#reporterror_-errorcode-errorcode-withmessage-s-string)

-----
#### `receive(_ motionDna: MotionDna)`
This event receives the estimation results using a `MotionDna` object. Check out the [Getters](#getters) section to learn how to read data out of this object.

#### `receiveNetworkData(_ motionDna: MotionDna)`
This event receives estimation results from other devices in the server room. In order to receive anything, make sure you call `startUDP` to connect to a room. Again, it provides access to a `MotionDna` object, which can be unpacked the same way as above.

If you aren't receiving anything, then the room may be full, or there may be an error in your connection. See the `reportError` event below for more information.

#### `receiveNetworkData(_ networkCode: NetworkCode, withPayload map: Dictionary<>)`
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

#### `reportError(_ errorCode: ErrorCode, withMessage s: String)`
Report any errors of the estimation or internal SDK. There are five possible errors as follows:
* `ERROR_SENSOR_TIMING`
* `ERROR_AUTHENTICATION_FAILED`
* `ERROR_SENSOR_MISSING`
* `ERROR_SDK_EXPIRED`
* `ERROR_PERMISSION`
* `WRONG_FLOOR_INPUT`

The errors are self-explanatory, and the error description can be printed to gain further information on what caused the error. The most important error that should be handled is the permissions error, as this is the only error that is really relevant to the user. The other errors may be handled by simply shutting down the SDK.

-----

## Getters ##

These are methods of the `MotionDna` object, and are used to get data from these event objects returned from the callbacks `receiveMotionDna()` and `receiveNetworkData()`.

* [`getAttitude() -> Attitude`](#getattitude---attitude)
* [`getLocation() -> Location`](#getlocation---location)
* [`getMotion() -> Motion`](#getmotion---motion)
* [`getID() -> String`](#getid---string)
* [`getDeviceName() -> String`](#getdevicename---string)
* [`getMotionStatistics() -> MotionStatistics`](#getmotionstatistics---motionstatistics)
* [`getQuaternion() -> OrientationQuaternion`](#getquaternion---orientationquaternion)
* [`getTimestamp() -> Timestamp`](#gettimestamp---timestamp)


#### `getAttitude() -> Attitude`
Gets the attitude of the current device. Attitude contains the following attributes:
* `roll: Double`
* `pitch: Double`
* `yaw: Double`

These are self-explanatory.

#### `getLocation() -> Location`
Gets the location of the current device. The location contains the following attributes:
* `locationStatus: LocationStatus`
	* `UNINITIALIZED` --> When the SDK is running in cartesian mode (XYZ frame only).
	* `NAVISENS_INITIALIZING` --> After `setLocationNavisens` has been called, the SDK is automatically initializing.
	* `NAVISENS_INITIALIZED` --> Whenever a user has input a location (through `setLocationLatitudeLongitude` or other) or 					    when the SDK has found a location automatically (after `setLocationNavisens` has been called).

* `localLocation: XYZ`, units in meters.
* `globalLocation: GlobalLocation`, units in degrees (latitude/longitude).
* `heading: Double`, with respect to True North in the 0-360 frame.
* `localHeading: Double`
* `uncertainty: XY`
* `verticalMotionStatus: VerticalMotionStatus`
* `floor: Int`
* `absoluteAltitude: Double`
* `absoluteAltitudeUncertainty: Double`

The `locationStatus` can be used to determine whether or not our internal estimations have finished solving for your location.

The `localLocation` provides a location in meters on an imaginary cartesian world where the start location is at the origin.

The `globalLocation` on the other hand provides a latitude, longitude, and altitude.

The `uncertainty` is the uncertainty in the local location estimation.

The `heading` and `localHeading` are self-explanatory, and provide the global and local heading values respectively.

The `verticalMotionStatus` is the vertical motion status type.  Due to the low sample rate and delay in barometer data on iOS devices, the motion status’ may be delayed by a few seconds. The options include:
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
> - **`addFloorNumber(_ floor: Int, AndHeight height: Double)`**
>   - Appends a floor number with its respective height in meters to a vector that we use internally and correlate it to the current internal altitude measurements and output the proper floor based on the floors and heights you have inputted in our core.
>   - Then the `.getLocation().floor` outputted will be calculated using the floor numbers and heights you have inputted into our core.
>   - Note: Please enter the floor heights linearly starting from floor 0, then 1, 2, etc.
> - **`setAverageFloorHeight(_ floorHeight: Double)`**
>   - Sets the average floor height we use to calculate the floor transitions internally, our default value is 4.5. When an averageFloorHeight is inputted this will override our 4.5m default value and use the value you have entered to measure the floor fluctuations. If all floors have the same height you can simply set the average floor height. 
> - **`setFloorNumber(_ floor: Int)`**
>   - This method allows you to enter the current floor you are on, which overrides our internal estimate. We start measuring floor fluctuations from the floor you have inputted using the barometric data we receive from the device.
> 
> Known limitations:
> 
> If the floor height differs significantly from our default values the floor fluctuations will start reporting inaccuracies due to our default value, if you tweak this default value based on a known floor height of the building you are testing in you will be receiving better results.
>
> Behavior:
> 
> If you initialize with setLocationNavisens, Navisens will manage global initialization of your start position and will also override the floor number and set to zero when we detect that the user is outdoors at street level. 

#### `getMotion() -> Motion`
Gets some more motion statistics, which provide the step frequency and the motion type. The motion type can be `STATIONARY`, `FIDGETING`, or `FORWARD`.

#### `getID() -> String`
Gets the unique id of the device.

#### `getDeviceName() -> String`
Gets the name of the device.

#### `getMotionStatistics() -> MotionStatistics`
Gets some motion statistics. These report the aggregate percentage of time spent in each motion type like the `getMotion()` method provides.

#### `getOrientationQuaternion() -> OrientationQuaternion`
Gets the look direction of the device. This can be useful for rendering applications. If you are dealing with augmented reality, make sure to call `setARModeEnabled(true)` to enable AR mode so the quaternion is broadcast in realtime.

#### `getTimestamp() -> Timestamp`
This timestamp represents the time at which the packet arrived, the time frame is with respect to when the phone last booted up.
Based on your configuration (e.g. PowerConsumptionMode.PERFORMANCE/MEDIUM_CONSUMPTION/LOW_CONSUMPTION), 
you will receive MotionDna events at different intervals:
- PERFORMANCE: 0.04s intervals
- MEDIUM_CONSUMPTION: 0.08s intervals
- LOW_CONSUMPTION: 0.16s intervals
