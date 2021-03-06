[![pub package](https://img.shields.io/pub/v/permission_handler.svg)](https://pub.dartlang.org/packages/permission_handler) [![Build status](https://github.com/Baseflow/flutter-permission-handler/actions/workflows/app_facing_package.yaml/badge.svg?branch=master)](https://github.com/Baseflow/flutter-permission-handler/actions/workflows/app_facing_package.yaml) [![style: effective dart](https://img.shields.io/badge/style-effective_dart-40c4ff.svg)](https://github.com/tenhobi/effective_dart)

On most operating systems, permissions aren't just granted to apps at install time.
Rather, developers have to ask the user for permissions while the app is running.

This plugin provides a cross-platform (iOS, Android) API to request permissions and check their status.
You can also open the device's app settings so users can grant a permission.  
On Android, you can show a rationale for requesting a permission.

## Setup

While the permissions are being requested during runtime, you'll still need to tell the OS which permissions your app might potentially use. That requires adding permission configuration to Android- and iOS-specific files.

<details>
<summary>Android</summary>
  
**Upgrade pre 1.12 Android projects**
  
Since version 4.4.0 this plugin is implemented using the Flutter 1.12 Android plugin APIs. Unfortunately this means App developers also need to migrate their Apps to support the new Android infrastructure. You can do so by following the [Upgrading pre 1.12 Android projects](https://github.com/flutter/flutter/wiki/Upgrading-pre-1.12-Android-projects) migration guide. Failing to do so might result in unexpected behaviour. Most common known error is the permission_handler not returning after calling the `.request()` method on a permission. 

**AndroidX**

As of version 3.1.0 the <kbd>permission_handler</kbd> plugin switched to the AndroidX version of the Android Support Libraries. This means you need to make sure your Android project is also upgraded to support AndroidX. Detailed instructions can be found [here](https://flutter.dev/docs/development/packages-and-plugins/androidx-compatibility).

The TL;DR version is:
1. Add the following to your "gradle.properties" file:
```
android.useAndroidX=true
android.enableJetifier=true
```
2. Make sure you set the `compileSdkVersion` in your "android/app/build.gradle" file to 28:
```
android {
  compileSdkVersion 28
  ...
}
```
3. Make sure you replace all the `android.` dependencies to their AndroidX counterparts (a full list can be found here: https://developer.android.com/jetpack/androidx/migrate).

Add permissions to your `AndroidManifest.xml` file.
There's a `debug`, `main` and `profile` version which are chosen depending on how you start your app.
In general, it's sufficient to add permission only to the `main` version.
[Here](https://github.com/Baseflow/flutter-permission-handler/blob/develop/permission_handler/example/android/app/src/main/AndroidManifest.xml)'s an example `AndroidManifest.xml` with a complete list of all possible permissions.

</details>

<details>
<summary>iOS</summary>

Add permission to your `Info.plist` file.
[Here](https://github.com/Baseflow/flutter-permission-handler/blob/develop/permission_handler/example/ios/Runner/Info.plist)'s an example `Info.plist` with a complete list of all possible permissions.

> IMPORTANT: ~~You will have to include all permission options when you want to submit your App.~~ This is because the `permission_handler` plugin touches all different SDKs and because the static code analyser (run by Apple upon App submission) detects this and will assert if it cannot find a matching permission option in the `Info.plist`. More information about this can be found [here](https://github.com/BaseflowIT/flutter-permission-handler/issues/26).

The <kbd>permission_handler</kbd> plugin use [macros](https://github.com/BaseflowIT/flutter-permission-handler/blob/develop/permission_handler/ios/Classes/PermissionHandlerEnums.h) to control whether a permission is supported.

You can remove permissions you don't use:

1. Add the following to your `Podfile` file:
   ```ruby
   post_install do |installer|
     installer.pods_project.targets.each do |target|
       target.build_configurations.each do |config|
         ... # Here are some configurations automatically generated by flutter
   
         # You can remove unused permissions here
         # for more infomation: https://github.com/BaseflowIT/flutter-permission-handler/blob/develop/permission_handler/ios/Classes/PermissionHandlerEnums.h
         # e.g. when you don't need camera permission, just add 'PERMISSION_CAMERA=0'
         config.build_settings['GCC_PREPROCESSOR_DEFINITIONS'] ||= [
           '$(inherited)',
  
           ## dart: PermissionGroup.calendar
           # 'PERMISSION_EVENTS=0',
  
           ## dart: PermissionGroup.reminders
           # 'PERMISSION_REMINDERS=0',
  
           ## dart: PermissionGroup.contacts
           # 'PERMISSION_CONTACTS=0',
  
           ## dart: PermissionGroup.camera
           # 'PERMISSION_CAMERA=0',
  
           ## dart: PermissionGroup.microphone
           # 'PERMISSION_MICROPHONE=0',
  
           ## dart: PermissionGroup.speech
           # 'PERMISSION_SPEECH_RECOGNIZER=0',
  
           ## dart: PermissionGroup.photos
           # 'PERMISSION_PHOTOS=0',
  
           ## dart: [PermissionGroup.location, PermissionGroup.locationAlways, PermissionGroup.locationWhenInUse]
           # 'PERMISSION_LOCATION=0',
          
           ## dart: PermissionGroup.notification
           # 'PERMISSION_NOTIFICATIONS=0',
  
           ## dart: PermissionGroup.mediaLibrary
           # 'PERMISSION_MEDIA_LIBRARY=0',
  
           ## dart: PermissionGroup.sensors
           # 'PERMISSION_SENSORS=0'
         ]
  
       end
     end
   end
   ```
2. Remove the `#` character in front of the permission you do not want to use. For example if you don't need access to the calendar make sure the code looks like this:
   ```ruby
           ## dart: PermissionGroup.calendar
           'PERMISSION_EVENTS=0',
   ```
3. Delete the corresponding permission description in `Info.plist`
   e.g. when you don't need camera permission, just delete 'NSCameraUsageDescription'
   The following lists the relationship between `Permission` and `The key of Info.plist`:
   | Permission                                                                                  | Info.plist                                                                                                    | Macro                        |
   | ------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------- | ---------------------------- |
   | PermissionGroup.calendar                                                                    | NSCalendarsUsageDescription                                                                                   | PERMISSION_EVENTS            |
   | PermissionGroup.reminders                                                                   | NSRemindersUsageDescription                                                                                   | PERMISSION_REMINDERS         |
   | PermissionGroup.contacts                                                                    | NSContactsUsageDescription                                                                                    | PERMISSION_CONTACTS          |
   | PermissionGroup.camera                                                                      | NSCameraUsageDescription                                                                                      | PERMISSION_CAMERA            |
   | PermissionGroup.microphone                                                                  | NSMicrophoneUsageDescription                                                                                  | PERMISSION_MICROPHONE        |
   | PermissionGroup.speech                                                                      | NSSpeechRecognitionUsageDescription                                                                           | PERMISSION_SPEECH_RECOGNIZER |
   | PermissionGroup.photos                                                                      | NSPhotoLibraryUsageDescription                                                                                | PERMISSION_PHOTOS            |
   | PermissionGroup.location, PermissionGroup.locationAlways, PermissionGroup.locationWhenInUse | NSLocationUsageDescription, NSLocationAlwaysAndWhenInUseUsageDescription, NSLocationWhenInUseUsageDescription | PERMISSION_LOCATION          |
   | PermissionGroup.notification                                                                | PermissionGroupNotification                                                                                   | PERMISSION_NOTIFICATIONS     |
   | PermissionGroup.mediaLibrary                                                                | NSAppleMusicUsageDescription, kTCCServiceMediaLibrary                                                         | PERMISSION_MEDIA_LIBRARY     |
   | PermissionGroup.sensors                                                                     | NSMotionUsageDescription                                                                                      | PERMISSION_SENSORS           |
4. Clean & Rebuild

</details>



## How to use

There are a number of [`Permission`](https://pub.dev/documentation/permission_handler_platform_interface/latest/permission_handler_platform_interface/Permission-class.html#constants)s.
You can get a `Permission`'s `status`, which is either `granted`, `denied`, `restricted` or `permanentlyDenied`.

```dart
var status = await Permission.camera.status;
if (status.denied) {
  // We didn't ask for permission yet or the permission has been denied before but not permanently.
}

// You can can also directly ask the permission about its status.
if (await Permission.location.isRestricted) {
  // The OS restricts access, for example because of parental controls.
}
```

Call `request()` on a `Permission` to request it.
If it has already been granted before, nothing happens.  
`request()` returns the new status of the `Permission`.

```dart
if (await Permission.contacts.request().isGranted) {
  // Either the permission was already granted before or the user just granted it.
}

// You can request multiple permissions at once.
Map<Permission, PermissionStatus> statuses = await [
  Permission.location,
  Permission.storage,
].request();
print(statuses[Permission.location]);
```

Some permissions, for example location or acceleration sensor permissions, have an associated service, which can be `enabled` or `disabled`.

```dart
if (await Permission.locationWhenInUse.serviceStatus.isEnabled) {
  // Use location.
}
```

You can also open the app settings:

```dart
if (await Permission.speech.isPermanentlyDenied) {
  // The user opted to never again see the permission request dialog for this
  // app. The only way to change the permission's status now is to let the
  // user manually enable it in the system settings.
  openAppSettings();
}
```

On Android, you can show a rationale for using a permission:

```dart
bool isShown = await Permission.contacts.shouldShowRequestRationale;
```

## Issues

Please file any issues, bugs or feature request as an issue on our [GitHub](https://github.com/Baseflow/flutter-permission-handler/issues) page. Commercial support is available if you need help with integration with your app or services. You can contact us at [hello@baseflow.com](mailto:hello@baseflow.com).

## Want to contribute

If you would like to contribute to the plugin (e.g. by improving the documentation, solving a bug or adding a cool new feature), please carefully review our [contribution guide](./CONTRIBUTING.md) and send us your [pull request](https://github.com/Baseflow/flutter-permission-handler/pulls).

## Author

This Permission handler plugin for Flutter is developed by [Baseflow](https://baseflow.com). You can contact us at <hello@baseflow.com>
