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
