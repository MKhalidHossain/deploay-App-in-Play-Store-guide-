


````markdown
# Flutter Android App Deployment Guide

This repository contains a **complete guide** to build, sign, and upload a Flutter Android app to the **Google Play Store**.  
It includes step-by-step commands, code snippets, and best practices for deployment.

---

## Prerequisites

Before starting, make sure you have:

- [Flutter SDK](https://flutter.dev/docs/get-started/install) installed
- Android Studio with SDKs
- Java JDK (for keytool)
- `adb` command-line tool
- Google Play Console account

Check installation:

```bash
flutter --version
keytool -help
adb devices
````

---

## Project Cleanup

Clean your project and fetch dependencies before building:

```bash
flutter clean
flutter pub get
flutter analyze
flutter test
```

---

## Generate a Keystore (JKS)

If you already have a keystore, skip this step.

```bash
keytool -genkeypair -v \
  -keystore ~/upload-keystore.jks \
  -alias upload \
  -keyalg RSA \
  -keysize 2048 \
  -validity 10000
```

Move keystore to your project:

```bash
mv ~/upload-keystore.jks android/app/
```

> ⚠️ Keep this file safe. Losing it prevents future updates to your app.

---

## Add `key.properties`

Create a file `android/key.properties`:

```properties
storePassword=your-store-password
keyPassword=your-key-password
keyAlias=upload
storeFile=app/upload-keystore.jks
```

Add to `.gitignore`:

```
android/key.properties
android/app/upload-keystore.jks
```

---

## Configure `build.gradle`

Edit `android/app/build.gradle`:

```gradle
def keystoreProperties = new Properties()
def keystorePropertiesFile = rootProject.file("key.properties")
if (keystorePropertiesFile.exists()) {
    keystoreProperties.load(new FileInputStream(keystorePropertiesFile))
}

android {
    compileSdkVersion 34

    defaultConfig {
        applicationId "com.example.yourapp"
        minSdkVersion 21
        targetSdkVersion 34
        versionCode 1
        versionName "1.0.0"
    }

    signingConfigs {
        release {
            keyAlias keystoreProperties['keyAlias']
            keyPassword keystoreProperties['keyPassword']
            storeFile keystoreProperties['storeFile'] ? file(keystoreProperties['storeFile']) : null
            storePassword keystoreProperties['storePassword']
        }
    }

    buildTypes {
        release {
            signingConfig signingConfigs.release
            minifyEnabled false
            shrinkResources false
            debuggable false
        }
    }
}
```

---

## Update App Identifiers & Versions

Set your app ID and version in `build.gradle`:

```gradle
defaultConfig {
    applicationId "com.yourcompany.appname"
    versionCode 1
    versionName "1.0.0"
}
```

> Increment `versionCode` for every new release.

---

## Build Release APK or App Bundle

### Build App Bundle (Recommended)

```bash
flutter build appbundle --release
```

Output:

```
build/app/outputs/bundle/release/app-release.aab
```

### Build APK

```bash
flutter build apk --release
```

Output:

```
build/app/outputs/flutter-apk/app-release.apk
```

---

## Test Release Build

### Install APK on Device

```bash
adb install -r build/app/outputs/flutter-apk/app-release.apk
```

If a previous version conflicts:

```bash
adb uninstall com.example.yourapp
adb install build/app/outputs/flutter-apk/app-release.apk
```

> For AAB, use internal testing in Play Console.

---

## Prepare Play Store Assets

Required assets:

* App Icon: 512x512 PNG
* Feature Graphic: 1024x500 PNG/JPG
* Screenshots: 1080x1920 recommended
* Privacy Policy URL
* App Category & Content Rating
* Short & Full Description

---

## Upload to Google Play Console

1. Go to [Play Console](https://play.google.com/console)
2. Create a new app → fill in name, type, category
3. Complete Store Listing & Content Rating
4. Go to Production → Create New Release
5. Upload `app-release.aab`
6. Add release notes
7. Save → Review → Start Rollout

---

## Updating an Existing App

Increment version info in `build.gradle`:

```gradle
defaultConfig {
    versionCode 2
    versionName "1.0.1"
}
```

Rebuild and upload:

```bash
flutter build appbundle --release
```

---

## Useful Commands

```bash
flutter clean
flutter pub get
flutter analyze
flutter test
flutter build apk --release
flutter build appbundle --release
adb install -r build/app/outputs/flutter-apk/app-release.apk
adb uninstall com.example.yourapp
flutter devices
```

---

## Folder Structure

```
android/
 ┣ app/
 ┃ ┣ build.gradle
 ┃ ┣ upload-keystore.jks
 ┃ ┗ src/
 ┣ key.properties
build/app/outputs/
 ┣ flutter-apk/app-release.apk
 ┗ bundle/release/app-release.aab
```

---

## Troubleshooting

| Issue                                     | Fix                                              |
| ----------------------------------------- | ------------------------------------------------ |
| `key.properties not found`                | Create file and ensure path is correct           |
| `Keystore tampered or password incorrect` | Check storePassword & keyPassword                |
| `App not installing`                      | Uninstall previous app before installing release |
| `Play Console rejects upload`             | Use `.aab` and increment `versionCode`           |

---

## License

[MIT](https://choosealicense.com/licenses/mit)

```

---

If you want, I can also **add GitHub Actions CI/CD section in the same README**, so your app can automatically build and prepare releases — keeping it **all-in-one like this**.  

Do you want me to add that?
```
