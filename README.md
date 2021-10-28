# What's the deal with Adding a Security Exception to an APK?
In Android 7.0, Google introduced changes to the way user Certificate Authorities (CA) are trusted. These changes prevent third-parties from listening to network requests coming out of the application:
More info: 
1) https://developer.android.com/training/articles/security-config.html
2) http://android-developers.blogspot.com/2016/07/changes-to-trusted-certificate.html

This script injects into the APK network security exceptions that allow third-party software like Charles Proxy/Fiddler to listen to the network requests and responses of some Android applications.

## High Level Steps
1. Split the .apk file into a nice normal folder where we can add the Charlse Certificate. `java -jar ./apktool.jar d -f -s -o ./del_me mdb-clx-master-testk8s-3.7.9-776-debug.apk`
1. Put the charles certificate (aka cv_ca) in /res/raw or a similar location.
1. Trust the Charles Certificate via the `network_security_config.xml` file. See the [example network_security_config.xml](./network_security_config.xml)
1. Register the `network_security_config.xml` file in the AndroidManifest.xml file. May already have been done.
1. Put the app back together `java -jar ./apktool.jar b -o ./hand-modified-app.apk tmp_app_folder`
1. Repackage the app with the `jarsigner.jarsigner -verbose -keystore ~/.android/debug.keystore -storepass android -keypass android ./hand-modified-mdb.apk androiddebugkey`. You may have to run this if you get an error.`keytool -genkey -v -keystore ~/.android/debug.keystore -storepass android -alias androiddebugkey -keypass android -dname "CN=Android Debug,O=Android,C=US"`

There may also be some "enable debug" mode stuff in here too.  Its worth noting here that you MUST use a Debug build of your app. See some [differences between Debug vs Production](https://stackoverflow.com/questions/38864358/difference-between-debug-and-release-apks) and the [Android Stuido docs](https://developer.android.com/studio/publish/preparing).

## Getting Started

Download the `addSecurityException.sh` script and the `network_security_config.xml`  file and place them in the same directory.

Note: the XML file may be optional, some apps already have a `network_security_config.xml` file.

### Prerequisites
- Certificate that will be re-encrypting traffic.  Be sure to get the Charles Root Certificate and not the Root Cert + Key. For Charles see [here](https://www.charlesproxy.com/documentation/using-charles/ssl-certificates/).

- `~/.android/debug.keystore`. If you don't have it you can try generating it https://stackoverflow.com/questions/8576732/there-is-no-debug-keystore-in-android-folder. 

```
keytool -genkey -v -keystore ~/.android/debug.keystore -storepass android -alias androiddebugkey -keypass android -dname "CN=Android Debug,O=Android,C=US"
```

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
