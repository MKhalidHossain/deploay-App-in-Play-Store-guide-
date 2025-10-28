Flutter Android App Bundle (AAB) Build & Play Store Deployment Guide
A comprehensive guide for building signed release bundles and deploying Flutter apps to Google Play Store.

📋 Prerequisites
Flutter SDK installed and configured

Android Studio with latest Android SDK

Google Play Developer Account ($25 one-time fee)

Complete your app development and testing

🚀 Step-by-Step Guide
🔑 Step 1 — Generate Keystore
Run the following command in your terminal:

bash
keytool -genkey -v -keystore my-key.jks -keyalg RSA -keysize 2048 -validity 10000 -alias my-key-alias
💡 Important Notes:

Replace my-key.jks and my-key-alias with your own names

You'll be asked for:

Keystore password

First and Last name

Organizational unit & organization

City, State, and Country codes

When finished, your my-key.jks file will be created.

📁 Move the keystore file:
Place it in your project at:

text
/android/app/my-key.jks
⚙️ Step 2 — Create key.properties
Navigate to the android directory and create the properties file:

bash
cd android
touch key.properties
Open key.properties and add the following configuration:

properties
storePassword=your_keystore_password
keyPassword=your_key_password
keyAlias=your_key_alias
storeFile=../app/my-key.jks
Replace the placeholder values with your actual credentials.

⚙️ Step 3 — Configure build.gradle
Open android/app/build.gradle and add the following code:

gradle
def keystoreProperties = new Properties()
def keystorePropertiesFile = rootProject.file('key.properties')
if (keystorePropertiesFile.exists()) {
    keystoreProperties.load(new FileInputStream(keystorePropertiesFile))
}

android {
    namespace "com.yourcompany.appname"
    compileSdkVersion 34

    defaultConfig {
        applicationId "com.yourcompany.appname"
        minSdkVersion 21
        targetSdkVersion 34
        versionCode 2
        versionName "1.1.0"
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
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
}
⚠️ Important:

Update namespace, applicationId, and package name to match your app

Increment versionCode for each new release

Update versionName following semantic versioning

🧱 Step 4 — Build Signed Release Bundle
From your project root directory, run:

bash
flutter clean
flutter pub get
flutter build appbundle --release
✅ Output file:

text
build/app/outputs/bundle/release/app-release.aab
🧪 Step 5 — (Optional) Build APK for Testing
For testing on physical devices before Play Store upload:

bash
flutter build apk --release
✅ Output:

text
build/app/outputs/flutter-apk/app-release.apk
🧭 Step 6 — Upload to Google Play Console
Go to Google Play Console

Select your app → Release → Production → Create new release

Click Upload your app bundle and select the generated .aab file

Add release notes:

text
- Bug fixes
- Performance improvements
- New features (if any)
Click Next → Review → Start rollout to production ✅

🔒 Key Management
If You Lost or Need to Replace Your JKS
Request a key reset from Play Console:

Go to Setup → App Integrity → App Signing

Click Request key reset

Upload your new .pem file (allowed once per year)

Export .pem public key from your JKS:

bash
keytool -export -rfc -keystore my-key.jks -alias my-key-alias -file upload_certificate.pem
🛠 Common Terminal Commands
Purpose	Command
🧹 Clean project	flutter clean
📦 Get dependencies	flutter pub get
🏗️ Build release AAB	flutter build appbundle --release
🔍 Build release APK	flutter build apk --release
🔑 View keystore info	keytool -list -v -keystore my-key.jks
⚡ Install APK on device	adb install build/app/outputs/flutter-apk/app-release.apk
📁 Project Structure
text
android/
 ├── app/
 │   ├── build.gradle
 │   ├── my-key.jks
 │   └── src/
 ├── key.properties
 └── build.gradle
🔒 Security - Add to .gitignore
Never commit your signing credentials to version control!

Add these lines to your .gitignore:

gitignore
# Flutter build
/build/

# Android signing files
android/key.properties
android/app/*.jks
android/app/*.keystore

# iOS signing (if applicable)
ios/Runner/GoogleService-Info.plist
🏁 Completion
🎉 Congratulations! You have successfully built, signed, and uploaded your Flutter app to the Play Store.

🔐 Important Reminders:

Keep your JKS file secure and backed up

Store passwords in a secure password manager

Never share your signing credentials

Keep Play Console credentials secure

🤖 Optional: GitHub Actions CI/CD
Automate your build and deployment process with GitHub Actions. Create .github/workflows/deploy.yml:

yaml
name: Deploy to Play Store
on:
  push:
    tags:
      - 'v*'

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-java@v3
        with:
          java-version: '11'
          distribution: 'temurin'
      - uses: subosito/flutter-action@v2
        with:
          flutter-version: '3.19.0'
      - run: flutter pub get
      - run: flutter test
      - run: flutter build appbundle --release
        env:
          KEY_STORE: ${{ secrets.KEY_STORE }}
          KEY_STORE_PASSWORD: ${{ secrets.KEY_STORE_PASSWORD }}
          KEY_ALIAS: ${{ secrets.KEY_ALIAS }}
          KEY_ALIAS_PASSWORD: ${{ secrets.KEY_ALIAS_PASSWORD }}
      - uses: r0adkll/upload-google-play@v1
        with:
          serviceAccountJsonPlainText: ${{ secrets.GCP_SERVICE_ACCOUNT }}
          packageName: com.yourcompany.appname
          releaseFiles: build/app/outputs/bundle/release/app-release.aab
          track: production
Author: [Your Name]
Last Updated: 2024-12-19

Need help? Check the Flutter documentation or create an issue.

This response is AI-generated, for reference only.
