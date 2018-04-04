# Navi Help Me

This tutorial is designed to lead you and/or a group of people through using a plugin and designing custom plugins.

## Preamble

Our objective is to build an app using some of plugin pieces, and then extend the functionality provided with our own custom plugin and custom behavior!

## Reference Code

### Fig. 1
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

### Fig. 2a
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

### Fig. 2b
```diff
dependencies {
    compile group: "com.navisens", name: "motiondnaapi", version: "1.2.2-SNAPSHOT", changing: true
    compile 'org.altbeacon:android-beacon-library:2.+'
+    compile 'com.navisens:navisenscore:3.1.0'
+    compile 'com.navisens:navishare:0.3.0'

    androidTestCompile('com.android.support.test.espresso:espresso-core:2.2.2', {
        exclude group: 'com.android.support', module: 'support-annotations'
    })
    compile 'com.android.support:appcompat-v7:25.3.1'
    compile 'com.android.support.constraint:constraint-layout:1.0.2'
    testCompile 'junit:junit:4.12'
}
```

### Fig. 3
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

### Fig. 4
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

### Fig. 5
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

### Fig. 6
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

### Fig. 7
```diff
    @Override
    public boolean init(NavisensCore navisensCore, Object[] objects) {
        core = navisensCore;

+        core.getSettings().requestCallbackRate(0);
+        core.applySettings();

        return true;
    }
```

### Fig. 8a
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

### Fig. 8b
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

### Fig. 8c
```diff
    @Override
    public boolean stop() {
        core.remove(this);
+        core.broadcast(PLUGIN_IDENTIFIER, NavisensCore.OPERATION_STOP);
        return true;
    }
```

### Fig. 9
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

### Fig. 10a
```diff
public class NaviTutorial implements NavisensPlugin {
    private static final String PLUGIN_IDENTIFIER = "com.navisens.other.navitutorial";

    private NavisensCore core;
+    private MotionDna.MotionType motionType;
    
...
    
}
```

### Fig. 10b
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

### Fig. 10c
```diff
    @Override
    public void receiveMotionDna(MotionDna motionDna) {
+        motionType = motionDna.getMotion().motionType;
    }
```

### Fig. 10d
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
