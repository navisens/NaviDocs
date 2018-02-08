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
  core.subscribe(this, NavisensCore.MOTION_DNA | NavisensCore.NETWORK_DNA | NavisensCore.ERRORS);
```

If you do not want to subscribe to anything, you can ignore the method call. You can also do this:

```swift
  core.subscribe(this, NavisensCore.NOTHING);
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


