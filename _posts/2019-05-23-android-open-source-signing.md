Simple config to hide signing keystore but allowing to build release variant for contributors. Useful for open source Apps and pet projects.


**TL;DR repository**: https://github.com/IvanShafran/android-open-source-app-signing

By default Android app has debug build in which APK is signed automatically with debug keystore. But for a release build, you have to sign with a manually created keystore.

If you develop an open source app, you can’t keep a keystore together with code due to security reasons. You can easily exclude it in the .gitignore file, but in this case, your contributors won’t build an app in a release variant. It complicates open source development because some bugs can only be discovered in a release build. Moreover, you can’t run UI tests for release builds directly from a repository. Let’s solve the problem!

# First step. Create two keystores

Create one keystore for production build with strong passwords. And create another for contributors with any password. Place them both in the main app module folder. For default created project folder is “app.”

[How to generate keystore](https://developer.android.com/studio/publish/app-signing#generate-key)


![Two files: production.jks and contributor.jks](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/c5apwe9su3099os9qp89.png)

**At this point, add `production.jks` to `.gitignore` .**

# Second step. Create two property files

Create two properties files in the same folder as described below.

```
# Sign key alias for contributors.jks
releaseSignKeyAlias=key
# Sign key password for contributors.jks
releaseSignKeyPassword=password
# Path to contributors.jks
releaseStoreFilePath=./contributors.jks
# Keystore password for contributors.jks
releaseStorePassword=password
```

Also, create production.properties with production key and passwords.

![Two files: contributor.properties and production.properties](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/7tfc9ocruh2h4p05x32k.png)

**At this point, add `production.properties` to `.gitignore` .**

# Third step. Add script

Add script listed below to the same folder and apply it in build.gradle. Script checks if production properties file exists in the folder and if it exists script uses production keystore otherwise contributors. That’s all magic :)

**Don’t forget to add production.jks and production.properties to .gitignore .**

settings.gradle:
```groovy
def propertiesFilename = "production.properties"
if (!project.file(propertiesFilename).exists()) {
    propertiesFilename = "contributors.properties"
}

def signingProperties = new Properties()
signingProperties.load(new FileInputStream(file(propertiesFilename)))

android {
    signingConfigs {
        release {
            keyAlias signingProperties.releaseSignKeyAlias
            keyPassword signingProperties.releaseSignKeyPassword
            storeFile file(signingProperties.releaseStoreFilePath)
            storePassword signingProperties.releaseStorePassword
        }
    }
}
```

build.gradle:
```groovy
...
apply from: 'signing.gradle'
...
android {
  ...
  buildTypes {
        release {
            signingConfig signingConfigs.release
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
}
```


![Two files highlighted: build.gradle and settings.gradle](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/3a0b7dg4j1xd86e0udq4.png)

**Full sample here:** https://github.com/IvanShafran/android-open-source-app-signing