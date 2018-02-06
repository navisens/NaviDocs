# Publishing NavisensPlugins for Android

Publishing plugins for Android is very easy.

Once you have a working `NavisensPlugin`, all you need to do is configure a maven repository.

First, you will need to set up a gradle environment to compile your code into a library. The recommended option is to create a new module with a fresh gradle buildscript. This method takes more work, but is more flexible than simply modifying the main gradle buildscript.

Whatever way you choose, you will need to open up the **module gradle buildscript**

A sample will be provided at the bottom. You should take the following steps to modify your buildscript

### 1. Ensure that the project is converted to compilation of a library by using

```gradle
apply plugin: 'com.android.library'
```

This should replace any other `apply plugin: 'com.android.application'` calls, or otherwise.

### 2. Make sure you don't have an remaining Android application flags.

For example, if you go in `android`, then `defaultConfig`, you should remove the `applicationId` line if it exists.

### 3. Add a `sourceSets` tag

If your files are in a different module, or you have files you wish to ignore (such as `MainActivity` or other testing files), you should add a `sourceSets` tag as follows:

```gradle
android {
    // ...
    
    sourceSets {
        main {
            // If you have any assets you must package with your library
            assets {
                srcDir '../app/src/main/assets/'
            }
            java {
                // All of your source files
                srcDir '../app/src/main/java/PATH/TO/YOUR/SOURCE/CODE/'
                
                // Any files you wish to exclude that are also inside the source code folder
                exclude '**/MainActivity.java'
            }
        }
    }
}
```

If you are stuck, just look online for more information.

### 4. Next, go to your `dependencies` section.

You must now change all of the dependencies you used from `compile` to `provided`. The provided keyword means your library should not also have a compiled reference (as is required if you are building an application). However, this also means that whoever wants to use your library must also include all of the dependencies you used, so keep track of them and make sure to include those dependencies wherever you use your library.

### 5. Finally, we will include the code that will tell gradle to automagically build our maven repo.

To do that, at the very bottom of the file, add the following:

```gradle
apply plugin: 'maven'

uploadArchives {
    repositories.mavenDeployer {
        repository(url: "file://PATH/TO/YOUR/OUTPUT/DIRECTORY/")
        pom.project {
            groupId 'identifier'      // something like com.example
            artifactId 'pluginName'   // the name of your plugin
            version "1.0.0"           // use semantic versioning please!
        }
    }
}
```

All done! Make sure the output directory you specified exists. Now go along to the gradle scripts on the right. Make sure you are selecting the correct module. In the newly created **upload** section, you should find a new gradle task called **uploadArchives**. Go ahead and run that, and your maven repo will be generated at the folder specified.

Now push your library up to a public repository like Github in order to share it with everyone!

Sample gradle buildscript:

```gradle
apply plugin: 'com.android.library'

android {
    compileSdkVersion 25
    buildToolsVersion "25.0.3"
    defaultConfig {
        minSdkVersion 21
        targetSdkVersion 25
        versionCode 1
        versionName "1.0"
        testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"
    }
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }

    sourceSets {
        main {
            assets {
                srcDir '../app/src/main/assets/'
            }
            java {
                srcDir '../app/src/main/java/com/navisens/pojostick/navisensmaps/'
                exclude '**/MainActivity.java'
            }
        }
    }
}

dependencies {
    provided group: "com.navisens", name: "motiondnaapi", version: "1.0.0-SNAPSHOT", changing: true
    compile 'org.altbeacon:android-beacon-library:2.+'
    compile 'com.android.support:appcompat-v7:25.3.1'
    testCompile 'junit:junit:4.12'
    provided 'com.navisens:navisenscore:2.0.2'
}

apply plugin: 'maven'

uploadArchives {
    repositories.mavenDeployer {
        def navisensPath = file(getProperty('aar.navisensPath'))
        repository(url: "file://${navisensPath.absolutePath}")
        pom.project {
            groupId 'com.navisens'
            artifactId 'navisensmaps'
            version "1.0.0"
        }
    }
}
```
