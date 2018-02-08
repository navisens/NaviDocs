# Creating New NavisensPlugins for Android

<sup>(Don't speak [`iOS`](/Tutorials/creating-navisensplugins.iOS.md)? Try [`Android`](/Tutorials/creating-navisensplugins.Android.md) instead!)</sup>

To create new NavisensPlugins for iOS, we recommend you use the Xcode IDE. The following tutorials will be done using Xcode.

Here is a brief overview of how you can create your own NavisensPlugins with custom behavior and/or even to share them online. If you are already familiar with some of these steps, feel free to skip forwards. If you would like to learn how to publish your plugin after you are done, you can use cocoapods [here](https://guides.cocoapods.org/making/index.html)!

1. [Creating a new project](#creating-a-new-project)

First, we need to create a new basic Cocoa Framework.

2. [Setting up podfile and workspace](#setting-up-podfile-and-workspace)

Here, we set up the podfile and convert our project into a workspace.

3. [Implementing NavisensPlugin](#implementing-navisensplugin)

NavisensPlugin has several methods that must be implemented in order for our NavisensCore to interface with it correctly.

4. [Using NavisensCore](#using-navisenscore)

The NavisensCore class is the central communication node between your plugin and any process which wishes to use the Navisens SDK (along with your new plugin).

5. [Using Navisens data and interplugin communication](#using-navisens-data-and-interplugin-communication)

Once we have everything set up, we can begin receiving data to/from the core or even other plugins!

## Creating a New Project

We need to create a blank project for now. In the next step, we'll set up the Podfile and configure the build environment.

1. Open Xcode, and go to **File**, and click **New**, and finally click **Project**
2. Choose **iOS** at the top, and select any Application you want. Alternatively, if you plan on publishing the plugin as a standalone library in the future, use **Cocoa Touch Framework** in the Frameworks section instead.
3. Give your product a name, and select a language. We will be using Swift 4 (it just says swift in the languages)

Now we can configure our Podfile, and set up the rest of our work environment.

## Setting up Podfile and Workspace

We will now set up our podfile, and configure the build settings of our workspace. First, close the project if you have it open.

1. Go in the same folder as the project, and create a new file. Name it `Podfile`.
2. Inside the `Podfile`, we want to add all of our dependencies. Copy the following into your `Podfile`:

```ruby
source 'https://github.com/CocoaPods/Specs.git'
platform :ios, '9.1'

target ‘ProjectName’ do
  use_frameworks!
  pod 'MotionDnaSDK', :git => 'https://github.com/navisens/iOS-SDK.git'
  pod ‘NavisensPlugins/NavisensCore’, :git => 'https://github.com/navisens/iOS-Plugin.git', :branch => 'repositories'
end

post_install do |installer|
    installer.pods_project.targets.each do |target|
        target.build_configurations.each do |configuration|
            target.build_settings(configuration.name)['ARCHS'] = 'armv7 arm64 i386 x86_64'
        end
    end
end
```

If you want to use other libraries, make sure to add them in too. Change the `ProjectName` to match the name of the project you made in step 1.

3. Now, we can open a command terminal, and navigate to your project directory, where there will be a podfile.
4. Make sure you have the `pod` command installed. Now run `pod install` from within the directory.
5. After it is done downloading everything, you can open up the **workspace** now. You could open it from command line using `open ProjectName.xcworkspace`.
6. Once the project is opened, go to the Project Settings by clicking on your project in the navigator on the left. Go to the **Build Settings**, and search for `bitcode` in the search. Once you find the option **Enable Bitcode**, disable it.
7. Next, *if you are going to publish a standalond library*, search for **Build Active Architecture Only** in both your project and the pod project. Set it to **No** for both **Debug** and **Release**.

Now we can start writing code.

## Implementing `NavisensPlugin`

Now that we are done setting up, we will briefly discuss how `NavisensCore` works, and how the `NavisensPlugin` fits in with all of this.

The `NavisensCore` is in charge of starting up our Navisens SDK using the provided SDK key. After it has instantiated our SDK, it begins awaiting plugins. Anytime a user wants to use a plugin, the user must invoke the function [`add(NavisensPlugin)`](https://github.com/navisens/iOS-Plugin/blob/master/navisenscore#func-addt-navisensplugin-_-navisensplugin-ttype-withparams-params-any---t). Afterwards, all control of the Navisens SDK is handed off to the plugins. This is very important! It is entirely up to the plugins system to invoke any Navisens SDK functions. While the `NavisensCore` will run the SDK, a user would need a plugin in order to set the user's location, etc.

When the `NavisensCore` is requested to instantiate a new `NavisensPlugin`, it will first call the constructor of the requested plugin. Immediately afterwards, it calls the plugin's own `initialize` method, passing in any of the user's parameters or arguments.

To shut down the system, `NavisensCore` will request all plugins to stop via the `stop` method, before killing the Navisens SDK.

Below, we will present the `NavisensPlugin` interface. The `initialize` and `stop` methods are described below, while the remaining methods are described in the [using Navisens data and interplugin communication](#using-navisens-data-and-interplugin-communication) section.

```swift
public interface NavisensPlugin {
  open func initialize(usingCore core: NavisensCore, andArgs args: [Any]) throws -> Bool
  open func stop() throws -> Bool
  open func receiveMotionDna(_ motionDna: MotionDna!) throws
  open func receiveNetworkData(_ motionDna: MotionDna!) throws
  open func receiveNetworkData(_ networkCode: NetworkCode, withPayload map: Dictionary<AnyHashable, Any>) throws
  open func receivePluginData(_ tag: String, data: Any) throws
  open func reportError(_ errorCode: ErrorCode, withMessage s: String) throws
}
```

The lifecycle of a plugin is very simple.
1. (External) `NavisensCore.add` is called, requesting the plugin
2. The plugin constructor is called
3. The plugin's `inititialize` method is called
4. (External) A request to stop the plugin is made 
5. The plugin's `stop` method is called
6. All of the resources that were requested by the plugin are discontinued and released
7. The plugin reference is dropped from the core

Note that all functions will throw an error if they are called, and unimplemented. It is not necessary to implement all functions if you can guarantee they will not be used. `initialize` and `stop` are guaranteed to be called by the core, while the remaining five receivers are guaranteed *not* to be called unless the plugin requests the core for the relevant resources.

#### `func initialize(usingCore core: NavisensCore, andArgs args: [Any]) throws -> Bool`

The `initialize` method is guaranteed to be called shortly after the constructor is called. A reference to the `NavisensCore` is passed in. You should save a reference to this object for later use. The `args` parameter contains any arguments that were passed in by the user. These may be used to further customize the behavior of a plugin.

See the [using NavisensCore](#using-navisenscore) below for details on what methods can be called from the `NavisensCore`.

At the end, you must return `true` if everything was initialized correctly. If you return `false`, the core will call the `stop` function shortly afterwards, terminating the plugin lifecycle.

Here are some methods of `NavisensCore` that you should call in the `init` method:

* [`core.subscribe`](#func-subscribe_-plugin-navisensplugin-to-which-int) can be used to request resources, or listen for certain types of events. These events include data like the core motion statistics for user location, network data for messages from other devices, or even errors from the SDK.

**Examples**

If you want to subscribe to all `MotionDna` from both the current device stream, all online devices, and any errors, use this:

```swift
  core.subscribe(self, to: NavisensCore.MOTION_DNA | NavisensCore.NETWORK_DNA | NavisensCore.ERRORS);
```

If you do not want to subscribe to anything, you can ignore the method call. You can also do this:

```swift
  core.subscribe(self, to: NavisensCore.NOTHING);
```

* `core.settings` is an object used to request the core apply certain settings. The settings object can also be used to force a setting.

**Examples**

If you want to increase the refresh rate from 100 ms to 50 ms, you can do this:

```swift
  core.settings.requestCallbackRate(50);
```

However, if another plugin already called `core.settings.requestCallbackRate(10)`, then the above call will not do anything, since 10 ms is at least better than 50 ms.

You can optionally force a state. Note that if you do so, however, the order of execution **does** matter, and should be noted in your documentation.

```java
  core.settings.overrideCallbackRate(50);
```

This call will force the state to 50 ms, even if another plugin had originally requested 10 ms.

* `core.motionDna` can be used to access even lower-level settings at the base SDK. Check out the API for more information on what you can do.

* [`core.applySettings()`](#func-applysettings) is used to commit all settings requested and apply them to the SDK.

#### `func stop() throws -> Bool`

This method will terminate a plugin. You must call `core.remove(self)` in this method so the `NavisensCore` will drop the reference to this plugin, to prevent memory leaks (unless your plugin can't be stopped at the current time).

If you return `true` from this method, you must release all resources, as `NavisensCore` will drop all references of this plugin, so you want to make sure not to have any memory leaks. If you return `false`, then you are telling the user that they cannot end this plugin at the current time, and that they should take action to resolve why.

## Using `NavisensCore`

The `NavisensCore` is the central interlocutor between all plugins.

Here is a brief API of features that `NavisensPlugin`s may use. These are **not** listed alongside the API presented [here](https://github.com/navisens/iOS-Plugin/tree/master/navisenscore#api) as they are designed to be used by plugins exclusively, although you may call them as normal if necessary.

#### `func subscribe(_ plugin: NavisensPlugin, to which: Int)`

This method subscribes a plugin to different resource streams. There are currently five resource streams as listed below:

* `MOTION_DNA` is the base stream of `MotionDna` for all location data of the current device
* `NETWORK_DNA` is a network stream of `MotionDna` of other devices that are on the same network (see the [NaviShare](https://github.com/navisens/iOS-Plugin/tree/master/navishare) plugin for more info).
* `NETWORK_DATA` is a network stream of strings from other devices that are on the same network (see the [NaviShare](https://github.com/navisens/iOS-Plugin/tree/master/navishare) plugin for more info).
* `PLUGIN_DATA` is an internal stream of raw tagged objects used to communicate between plugins.
* `ERRORS` is an error stream and is recommended. See [reportError](https://github.com/navisens/NaviDocs/blob/master/API.iOS.md#reporterror_-errorcode-errorcode-withmessage-s-string) for more information.

All streams are processed on their own thread, but all together. If one of your methods is expected to take a non-trivial amount of time to execute, it is recommended that you start a new thread to offload the work, so as to not stall the remaining plugins who are also awaiting resources from the corresponding streams. Also, the stream data is not copied to each plugin, so changing the resource data object may have unintended consequences.

The stream values can be mathematically or'd together to combine multiple streams (see example below). There are two additional symbolic streams:

* `NOTHING` is no streams
* `ALL` is all streams

**Examples**

If you want only `MOTION_DNA`, use this:

```swift
  core.subscribe(self, to: NavisensCore.MOTION_DNA);
```

If you want both `MOTION_DNA` and `ERRORS`, use this:

```swift
  core.subscribe(self, to: NavisensCore.MOTION_DNA | NavisensCore.ERRORS);
```

If you want everything except the `PLUGIN_DATA` stream, you can do this too:

```swift
  core.subscribe(self, to: NavisensCore.ALL ^ NavisensCore.PLUGIN_DATA);
```

#### `func unsubscribe(_ plugin: NavisensPlugin, from which: Int)`

This behaves the same as the above `subscribe` method, except that it stops providing the resources to the plugin.

#### `func broadcast(_ tag: String, data: Any)`

This method is used to communicate with other plugins. Any plugin that is currently subscribed to the resource `PLUGIN_DATA` will receive both the identifier, along with the Object array. The identifier is purely used by plugins to determine if the data being presented is something they recognize and can process. Since there is only one `PLUGIN_DATA` stream, the `tag` is designed as a secondary filter for this stream in the case that you have multiple mutually independent sets of `PLUGIN_DATA` resource-subscribing plugins.

#### `var motionDna: MotionDnaService?`

This method retrieves the base Navisens SDK object, which can be used to make low-level calls at the SDK level.

#### `var settings: NavisensSettings`

This method retrieves a shared settings object that all plugins can use to request various settings. The settings object will determine whether or not a requested setting will be applied. After settings are selected, make sure to call [`applySettings`](#func-applysettings) to commit changes. There are two types of methods to be called on the returned settings object: `request` methods and `override` methods.

When using the `request` methods, the settings object guarantees that the requested settings will be applied *only if* they are of equal or greater precedence than the currently applied setting. For example, when requesting a callback rate, if the current callback rate is 100 ms, and a request of 50 ms is issued, then the callback rate will be updated to 50 ms. However, if the current callback rate is 10 ms, and a request of 50 ms is issued, then the request will not change anything, since 10 ms is at least as good as 50 ms.

The behavior is different if you use the `override` methods, which will set the value regardless.

Finally, certain settings don't make sense to have a precedence, and thus will always behave like `override` methods, even when using the `request` version. An example is setting the host ip when using a shared server.

Below is listed the methods currently accepted. More low-level settings can be accessed through [`motionDna`](#var-motiondna-motiondnaservice). Note that a few other settings are set internally, but they are mainly for ease-of-use, and should not affect your plugin unless you explicitly require such settings.

```swift
public static class NavisensSettings {
    public func requestARMode()
    public func overrideARMode(_ mode: Bool?)
    
    public func requestCallbackRate(_ rate: Int)
    public func overrideCallbackRate(_ rate: Int?)
    
    public func requestGlobalMode()
    public func overrideEstimationMode(_ mode: EstimationMode?)
    
    public func requestPositioningMode(_ mode: ExternalPositioningState)
    public func overridePositioningMode(_ mode: ExternalPositioningState?)
    
    public func requestNetworkRate(_ rate: Int)
    public func overrideNetworkRate(_ rate: Int?)
    
    public func requestPowerMode(_ mode: PowerConsumptionMode)
    public func overridePowerMode(_ mode: PowerConsumptionMode?)
    
    public func requestHost(_ host: String, andPort port: String)
    public func overrideHost(_ host: String?, andPort port: String?)
    
    public func requestRoom(_ room: String)
    public func overrideRoom(_ room: String?)
}
```

As always, since all of the source code for the plugins system is open-source, if you are unsure of anything, go ahead and look at the source files.

#### `func applySettings()`

This method simply commits any settings you requested (or overrode) permanently. These settings are pushed to the main SDK, and applied immediately. If you do not call this method, none of the settings you requested earlier will take effect.

#### `func startServices()`

This method should be called if your plugin needs to begin running any background services. Only two services are started at the current time. The first is a global localization algorithm, which will only run if you request that the estimation mode be set to global using `NavisensSettings.requestGlobalMode`. The second connects to a server, if you are sharing your location with other devices, and only runs if you have set a non-null host and port.

## Using Navisens Data and Interplugin Communication

Once your plugin is running, you will begin receiving any requested resources from subscribed streams. The following summarizes the resources you get.

#### `func receiveMotionDna(_ motionDna: MotionDna!) throws`

This provides the `MotionDna` resource from the current device. Check out [here](https://github.com/navisens/NaviDocs/blob/master/API.iOS.md#getters) for information on the most commonly used components of the `MotionDna` object. For more information about the callback, see [here](https://github.com/navisens/NaviDocs/blob/master/API.iOS.md#receive_-motiondna-motiondna).

#### `func receiveNetworkData(_ motionDna: MotionDna!) throws`

This provides the `MotionDna` resource from all other devices in the current network room, if connected to a network room. Check out [here](https://github.com/navisens/NaviDocs/blob/master/API.iOS.md#getters) for information on the most commonly used components of the `MotionDna` object. For more information about the callback, see [here](https://github.com/navisens/NaviDocs/blob/master/API.iOS.md#receivenetworkdata_-motiondna-motiondna).

#### `func receiveNetworkData(_ networkCode: NetworkCode, withPayload map: Dictionary<AnyHashable, Any>) throws`

This provides information when working with NetworkData, including queries for devices, raw network data, or any network-related errors. For more information, see [here](https://github.com/navisens/NaviDocs/blob/master/API.iOS.md#receivenetworkdata_-networkcode-networkcode-withpayload-map-dictionary).

#### `func receivePluginData(_ tag: String, data: Any) throws`

This is a unique callback that deals with communications between plugins. All plugins using this stream will receive data broadcast from any other plugin. The identifier can be used to identify what type of data is being sent, and from what plugin. It is recommended that to make your identifier unique, you prefix all of your identifiers with the identifier of your project.

#### `func reportError(_ errorCode: ErrorCode, withMessage s: String) throws`

This reports any errors that may occur at the SDK level. For more information about the callback, see [here](https://github.com/navisens/NaviDocs/blob/master/API.iOS.md#reporterror_-errorcode-errorcode-withmessage-s-string).

