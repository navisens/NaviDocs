# Navi Help Me

This tutorial is designed to lead you and/or a group of people through using a plugin and designing custom plugins.

## Preamble

Our objective is to build an app using some Navisens plugin pieces, and then extend the functionality by writing our own custom plugin!

The app we will build is a simple space-shooter style app, where you "fly" your ship by walking around with a mobile device, and tap to shoot at other people. We will add an additional plugin which allows us to respond to certain events, such as adding haptic feedback everytime we shoot a bullet. At the end, we can optionally package our plugin and publish it online.

## Gradle Setup

The first step when starting a project is to set up our buildscripts so we can compile and dependencies. Start by creating a new gradle project. You can use Android Studio. Make an application with an empty activity. And then open up your project buildscript.

In the `allprojects` section, inside `repositories`, add the maven repositories as below. This is where gradle will search for our dependencies.

### Project Buildscript
```gradle
allprojects {
    repositories {
        jcenter()

        maven {
            url 'https://oss.sonatype.org/content/groups/public'
        }
        maven {
            url 'https://maven.fabric.io/public'
        }
        maven {
            url 'https://raw.github.com/navisens/Android-Plugin/repositories'
        }
    }
}
```

Next, we want to open up our module buildscript. In here, we need to import the `motiondnaapi` dependencies. This is our main SDK. While you may choose to develop using our native SDK, the plugins provide an open-source framework so you can more easily develop the app.

### Module:app buildscript
```gradle
dependencies {
    compile group: "com.navisens", name: "motiondnaapi", version: "1.2.2-SNAPSHOT", changing: true
    compile 'org.altbeacon:android-beacon-library:2.+'

    androidTestCompile('com.android.support.test.espresso:espresso-core:2.2.2', {
        exclude group: 'com.android.support', module: 'support-annotations'
    })
    compile 'com.android.support:appcompat-v7:25.3.1'
    compile 'com.android.support.constraint:constraint-layout:1.0.2'
    testCompile 'junit:junit:4.12'
}
```

In order to use our plugins, we need to import some additional dependencies. Go ahead and add the dependencies below. Note that there is likely a more updated version of the SDK and plugins, and you should go to the [repositories](https://github.com/navisens/Android-Plugin) to find the latest version of each plugin. For now, we will be adding the `NavisensCore`, `NaviShare`, and `NaviHelp` plugins.

The `NavisensCore` is the base plugin that all other plugins will depend on. If you are using the plugin system, then you must add this dependency. All other dependencies are optional unless a plugin specifically requires another plugin. For example, the `NaviHelp` plugin requires the `NaviShare` plugin in order to function properly.

### Module:app buildscript
```diff
dependencies {
    compile group: "com.navisens", name: "motiondnaapi", version: "1.2.2-SNAPSHOT", changing: true
    compile 'org.altbeacon:android-beacon-library:2.+'
+    compile 'com.navisens:navisenscore:3.1.0'
+    compile 'com.navisens:navishare:0.3.0'
+    compile 'com.navisens:navihelp:1.2.1'

    androidTestCompile('com.android.support.test.espresso:espresso-core:2.2.2', {
        exclude group: 'com.android.support', module: 'support-annotations'
    })
    compile 'com.android.support:appcompat-v7:25.3.1'
    compile 'com.android.support.constraint:constraint-layout:1.0.2'
    testCompile 'junit:junit:4.12'
}
```

## Navisens Plugins Basic Usage

Now we can get to writing code. We can start with the basic plugins usage. The first step is to alwasy create a new `NavisensCore`. You should do this at the beginning of app creation, and use the same reference throughout the entire application. For now, we'll just create a local `NavisensCore`, since we won't be using multiple activities.

The `NavisensCore` requires an API key. You can apply for one on our [website](https://navisens.com/#contact). You must also pass in the current activity. This is used by the native SDK to access permissions and other resources.

In order to declare a new plugin, we use the `core.init` method, and pass in a _class reference_ to the plugin we wish to instantiate. The returned object is then an instance of the _class_ passed in. We can create a `NaviShare` for example as follows. With this code, your app should run without errors. If there are any, you should resolve them now before moving on.

### MainActivity.java
```java
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        if (savedInstanceState == null) {
            final NavisensCore core = new NavisensCore(MOTIONDNA_KEY, this);
            final NaviShare share = core.init(NaviShare.class);
        }
    }
```

## Navisens Plugins NaviHelp

Now we can move on to using the `NaviHelp` plugin. If you wanted to learn more about the `NaviHelp` plugin, you might start by going to the plugins page about [`NaviHelp`](https://github.com/navisens/Android-Plugin/tree/master/navihelp). On this page, you can see the latest version of `NaviHelp`, as well as any additional setup required to use the class. The docs say that we need to pass in a reference to `NaviShare` when we create a `NaviHelp`. We can pass in additional arguments to `core.init` after the initial _class reference_. 

Additionally, since `core.init` will return a `NaviHelp` object, we can chain additional methods afterwards that participate in further setting up the plugin. For example, below, we continue setting up our spaceship username and color, and finally we call the `start` method to begin the app. We need to add the `NaviHelp` object to a view so we can display it. Since `NaviHelp` is a fragment, we use the fragment manager to add our `NaviHelp` to the main content viewport.

### MainActivity.java
```diff
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        if (savedInstanceState == null) {
            final NavisensCore core = new NavisensCore(MOTIONDNA_KEY, this);
            final NaviShare share = core.init(NaviShare.class);
            
+            final NaviHelp help = core.init(NaviHelp.class, share)
+                    .setUsername("user's name")
+                    .setColor("#ffaa00")
+                    .start();
+            getFragmentManager().beginTransaction().add(android.R.id.content, help).commit();
        }
    }
```

By this point, you should be able to run the app on most modern Android devices. Go ahead and walk around the room to fly your ship, and tap on the screen to fire a laser. If there are multiple developers, and all devices are connected to the internet, then you should see each other too! If you get hit with a laser, your phone will vibrate.

Note: The view may feel like it is stuttering. This is intentional, and we will learn how to fix this in the next section.

Warning: The spaceships are rendered with WebGL, which may use up battery life quickly as it uses GPU!

## New Navisens Plugin: Fixes

Now that you have tried running the app, you should already be able to tell that there was an issue. Your motions were stuttering, occuring at a lower than optimal frame rate. The cause of this is that the default rate at which the native SDK reports location updates is 10-25 Hz. This is too slow for our uses, since we rely on realtime movements. So we should speed it up.

We might fix this issue by simply telling the native SDK to use a higher callback rate via `core.getMotionDna().setCallbackRate()`. However, there is an issue doing it this way. If we set a callback rate this way, and another plugin also sets a callback rate, the SDK will only use the last request, even if it happens to be worse. Fortunately, the plugins system deals with this problem. If a plugin wants a callback rate, it may request a certain rate. However, the requested rate will only apply if the current SDK callback rate is at least as good as the requested value. For example, if the SDK callback rate is already responding every 50 milliseconds, then having a plugin request 100 milliseconds will result in nothing happening, since 50 milliseconds is certainly a better callback rate than 100 milliseconds. This is the motivation we have for using the plugins system.

So how do we create a new plugin? First, let's create a new module. We will call this module `NaviTutorial`. It will be an `Android Library`, so there will be no activity. Next, we need to set up all of the dependencies. In the module dependencies, we only need to import the `motiondnaapi` dependencies, and then also the `NavisensCore` and `NaviHelp`. Our plugin will not need access to `NaviShare`.

Next, we will create a new class. This class will add the features we want. It must implement NavisensPlugin. Add all required methods. It is good practice to add an identifier for this plugin as a string, so other plugins will be able to communicate with this plugin.

The general life cycle of a Navisens Plugin starts with the object being constructed, and then the `init` method being called on the object. Upon successful initialization, `init` should return `true`, otherwise the object will be destroyed immediately. When the plugin is done, the `stop` method is called. If `true` is returned, then the core will release all resources that the plugin is using, and remove all references to the plugin, otherwise the plugin is not successfully destroyed, and `stop` must be called again.

We should keep a reference to the core as follows.

### NaviTutorial.java
```java
public class NaviTutorial implements NavisensPlugin {
    private static final String PLUGIN_IDENTIFIER = "com.navisens.navidocs.tutorials.navitutorial";

    private NavisensCore core;

    @Override
    public boolean init(NavisensCore navisensCore, Object[] objects) {
        core = navisensCore;
        return true;
    }

    @Override
    public boolean stop() {
        core.remove(this);
        return true;
    }

    @Override
    public void receiveMotionDna(MotionDna motionDna) {}

    @Override
    public void receiveNetworkData(MotionDna motionDna) {}

    @Override
    public void receiveNetworkData(MotionDna.NetworkCode networkCode, Map<String, ? extends Object> map) {}

    @Override
    public void receivePluginData(String s, int i, Object... objects) {}

    @Override
    public void reportError(MotionDna.ErrorCode errorCode, String s) {}
}
```

Our objective is to request a faster callback rate, so the game feels more smooth. We can do that by accessing the settings and requesting a callback rate of 0 milliseconds. This will tell the SDK to respond as fast as it is capable of doing (whenever a new location estimation is computed). Next, we need to commit this change with `applySettings`.

### NaviTutorial.java
```diff
    @Override
    public boolean init(NavisensCore navisensCore, Object[] objects) {
        core = navisensCore;

+        core.getSettings().requestCallbackRate(0);
+        core.applySettings();

        return true;
    }
```

At this point, we have a second plugin that will fix the issue we had. Now, we just need to add this plugin to our main app. First, we need to return to the main app's module buildscript. We can add the module dependency this way.

### Module:app buildscript
```diff
dependencies {
    compile group: "com.navisens", name: "motiondnaapi", version: "1.2.2-SNAPSHOT", changing: true
    compile 'org.altbeacon:android-beacon-library:2.+'
    compile 'com.navisens:navisenscore:3.1.0'
    compile 'com.navisens:navishare:0.3.0'
+    compile project(':navitutorial')

    androidTestCompile('com.android.support.test.espresso:espresso-core:2.2.2', {
        exclude group: 'com.android.support', module: 'support-annotations'
    })
    compile 'com.android.support:appcompat-v7:25.3.1'
    compile 'com.android.support.constraint:constraint-layout:1.0.2'
    testCompile 'junit:junit:4.12'
}
```

Finally, go back to our `MainActivity.java` and now tell the `core` to `init` our new `NaviTutorial` plugin. When we run the app now, you should see that the application feels much smoother and react closer to realtime.

## New Navisens Plugin: Augments

Now, we will add additional functionalities. We will start with haptic feedback. We want the phone to vibrate whenever we shoot a laser. Fortunately, the `NaviHelp` plugin broadcasts every time the user shoots a laser. We can capture this with the interplugin system in `NaviTutorial`. First, we need to acquire the resources necessary. We do this by using `core.subscribe` to begin listening for a specific resource. We want `PLUGIN_DATA`, so we can tap into interplugin communication channel. Then, we should `broadcast` a message to all plugins that this plugin has been initialized.

### NaviTutorial.java
```diff
    @Override
    public boolean init(NavisensCore navisensCore, Object[] objects) {
        core = navisensCore;

+        core.subscribe(this, NavisensCore.PLUGIN_DATA);
+        core.broadcast(PLUGIN_IDENTIFIER, NavisensCore.OPERATION_INIT);

        core.getSettings().requestCallbackRate(0);
        core.applySettings();

        return true;
    }
```

It is also good practice to broadcast to all plugins when we stop.

### NaviTutorial.java
```diff
    @Override
    public boolean stop() {
        core.remove(this);
+        core.broadcast(PLUGIN_IDENTIFIER, NavisensCore.OPERATION_STOP);
        return true;
    }
```

And finally, we can go to the callback method `receivePluginData` to handle interplugin communication. The callback has multiple parameters. The first is the identifier of the broadcaster, and the second is an additional integer to help differentiate different commands. Further arguments may be provided if needed.

First, we can respond to the the core `OPERATION_INIT` by acknowledging the specific plugin. This resolves any conflicts in calling order of the plugins.

### NaviTutorial.java
```java
    @Override
    public void receivePluginData(String s, int i, Object... objects) {
        switch (s) {
            case NaviHelp.PLUGIN_IDENTIFIER:
                switch (i) {
                    case NavisensCore.OPERATION_INIT:
                        core.broadcast(PLUGIN_IDENTIFIER, NavisensCore.OPERATION_ACK, NaviHelp.PLUGIN_IDENTIFIER);
                        break;
                }
                break;
        }
    }
```

Next, we can respond to the `NaviHelp`-specific `OPERATION_SHOOT`. We can broadcast the `NaviHelp.OPERATION_VIBRATE` message. We pass an additional argument the number of milliseconds to vibrate for. 25 milliseconds is good, or if using an older device, a longer vibration time might be required if the device is not capable of short vibrations.

### NaviTutorial.java
```diff
    @Override
    public void receivePluginData(String s, int i, Object... objects) {
        switch (s) {
            case NaviHelp.PLUGIN_IDENTIFIER:
                switch (i) {
                    case NavisensCore.OPERATION_INIT:
                        core.broadcast(PLUGIN_IDENTIFIER, NavisensCore.OPERATION_ACK, NaviHelp.PLUGIN_IDENTIFIER);
                        break;
+                    case NaviHelp.OPERATION_SHOOT:
+                        core.broadcast(PLUGIN_IDENTIFIER, NaviHelp.OPERATION_VIBRATE, 25);
+                        break;
                }
                break;
        }
    }
```

Now, if we run the plugin, whenever we tap and fire a laser, we should feel a short vibration.

## New Navisens Plugin: Addendum

Finally, for fun, we will add one more feature. If the user holds down instead, and stops moving, they can charge up a large laser that is fired when they release. To do this, we will need to know if the user has stopped moving. We can do that by tracking the user's `MotionType`. This is provided by the MotionDna SDK, and tells us if the user is `STATIONARY`, `FIDGETING`, or moving `FORWARD`.

### NaviTutorial.java
```diff
public class NaviTutorial implements NavisensPlugin {
    private static final String PLUGIN_IDENTIFIER = "com.navisens.other.navitutorial";

    private NavisensCore core;
+    private MotionDna.MotionType motionType;
    
...
    
}
```

Next, we'll need to subscribe to an additional resource: the device `MOTION_DNA`. In our `core.subscribe` call in `init`, we can add additional resources by "or"-ing two different resources as below.

### NaviTutorial.java
```diff
    @Override
    public boolean init(NavisensCore navisensCore, Object[] objects) {
        core = navisensCore;

-        core.subscribe(this, NavisensCore.PLUGIN_DATA);
+        core.subscribe(this, NavisensCore.MOTION_DNA | NavisensCore.PLUGIN_DATA);
        core.broadcast(PLUGIN_IDENTIFIER, NavisensCore.OPERATION_INIT);

        core.getSettings().requestCallbackRate(0);
        core.applySettings();

        return true;
    }
```

Now, we can update the `motionType` variable to keep track of the user's motion by using the `receiveMotionDna` callback.

### NaviTutorial.java
```diff
    @Override
    public void receiveMotionDna(MotionDna motionDna) {
+        motionType = motionDna.getMotion().motionType;
    }
```

And finally, back in the `receivePluginData`, we add an additional clause to our switch-case. If we try to do a `NaviHelp.OPERATION_CHARGE`, we want to first check the motion type to make sure the user is standing still, and then we can respond by acknowledging the `NaviHelp.OPERATION_CHARGE` back. We should also add haptic feedback with this action too. Since this is a larger laser, we can vibrate longer to make it feel better.

### NaviTutorial.java
```diff
    @Override
    public void receivePluginData(String s, int i, Object... objects) {
        switch (s) {
            case NaviHelp.PLUGIN_IDENTIFIER:
                switch (i) {
                    case NavisensCore.OPERATION_INIT:
                        core.broadcast(PLUGIN_IDENTIFIER, NavisensCore.OPERATION_ACK, NaviHelp.PLUGIN_IDENTIFIER);
                        break;
                    case NaviHelp.OPERATION_SHOOT:
                        core.broadcast(PLUGIN_IDENTIFIER, NaviHelp.OPERATION_VIBRATE, 25);
                        break;
+                    case NaviHelp.OPERATION_CHARGE:
+                        if (motionType != MotionDna.MotionType.FORWARD) {
+                            core.broadcast(PLUGIN_IDENTIFIER, NaviHelp.OPERATION_CHARGE);
+                            core.broadcast(PLUGIN_IDENTIFIER, NaviHelp.OPERATION_VIBRATE, 1000);
+                        }
+                        break;
                }
                break;
        }
    }
```

At this point, you should have a working app. If you stand still and hold down, you will charge up a large beam that you can fire!

The last thing we might want to do is publish this plugin. You can learn more about how to do that [here](https://github.com/navisens/NaviDocs/blob/master/Tutorials/publishing-navisensplugins.Android.md).
