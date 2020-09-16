  - [iOS MotionDnaSDK API
    Documentation](#ios-motiondnasdk-api-documentation)
      - [Control](#control)
          - [func start(withDeveloperKey:
            String)](#func-startwithdeveloperkey-string)
          - [func start(withDeveloperKey: String, andConfigurations:
            \[String :
            Any\])](#func-startwithdeveloperkey-string-andconfigurations-string-any)
          - [func stop()](#func-stop)
          - [func reset()](#func-reset)
          - [func pause()](#func-pause)
          - [func resume()](#func-resume)
      - [Configuration](#configuration)
      - [Global Estimation
        Modification](#global-estimation-modification)
          - [func setLocationLatitude(latitude: Double, longitude:
            Double)](#func-setlocationlatitudelatitude-double-longitude-double)
          - [func setLocationLatitude(latitude: Double, longitude:
            Double, andHeadingInDegrees: Double)
            ](#func-setlocationlatitudelatitude-double-longitude-double-andheadingindegrees-double)
          - [func setHeadingInDegrees(heading:
            Double)](#func-setheadingindegreesheading-double)
      - [Cartesian Estimation
        Modification](#cartesian-estimation-modification)
          - [func setCartesianHeading(heading:
            Double)](#func-setcartesianheadingheading-double)
          - [func setCartesianPositionX( x: Double, y:
            Double)](#func-setcartesianpositionx-x-double-y-double)
      - [SDK Info](#sdk-info)
          - [static func sdkBuild() -\>
            String](#static-func-sdkbuild---string)
          - [static func sdkVersion() -\>
            String](#static-func-sdkversion---string)
      - [Callbacks (MotionDnaSDKDelegate
        implementation)](#callbacks-motiondnasdkdelegate-implementation)
          - [func receive(\_ motionDna:
            MotionDna)](#func-receive_-motiondna-motiondna)
          - [func report(\_ status: MotionDnaSDKStatus, withMessage
            message:
            String?)](#func-report_-status-motiondnasdkstatus-withmessage-message-string)
      - [Estimation Properties](#estimation-properties)

# iOS MotionDnaSDK API Documentation

## Control

### func start(withDeveloperKey: String)

### func start(withDeveloperKey: String, andConfigurations: \[String : Any\])

### func stop()

### func reset()

### func pause()

### func resume()

## Configuration

|                       |                                            |             |                                                                                                    |
| --------------------- | ------------------------------------------ | ----------- | -------------------------------------------------------------------------------------------------- |
| **Keyword**           | **Options**                                | **Default** | **Description**                                                                                    |
| model                 | String (“simple”, “standard”, “headmount”) | “standard”  | Choose the type of model to use for motion estimation                                              |
| gps                   | Bool                                       | true        | Use gps in estimation                                                                              |
| corrected\_trajectory | Bool                                       | false       | Show path leading up to global position fix and a history the updated path then corrections happen |
| callback              | Double                                     | 40ms        | Rate (ms) at which estimation is delivered to app layer                                            |
| logging               | Bool                                       | false       | Record log file for debugging with Navisens team                                                   |

## Global Estimation Modification

### func setLocationLatitude(latitude: Double, longitude: Double)

### func setLocationLatitude(latitude: Double, longitude: Double, andHeadingInDegrees: Double) 

### func setHeadingInDegrees(heading: Double)

## Cartesian Estimation Modification

### func setCartesianHeading(heading: Double)

### func setCartesianPositionX( x: Double, y: Double)

## SDK Info

### static func sdkBuild() -\> String

### static func sdkVersion() -\> String

## Callbacks (MotionDnaSDKDelegate implementation)

### func receive(\_ motionDna: MotionDna)

### func report(\_ status: MotionDnaSDKStatus, withMessage message: String?)

## Estimation Properties

These are the values representing the estimation provided

|                   |                  |                                  |
| ----------------- | ---------------- | -------------------------------- |
| Class             | Variable/Getter  | Type                             |
| MotionDna         | attitude         | Attitude                         |
|                   | location         | Location                         |
|                   | MotionType       | enum MotionType                  |
|                   | classifiers      | Dictionary \[String:Classifier\] |
|                   | motionStatistics | MotionStatstics                  |
|                   | timestamp        | double                           |
|                   |                  |                                  |
| Attitude          | euler            | Euler                            |
|                   | quaternion       | Quaternion                       |
|                   |                  |                                  |
| Euler             | roll             | double                           |
|                   | pitch            | double                           |
|                   | yaw              | double                           |
|                   |                  |                                  |
| Quaternion        | x                | double                           |
|                   | y                | double                           |
|                   | z                | double                           |
|                   | w                | double                           |
|                   |                  |                                  |
| Location          | cartesian        | CartesianLocation                |
|                   | global           | GlobalLocation                   |
|                   |                  |                                  |
| CartesianLocation | x                | double                           |
|                   | y                | double                           |
|                   | z                | double                           |
|                   | heading          | double                           |
|                   |                  |                                  |
| GlobalLocation    | latitude         | double                           |
|                   | longitude        | double                           |
|                   | altitude         | double                           |
|                   | accuracy         | enum GlobalLocationAccuracy      |
|                   |                  |                                  |
| MotionStatistics  | fidgeting        | double                           |
|                   | walking          | double                           |
|                   | stationary       | double                           |
