# zipalign
Creating an apk for public publishing

Follow the info on the Ionic Framework Website:
(Original article: http://ionicframework.com/docs/guide/publishing.html)

Android Publishing
To generate a release build for Android, we can use the following cordova cli command:

    $ cordova build --release android
This will generate a release build based on the settings in your config.xml. Your Ionic app will have preset default values in this file, but if you need to customize how your app is built, you can edit this file to fit your preferences. Check out the config.xml file documentation for more information.

Next, we can find our unsigned APK file in platforms/android/build/outputs/apk. In our example, the file was platforms/android/build/outputs/apk/HelloWorld-release-unsigned.apk. Now, we need to sign the unsigned APK and run an alignment utility on it to optimize it and prepare it for the app store. If you already have a signing key, skip these steps and use that one instead.

Let’s generate our private key using the keytool command that comes with the JDK. If this tool isn’t found, refer to the installation guide:

    $ keytool -genkey -v -keystore my-release-key.keystore -alias alias_name -keyalg RSA -keysize 2048 -validity 10000
You’ll first be prompted to create a password for the keystore. Then, answer the rest of the nice tools’s questions and when it’s all done, you should have a file called my-release-key.keystore created in the current directory.

Note: Make sure to save this file somewhere safe, if you lose it you won’t be able to submit updates to your app!

To sign the unsigned APK, run the jarsigner tool which is also included in the JDK:

    $ jarsigner -verbose -sigalg SHA1withRSA -digestalg SHA1 -keystore my-release-key.keystore HelloWorld-release-unsigned.apk alias_name
This signs the apk in place. Finally, we need to run the zip align tool to optimize the APK. The zipalign tool can be found in /path/to/Android/sdk/build-tools/VERSION/zipalign. For example, on OS X with Android Studio installed, zipalign is in ~/Library/Android/sdk/build-tools/VERSION/zipalign. 

And here I experienced my personal problem:

    /usr/local/Cellar/android-sdk/24.4.1_1/build-tools/23.0.1/zipalign: No such file or directory
    /usr/local/bin/zipalign: line 3: exec: /usr/local/Cellar/android-sdk/24.4.1_1/build-tools/23.0.1/zipalign: cannot execute: No such file or directory

As I opened the zipalign `program`in my texteditor I found that inside instead of some compiled code:

    #!/bin/bash
    TOOL="/usr/local/Cellar/android-sdk/24.4.1_1/build-tools/23.0.1/zipalign"
    exec "$TOOL" "$@"

On top, after executing the **wannabe program ** my computer was going crazy and starting to use almost all my CPU, etc.
To make a long story short:
As I figured out that a lot of developers have a problem with `zipalign` (http://GIYF.com) I decided to tell you my solution:
I was lookig for the latest version of zipalign on my computer, copied it inside my apk-folder and then fired this command:

    $ ./zipalign -v 4 android-release-unaligned.apk ibiza2016-1.apk

instead of the proposed one: 

    $ zipalign -v 4 HelloWorld-release-unsigned.apk HelloWorld.apk

And it worked!

If you only need `zipalign` to get your Android app ready for being published you may try to just use the enclosed zipalign tool instead of downloading the entire Android Studio.

Now we have our final release binary called HelloWorld.apk and we can release this on the Google Play Store for all the world to enjoy!

(There are a few other ways to sign APKs. Refer to the official Android App Signing documentation for more information.)

If you still experience problems you may just try to create a `gradle build` directly from Android Studio.

Using my tip is your own risk. I will not be responsible for any damges, etc.

Good luck with your app!
