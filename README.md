# Add Security Exception to APK

In Android 7.0, Google introduced changes to the way user Certificate Authorities (CA) are trusted. These changes prevent third-parties from listening to network requests coming out of the application:
More info: 
1) https://developer.android.com/training/articles/security-config.html
2) http://android-developers.blogspot.com/2016/07/changes-to-trusted-certificate.html

This script injects into the APK network security exceptions that allow third-party software like Charles Proxy/Fiddler to listen to the network requests and responses of some Android applications.

## High Level Steps
1. Copy the Charles Certificate (aka the cert we want to trust) into the source file of the app. This could be any certificate, not just a Charles Proxy cert.
1. Trust the Charles Certificate via the `network_security_config.xml` file.
2. Register the `network_security_config.xml` file in the AndroidManifest.xml file.
3. Repackage the app with the jarsigner.

There may also be some "enable debug" mode stuff in here too.  Its worth noting here that you MUST use a Debug build of your app. See some [differences between Debug vs Production](https://stackoverflow.com/questions/38864358/difference-between-debug-and-release-apks) and the [Android Stuido docs](https://developer.android.com/studio/publish/preparing).

## Getting Started

Download the `addSecurityException.sh` script and the `network_security_config.xml`  file and place them in the same directory.

Note: the XML file may be optional, some apps already have a `network_security_config.xml` file.

### Prerequisites
- Certificate that will be re-encrypting traffic.  Be sure to get the Charles Root Certificate and not the Root Cert + Key. For Charles see [here](https://www.charlesproxy.com/documentation/using-charles/ssl-certificates/).

- `~/.android/debug.keystore`. If you don't have it you can try generating it https://stackoverflow.com/questions/8576732/there-is-no-debug-keystore-in-android-folder

APKTOOL is not needed anymore.

~~You will need `apktool` and the Android SDK installed~~

~~I recommend using `brew` on Mac to install `apktool`:~~

~~```brew install apktool```~~



## Usage

NOTE: Get your encrypting cert and add it to the `/res` directory and name it `cv_ca`.  
You can call it something else but you will need to change the name in `addSecurityExceptions.sh` and `network_security_config.xml`.

The script take two arguments: 
1) APK file path.
2) keystore file path (**optional** - Default is: ~/.android/debug.keystore )

### Examples

```
./addSecurityExceptions.sh myApp.apk

or

./addSecurityExceptions.sh myApp.apk ~/.android/debug.keystore

```

You should see a app called myApp_new.apk created.
