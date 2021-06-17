# notes
taken by Max D.

This cracked open the app
java -jar ./apktool.jar d -f -s -o ./del_me mdb-clx-master-testk8s-3.7.9-776-debug.apk


## adding the Charles cert
Seems simple at first. two steps

1. create file network_security_config.xml
2. add link to AndroidManifest.xml under the application tag. like <application ... android:networkSecurityConfig="@xml/network_security_config" ... ></application>
3. `android:debuggable="true"` must be true? maybe? [source](https://developer.android.com/guide/topics/manifest/application-element#debug)


Per the android step here: https://www.charlesproxy.com/documentation/using-charles/ssl-certificates/
I see the `res/xml/network_security_config.xml` file. 
It looks good?

```
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
    <debug-overrides>
        <trust-anchors>
            <certificates src="@raw/cv_ca" />
        </trust-anchors>
    </debug-overrides>
</network-security-config>
```

However most docs, including charles, say to use src="user" which I assume means the local system user? aka certs added by the user manually on the phone? Which does not apply here as RDC phones are locked down

[Google Docs](https://developer.android.com/training/articles/security-config.html) specify an approach under Configure a Custom CA.  The actual cert is included in this example `<certificates src="@raw/my_ca"/>`. Second step 

>Add the self-signed or non-public CA certificate, in PEM or DER format, to res/raw/my_ca

That has also been completed. I see cv_ca in the res/raw directory.


## final check
all seems well?
I had to add the network_security_config.xml reference to the application tag. Maybe that was the problem?

## put humpty dumpty back together
Ran this to try and put the directory ($tmpDir in the script) back into an .apk
```
java -jar ./apktool.jar b -o ./hand-modified-app.apk blownup_app_dir
```
[Fatal Error] :23:424: Attribute "android:networkSecurityConfig" was already specified for element "application".

I removed this from the appliaction: `android:networkSecurityConfig="@xml/network_config"` because I think its sketchy. Are they doing something unusual to [pin their certs?](https://developer.android.com/training/articles/security-config.html#CertificatePinning)

## put humpty dumpty back together v2
Ok so app install failed with all the above.  I reattempted by removing the reference to the network_security_config.xml file in the application tags. I replaced it with the customer's network_config.xml file and added the cv_ca cert reference inside their xml.

App still failed to install.

## put humpty dumpty back together v3
I have enabled Image Injection and Instrumentation
it now works ty Enrique

## put humpty dumpty back together v4
So the app launches but clicking the orane button leads to nothing. No HTTP calls show up in charles (for the customers domain anyway). And the screen just sits on a big blank white page.

This step may have been missing? doing this is the terminal step in the script. Trying again.
jarsigner -verbose -keystore ~/.android/debug.keystore -storepass android -keypass android ./hand-modified-mdb.apk androiddebugkey
