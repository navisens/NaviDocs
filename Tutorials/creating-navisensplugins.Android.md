# Creating New NavisensPlugins for Android

<sup>(Don't speak [`Android`](/Tutorials/creating-navisensplugins.Android.md)? Try [`iOS`](/Tutorials/creating-navisensplugins.iOS.md) instead!)</sup>

To create new NavisensPlugins for Android, we recommend you use the Android Studios IDE. The following tutorials will be done using Android Studios, but other IDEs may work as well.

Here is a brief overview of how you can create your own NavisensPlugins with custom behavior and/or even to share them online. If you are already familiar with some of these steps, feel free to skip forwards. If you would like to learn how to publish your plugin after you are done, see [here](/Tutorials/publishing-navisensplugins.Android.md)!

1. [Creating a new project](#creating-a-new-project)

First, we need to create a new basic Android Project for exporting libraries.

2. [Setting up dependencies](#setting-up-dependencies)

Here, we set up the relevant dependencies needed to get everything up and running.

3. [Implementing NavisensPlugin](#implementing-navisensplugin)

`NavisensPlugin` has several methods that must be implemented in order for our `NavisensCore` to interface with it correctly.

4. [Using NavisensCore](#using-navisenscore)

The `NavisensCore` class is the central communication node between your plugin and any process which wishes to use the Navisens SDK (along with your new plugin).

5. [Using Navisens data and interplugin communication](#using-navisens-data-and-interplugin-communication)

Once we have everything set up, we can begin receiving data to/from the core or even other plugins!

## Creating a New Project

We just need to create a new basic Android project, with no fancy attachments. 

1. Once you open up Android Studio, the first step is to create a new project.
2. We want to set our target devices to *Phone and Tablet*.
3. Finally, you may select the type of activity you want. There are two options here that are very important:
  * If you are creating a plugin that you want to use in *other* projects, you will need to publish it as a separate library. You do not need an activity in this case. However, you may optionally elect to create a test activity to make sure your plugin works. In either case, you should check out our short tutorial on how to [publish a plugin](/Tutorials/publishing-navisensplugins.Android.md) for more information on assessing and packing the correct source files to be distributed.
  * Alternatively, if you just want to set up a plugin that you will use only for your current project, then you should set up your project exactly as if you were just doing the project. In this case, you would just implement our `NavisensPlugin` as any normal interface.

It is a good idea to determine the use cases of your project now so you can minimize the amount of time moving files around. Even if you go with the latter option, you can always publish the plugin later on, though it will take a bit of extra configuration. In either case, the following sections will remain the same anyways.

Now we can go ahead and create our new empty project. Next, we'll set up our gradle buildscripts.

## Setting up Dependencies

In our gradle buildscripts, we need to include all of the Navisens dependencies, along with any other dependencies you wish to use.

Just like any other project that makes use of our `NavisensPlugin`s system, you must include the dependencies for the Navisens SDK, and `NavisensCore`.

1. Open up your **project's gradle buildscript**. If you aren't sure how to do this, it is recommended you check out the [Beer Test Tutorial](/BEER.Android.md) for a picture-by-picture tutorial on creating a project and setting up your dependencies.
2. Add the following into your **buildscript repositories**:

```gradle
maven {
  url 'https://oss.sonatype.org/content/groups/public'
}
maven {
  url 'https://maven.fabric.io/public'
}
maven {
  url 'https://raw.github.com/navisens/Android-Plugin/repositories'
}
```

3. Open up your **module's gradle buildscript**.
4. Add the following into your **dependencies**:

```gradle
compile group: "com.navisens", name: "motiondnaapi", version: "1.1.1-SNAPSHOT", changing: true
compile 'org.altbeacon:android-beacon-library:2.+'
compile 'com.navisens:navisenscore:2.1.0'
```

You should check out what the latest version of our Navisens SDK and `NavisensCore` are so you can use the latest stable version.

## Implementing `NavisensPlugin`

Now that we are done setting up, we will briefly discuss how `NavisensCore` works, and how the `NavisensPlugin` fits in with all of this.

The `NavisensCore` is in charge of starting up our Navisens SDK using the provided SDK key. After it has instantiated our SDK, it begins awaiting plugins. Anytime a user wants to use a plugin, the user must invoke the function [`init(NavisensPlugin)`](https://github.com/navisens/Android-Plugin/tree/master/navisenscore#public-t-extends-navisensplugin-t-initclasst-navisensplugin-object-args). Afterwards, all control of the Navisens SDK is handed off to the plugins. This is very important! It is entirely up to the plugins system to invoke and Navisens SDK functions. While the `NavisensCore` will run the SDK, a user would need a plugin in order to set the user's location, etc.

When the `NavisensCore` is requested to instantiate a new `NavisensPlugin`, it will first call the constructor of the requested plugin. Immediately afterwards, it calls the plugin's own `init` method, passing in any of the user's parameters or arguments.

To shut down the system, `NavisensCore` will request all plugins to stop via the `stop` method, before killing the Navisens SDK.

Below, we will present the `NavisensPlugin` interface. The `init` and `stop` methods are described below, while the remaining methods are described in the [using Navisens data and interplugin communication](#using-navisens-data-and-interplugin-communication) section.

```java
public interface NavisensPlugin {
    boolean init(NavisensCore core, Object[] args);
    boolean stop();
    void receiveMotionDna(MotionDna motionDna);
    void receiveNetworkData(MotionDna motionDna);
    void receiveNetworkData(MotionDna.NetworkCode networkCode, Map<String, ? extends Object> map);
    void receivePluginData(String identifier, Object data);
    void reportError(MotionDna.ErrorCode errorCode, String s);
}
```

The lifecycle of a plugin is very simple.
1. (External) `NavisensCore.init` is called, requesting the plugin
2. The plugin constructor is called
3. The plugin's `init` method is called
4. (External) A request to stop the plugin is made 
5. The plugin's `stop` method is called
6. All of the resources that were requested by the plugin are discontinued and released
7. The plugin reference is dropped from the core

#### `boolean init(NavisensCore core, Object[] args)`

The `init` method is guaranteed to be called shortly after the constructor is called. A reference to the `NavisensCore` is passed in. You should save a reference to this object for later use. The `args` parameter contains any arguments that were passed in by the user. These may be used to further customize the behavior of a plugin.

See the [using NavisensCore](#using-navisenscore) below for details on what methods can be called from the `NavisensCore`.

At the end, you must return `true` if everything was initialized correctly. If you return `false`, the core will call the `stop` function shortly afterwards, terminating the plugin lifecycle.

Here are some methods of `NavisensCore` that you should call in the `init` method:

* [`core.subscribe`](#void-subscribenavisensplugin-plugin-int-which) can be used to request resources, or listen for certain types of events. These events include data like the core motion statistics for user location, network data for messages from other devices, or even errors from the SDK.

**Examples**

If you want to subscribe to all `MotionDna` from both the current device stream, all online devices, and any errors, use this:

```java
  core.subscribe(this, NavisensCore.MOTION_DNA | NavisensCore.NETWORK_DNA | NavisensCore.ERRORS);
```

If you do not want to subscribe to anything, you can ignore the method call. You can also do this:

```java
  core.subscribe(this, NavisensCore.NOTHING);
```

* [`core.getSettings`](#navisenssettings-getsettings) is used to request the core apply certain settings. The settings object can also be used to force a setting.

**Examples**

If you want to increase the refresh rate from 100 ms to 50 ms, you can do this:

```java
  core.getSettings().requestCallbackRate(50);
```

However, if another plugin already called `core.getSettings().requestCallbackRate(10)`, then the above call will not do anything, since 10 ms is at least better than 50 ms.

You can optionally force a state. Note that if you do so, however, the order of execution **does** matter, and should be noted in your documentation.

```java
  core.getSettings().overrideCallbackRate(50);
```

This call will force the state to 50 ms, even if another plugin had originally requested 10 ms.

* [`core.getMotionDna()`](#motiondnaappliaction-getmotiondna) can be used to access even lower-level settings at the base SDK. Check out the API for more information on what you can do.

* [`core.applySettings()`](#void-applysettings) is used to commit all settings requested and apply them to the SDK.

#### `boolean stop()`

This method will terminate a plugin. You must call `core.remove(this)` in this method so the `NavisensCore` will drop the reference to this plugin, to prevent memory leaks (unless your plugin can't be stopped at the current time).

If you return `true` from this method, you must release all resources, as `NavisensCore` will drop all references of this plugin, so you want to make sure not to have any memory leaks. If you return `false`, then you are telling the user that they cannot end this plugin at the current time, and that they should take action to resolve why.

## Using `NavisensCore`

The `NavisensCore` is the central interlocutor between all plugins.

Here is a brief API of features that `NavisensPlugin`s may use. These are **not** listed alongside the API presented [here](https://github.com/navisens/Android-Plugin/tree/master/navisenscore#api) as they are designed to be used by plugins exclusively, although you may call them as normal if necessary.

#### `void subscribe(NavisensPlugin plugin, int which)`

This method subscribes a plugin to different resource streams. There are currently five resource streams as listed below:

* `MOTION_DNA` is the base stream of `MotionDna` for all location data of the current device
* `NETWORK_DNA` is a network stream of `MotionDna` of other devices that are on the same network (see the [NaviShare](https://github.com/navisens/Android-Plugin/tree/master/navishare) plugin for more info).
* `NETWORK_DATA` is a network stream of strings from other devices that are on the same network (see the [NaviShare](https://github.com/navisens/Android-Plugin/tree/master/navishare) plugin for more info).
* `PLUGIN_DATA` is an internal stream of raw tagged objects used to communicate between plugins.
* `ERRORS` is an error stream and is recommended. See [reportError](https://github.com/navisens/NaviDocs/blob/master/API.Android.md#void-reporterrorerrorcode-errorcode-string-errordescription) for more information.

All streams are processed on their own thread, but all together. If one of your methods is expected to take a non-trivial amount of time to execute, it is recommended that you start a new thread to offload the work, so as to not stall the remaining plugins who are also awaiting resources from the corresponding streams. Also, the stream data is not copied to each plugin, so changing the resource data object may have unintended consequences.

The stream values can be mathematically or'd together to combine multiple streams (see example below). There are two additional symbolic streams:

* `NOTHING` is no streams
* `ALL` is all streams

**Examples**

If you want only `MOTION_DNA`, use this:

```java
  core.subscribe(this, NavisensCore.MOTION_DNA);
```

If you want both `MOTION_DNA` and `ERRORS`, use this:

```java
  core.subscribe(this, NavisensCore.MOTION_DNA | NavisensCore.ERRORS);
```

If you want everything except the `PLUGIN_DATA` stream, you can do this too:

```java
  core.subscribe(this, NavisensCore.ALL ^ NavisensCore.PLUGIN_DATA);
```

#### `void unsubscribe(NavisensPlugin plugin, int which)`

This behaves the same as the above `subscribe` method, except that it stops providing the resources to the plugin.

#### `void broadcast(String identifier, Object data)`

This method is used to communicate with other plugins. Any plugin that is currently subscribed to the resource `PLUGIN_DATA` will receive both the identifier, along with the Object array. The identifier is purely used by plugins to determine if the data being presented is something they recognize and can process. Since there is only one `PLUGIN_DATA` stream, the `identifier` is designed as a secondary filter for this stream in the case that you have multiple mutually independent sets of `PLUGIN_DATA` resource-subscribing plugins.

#### `MotionDnaApplication getMotionDna()`

This method retrieves the base Navisens SDK object, which can be used to make low-level calls at the SDK level.

#### `NavisensSettings getSettings()`

This method retrieves a shared settings object that all plugins can use to request various settings. The settings object will determine whether or not a requested setting will be applied. After settings are selected, make sure to call [`applySettings`](#void-applysettings) to commit changes. There are two types of methods to be called on the returned settings object: `request` methods and `override` methods.

When using the `request` methods, the settings object guarantees that the requested settings will be applied *only if* they are of equal or greater precedence than the currently applied setting. For example, when requesting a callback rate, if the current callback rate is 100 ms, and a request of 50 ms is issued, then the callback rate will be updated to 50 ms. However, if the current callback rate is 10 ms, and a request of 50 ms is issued, then the request will not change anything, since 10 ms is at least as good as 50 ms.

The behavior is different if you use the `override` methods, which will set the value regardless.

Finally, certain settings don't make sense to have a precedence, and thus will always behave like `override` methods, even when using the `request` version. An example is setting the host ip when using a shared server.

Below is listed the methods currently accepted. More low-level settings can be accessed through [`getMotionDna`](#motiondnaapplication-getmotiondna). Note that a few other settings are set internally, but they are mainly for ease-of-use, and should not affect your plugin unless you explicitly require such settings.

```java
public static class NavisensSettings {
    public void requestARMode();
    public void overrideARMode(boolean mode);

    public void requestCallbackRate(int rate);
    public void overrideCallbackRate(int rate);

    public void requestGlobalMode();
    public void overrideEstimationMode(MotionDna.EstimationMode mode);

    public void requestPositioningMode(MotionDna.ExternalPositioningState mode);
    public void overridePositioningMode(MotionDna.ExternalPositioningState mode);

    public void requestNetworkRate(int rate);
    public void overrideNetworkRate(int rate);

    public void requestPowerMode(MotionDna.PowerConsumptionMode mode);
    public void overridePowerMode(MotionDna.PowerConsumptionMode mode);

    public void requestHost(String host, String port);
    public void overrideHost(String host, String port);

    public void requestRoom(String room);
    public void overrideRoom(String room);
}
```

As always, since all of the source code for the plugins system is open-source, if you are unsure of anything, go ahead and look at the source files.

#### `void applySettings()`

This method simply commits any settings you requested (or overrode) permanently. These settings are pushed to the main SDK, and applied immediately. If you do not call this method, none of the settings you requested earlier will take effect.

#### `void startServices()`

This method should be called if your plugin needs to begin running any background services. Only two services are started at the current time. The first is a global localization algorithm, which will only run if you request that the estimation mode be set to global using `NavisensSettings.requestGlobalMode`. The second connects to a server, if you are sharing your location with other devices, and only runs if you have set a non-null host and port.

## Using Navisens Data and Interplugin Communication

Once your plugin is running, you will begin receiving any requested resources from subscribed streams. The following summarizes the resources you get.

#### `void receiveMotionDna(MotionDna motionDna)`

This provides the `MotionDna` resource from the current device. Check out [here](https://github.com/navisens/NaviDocs/blob/master/API.Android.md#getters) for information on the most commonly used components of the `MotionDna` object. For more information about the callback, see [here](https://github.com/navisens/NaviDocs/blob/master/API.Android.md#void-receivemotiondnamotiondna-motiondna).

#### `void receiveNetworkData(MotionDna motionDna)`

This provides the `MotionDna` resource from all other devices in the current network room, if connected to a network room. Check out [here](https://github.com/navisens/NaviDocs/blob/master/API.Android.md#getters) for information on the most commonly used components of the `MotionDna` object. For more information about the callback, see [here](https://github.com/navisens/NaviDocs/blob/master/API.Android.md#void-receivenetworkdatamotiondna-motiondna).

#### `void receiveNetworkData(MotionDna.NetworkCode networkCode, Map<String, ? extends Object> map)`

This provides information when working with NetworkData, including queries for devices, raw network data, or any network-related errors. For more information, see [here](https://github.com/navisens/NaviDocs/blob/master/API.Android.md#void-receivenetworkdatanetworkcode-opcode-map-payload).

#### `void receivePluginData(String identifier, Object data)`

This is a unique callback that deals with communications between plugins. All plugins using this stream will receive data broadcast from any other plugin. The identifier can be used to identify what type of data is being sent, and from what plugin. It is recommended that to make your identifier unique, you prefix all of your identifiers with the identifier of your project.

#### `void reportError(MotionDna.ErrorCode errorCode, String s)`

This reports any errors that may occur at the SDK level. For more information about the callback, see [here](https://github.com/navisens/NaviDocs/blob/master/API.Android.md#void-reporterrorerrorcode-errorcode-string-errordescription).
