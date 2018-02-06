# Publishing NavisensPlugins for Android

Publishing plugins for Android is very easy.

Once you have a working `NavisensPlugin`, all you need to do is configure a maven repository.

First, you will need to set up a gradle environment to compile your code into a library. The recommended option is to create a new module with a fresh gradle buildscript. This method takes more work, but is more flexible than simply modifying the main gradle buildscript.

Whatever way you choose, you will need to open up the **module gradle buildscript**
