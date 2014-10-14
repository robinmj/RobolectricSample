# RobolectricSample

This fork of [RobolectricSample](https://github.com/robolectric/RobolectricSample) has been modified to work with Android Studio 0.8 and Gradle. I've borrowed heavily from [avbk's "Testing with Robolectric within Android Studio" gist](https://gist.github.com/avbk/d82c44ed8bef5782dbe5).

## Getting Set Up

### 1. Import This Project

    a. Use `VCS -> Checkout from Version Control -> Git` to clone this project. Set *Vcs Repository URL* to `git@github.com:robinmj/RobolectricSample.git`
    b. AndroidStudio will ask you if you want to open the build.gradle file. Choose *Yes*. This will open the `Import Project from Gradle` dialog. Make sure "Use default gradle wrapper" is selected, then click *Yes* again.

### 2. Install android-20 sdk and tools

If you don't have the android-20 sdk and tools installed, you will see a "Failed to sync Gradle project" error message. You can then click "Install missing platform(s) and sync project" to fix this.
 
### 3. Setup Task to compile and include test classes
In Android Studio you need to add a new Run-Configuration.

    a. Click on `Run -> Edit Configurations`
    b. In the dialog press the `+` Symbol and select `Gradle`
    c. Setup the configuration as to following:
        * Name: `Build test classes`
        * Gradle Project: `RobolectricSample:app`
        * Tasks: `testClasses copyTestClasses`

Save this configuration by pressing OK.


### 4. Setup JUnit Run-Configuration
In Android Studio you need to add a new Run-Configuration.

    a. Click on `Run -> Edit Configurations`
    b. In the dialog press the `+` Symbol and select `JUnit`
    c. Setup the configuration as to following:
        * Name: `Robolectric`
        * Test kind: `All in package`
        * Package: `com.pivotallabs`
        * Search for tests: `In single module`
        * VM Options: ```-ea -Dandroid.assets=build/intermediates/assets/debug -Dandroid.manifest=build/intermediates/manifests/debug/AndroidManifest.xml -Dandroid.resources=build/intermediates/res/debug``` (if you want to test a different flavour or buildtype, change `debug` in those paths accordingly)
        * Working Directory: `$MODULE_DIR$`
        * Use classpath of module: `app`
        * Check `Use alternate JRE` and select your default Java VM, e.g. `/usr/lib/jvm/java-1.7.0-openjdk-i386`
        * Under `Before Launch` press `+` and add `Build test classes` by selecting `Run Another Configuration`

Save this configuration by pressing OK.

### 5. Include a modified version of android.jar

Obviously you need to include the `android.jar`, but if you would do so you will fail with the famous stub-exception:
```
!!! JUnit version 3.8 or later expected:

java.lang.RuntimeException: Stub!
    at junit.runner.BaseTestRunner.<init>(BaseTestRunner.java:5)
    at junit.textui.TestRunner.<init>(TestRunner.java:54)
    at junit.textui.TestRunner.<init>(TestRunner.java:48)
    at junit.textui.TestRunner.<init>(TestRunner.java:41)
    ...
```

Unfortunately, newer versions of Android Studio don't provide a means to change the build path order
 in a way that makes it possible to override android system libraries. I think there's a way to do
 this by foregoing the android gradle plugin in favor of the java plugin, but I haven't figured it
 out yet. Instead, we'll just strip the JUnit classes out of android.jar

Open up a terminal (sorry Windows-Users, you have to figure out a more GUI-ish way I guess ;-) ) and enter the following:

```bash
me@localhost$ cd RobolectricSample/app
me@localhost$ mkdir testlibs
me@localhost$ cp $ANDROID_HOME/platforms/android-20/android.jar testlibs/android-20-stripped.jar
me@localhost$ zip --delete testlibs/android-20-stripped.jar junit*
deleting: junit/
deleting: junit/framework/
deleting: junit/framework/TestCase.class
deleting: junit/framework/Test.class
deleting: junit/framework/AssertionFailedError.class
deleting: junit/framework/ComparisonFailure.class
deleting: junit/framework/TestFailure.class
deleting: junit/framework/Assert.class
deleting: junit/framework/TestSuite.class
deleting: junit/framework/TestListener.class
deleting: junit/framework/Protectable.class
deleting: junit/framework/TestResult.class
deleting: junit/runner/
deleting: junit/runner/Version.class
deleting: junit/runner/TestSuiteLoader.class
deleting: junit/runner/BaseTestRunner.class
```
(note that `$ANDROID_HOME` points to the installation path of my Android SDK)

Now you should be able to run and debug your tests from Android Studio by running the `Robolectric` Configuration (due to [this bug](https://code.google.com/p/android/issues/detail?id=70959) the test might fail on the first try, but should succeed on the second try )
