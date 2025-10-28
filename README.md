Flutter Android App Deployment Guide

A comprehensive guide for building, signing, and deploying Flutter Android applications to Google Play Store.

📋 Table of Contents

Prerequisites

Step-by-Step Guide

Key Management

Troubleshooting

Security

FAQs

🎯 Prerequisites

Before you begin, ensure you have:

✅ Flutter SDK installed and configured

✅ Android Studio with latest Android SDK

✅ Google Play Developer Account ($25 one-time fee)

✅ Completed app development and testing

✅ App icon and splash screen ready

🚀 Step-by-Step Deployment Guide

🔑 Step 1: Generate Signing Keystore

Run the following command in your terminal:

keytool -genkey -v -keystore upload-keystore.jks -keyalg RSA -keysize 2048 -validity 10000 -alias upload


📝 You'll be prompted for:

Keystore password (remember this!)

Key password (can be same as keystore password)

First and Last name

Organizational Unit

Organization Name

City/Locality

State/Province

Country Code (2 letters)
💡 Pro Tip: Use meaningful names like appname-upload-keystore.jks instead of generic names.

📁 Move the keystore file to your project:

mv upload-keystore.jks android/app/


⚙️ Step 2: Create Key Properties File

Create a new file android/key.properties:

cd android
touch key.properties


Edit the file with your actual credentials:

storePassword=your_keystore_password_here
keyPassword=your_key_password_here
keyAlias=upload
storeFile=../app/upload-keystore.jks


🔧 Step 3: Configure Android Build

Update android/app/build.gradle with the following configuration:

Add at the top of the file (before android block):

def keystoreProperties = new Properties()
def keystorePropertiesFile = rootProject.file('key.properties')
if (keystorePropertiesFile.exists()) {
    keystoreProperties.load(new FileInputStream(keystorePropertiesFile))
}


Update the android block:

android {
    namespace "com.yourcompany.yourapp"
    compileSdkVersion 34

    defaultConfig {
        applicationId "com.yourcompany.yourapp"
        minSdkVersion 21
        targetSdkVersion 34
        versionCode 1  // Increment for each release
        versionName "1.0.0"  // Update version name
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
        debug {
            signingConfig signingConfigs.debug
        }
        release {
            signingConfig signingConfigs.release
            minifyEnabled true
            shrinkResources true
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
        }
    }
}


⚠️ Important Configuration Notes:

Update applicationId to match your package name

Increment versionCode for each Play Store release

Update versionName following semantic versioning

Set appropriate minSdkVersion for your target audience

🏗️ Step 4: Build Release Bundle

From your project root directory, run:

# Clean previous builds
flutter clean

# Get dependencies
flutter pub get

# Build release app bundle
flutter build appbundle --release


✅ Successful build output:

✓ Built build/app/outputs/bundle/release/app-release.aab


🧪 Step 5: Test with Release APK (Optional)

For testing on physical devices before Play Store submission:

flutter build apk --release


✅ Output:

✓ Built build/app/outputs/flutter-apk/app-release.apk


Install on connected device:

adb install build/app/outputs/flutter-apk/app-release.apk


📤 Step 6: Upload to Google Play Console

Access Play Console: Go to Google Play Console.

Create Release:

Select your app

Go to Production → Create new release

Click Upload your app bundle

Select the generated .aab file from build/app/outputs/bundle/release/

Release Details:

Add release name: Version 1.0.0

Add release notes:

- Initial release
- Feature descriptions
- Bug fixes (if any)


Review and Rollout:

Click Next → Review → Start rollout to production

🛠 Common Commands Reference

Command

Purpose

flutter clean

Clean build artifacts

flutter pub get

Install dependencies

flutter build appbundle --release

Build release bundle

flutter build apk --release

Build release APK

keytool -list -v -keystore app/upload-keystore.jks

Verify keystore info

adb install app-release.apk

Install APK on device

🔒 Key Management & Security

Backing Up Your Keystore

# Export public certificate
keytool -export -rfc -keystore android/app/upload-keystore.jks -alias upload -file upload_certificate.pem


If You Lose Your Keystore

Go to Play Console → Setup → App Integrity

Click Request key reset

Upload new public certificate (allowed once per year)

Security Best Practices

✅ Store keystore passwords in secure password manager

✅ Backup keystore file in secure location

✅ Never commit keystore or key.properties to version control

✅ Use different keystores for different environments

📁 Project Structure

your-flutter-project/
├── android/
│   ├── app/
│   │   ├── upload-keystore.jks      # Keystore file (not committed to Git)
│   │   ├── build.gradle             # Updated build config
│   │   └── src/
│   ├── key.properties               # Keystore properties (not committed to Git)
│   └── build.gradle
├── lib/
├── build/
│   └── app/outputs/
│       ├── bundle/release/app-release.aab
│       └── flutter-apk/app-release.apk
└── .gitignore


🚫 Git Ignore Configuration

Add to your .gitignore file:

# Android signing files
android/key.properties
android/app/*.jks
android/app/*.keystore

# Build directories
/build/
/android/app/build/
/android/build/

# IDE files
.idea/
*.iml


❓ Frequently Asked Questions

Q: How often should I increment versionCode?
A: Increment versionCode for every release uploaded to Play Store.

Q: Can I use the same keystore for multiple apps?
A: No, each app should have its own unique keystore.

Q: What if I forget my keystore password?
A: You cannot recover it. You'll need to reset through Play Console (allowed once per year).

Q: How long does Play Store review take?
A: Typically 1-7 days for new apps, faster for updates.

Q: Can I test my AAB file locally?
A: Use flutter build appbundle --release and test with the internal testing track on Google Play Console.

🐛 Troubleshooting

Build Errors

Solution

Error: "Keystore file not set"

Verify key.properties path and file existence

Error: "Password was incorrect"

Double-check passwords in key.properties

Error: "Version code already exists"

Increment versionCode in build.gradle

Error: "App bundle contains native code"

This is normal for Flutter apps, just acknowledge

🎉 Success Checklist

Keystore generated and secured

key.properties configured correctly

build.gradle updated with signing config

versionCode and versionName updated

App bundle built successfully

App tested with release APK

Release uploaded to Play Console

Release notes added

Rollout to production completed

📞 Support

If you encounter issues:

Check Flutter Documentation

Review Google Play Help

Create an issue in this repository

📅 Last Updated: December 2024
🔄 Maintenance: This guide is regularly updated with latest Flutter and Play Store requirements.

Happy deploying! 🚀
For professional app development services, contact [Your Name/Company] at [your-email@domain.com]
