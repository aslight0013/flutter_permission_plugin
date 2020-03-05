# Flutter Permission handler Plugin

[![pub package](https://img.shields.io/pub/v/permission_handler.svg)](https://pub.dartlang.org/packages/permission_handler) [![Build Status](https://app.bitrise.io/app/fa4f5d4bf452bcfb/status.svg?token=HorGpL_AOw2llYz39CjmdQ&branch=master)](https://app.bitrise.io/app/fa4f5d4bf452bcfb) [![style: effective dart](https://img.shields.io/badge/style-effective_dart-40c4ff.svg)](https://github.com/tenhobi/effective_dart)

A permissions plugin for Flutter. This plugin provides a cross-platform (iOS, Android) API to request and check permissions.

## Features

* Check if a permission is granted.
* Request permission for a specific feature.
* Open app settings so the user can enable a permission.
* Show a rationale for requesting permission (Android).

## Usage

To use this plugin, add `permission_handler` as a [dependency in your pubspec.yaml file](https://flutter.io/platform-plugins/). For example:

```yaml
dependencies:
  permission_handler: '^4.3.0'
```

> **NOTE:** As of version 3.1.0 the permission_handler plugin switched to the AndroidX version of the Android Support Libraries. This means you need to make sure your Android project is also upgraded to support AndroidX. Detailed instructions can be found [here](https://flutter.dev/docs/development/packages-and-plugins/androidx-compatibility). 
>
>The TL;DR version is:
>
>1. Add the following to your "gradle.properties" file:
>
>```
>android.useAndroidX=true
>android.enableJetifier=true
>```
>2. Make sure you set the `compileSdkVersion` in your "android/app/build.gradle" file to 28:
>
>```
>android {
>  compileSdkVersion 28
>
>  ...
>}
>```
>3. Make sure you replace all the `android.` dependencies to their AndroidX counterparts (a full list can be found here: https://developer.android.com/jetpack/androidx/migrate).

### Android and iOS specific permissions

For this plugin to work you will have to add permission configuration to your `AndroidManifest.xml` (Android) and `Info.plist` (iOS) files. This will tell the platform which hardware or software features your app needs. Complete lists of these permission options can be found in our example app here:

- [AndroidManifest.xml](https://github.com/Baseflow/flutter-permission-handler/blob/develop/example/android/app/src/main/AndroidManifest.xml) (note that there is a debug, main and profile version which are used depending on how you start your App. In general it is sufficient to add permissions only to the `main` version);
- [Info.plist](https://github.com/Baseflow/flutter-permission-handler/blob/develop/example/ios/Runner/Info.plist)

> IMPORTANT: ~~On iOS you will have to include all permission options when you want to submit your App.~~ This is because the `permission_handler` plugin touches all different SDKs and because the static code analyser (run by Apple upon App submission) detects this and will assert if it cannot find a matching permission option in the `Info.plist`. More information about this can be found [here](https://github.com/BaseflowIT/flutter-permission-handler/issues/26).

On iOS, the permission_handler plugin use [macros](https://github.com/BaseflowIT/flutter-permission-handler/blob/develop/ios/Classes/PermissionHandlerEnums.h) to control whether a permission is supported.

By default, all the permissions listed [here](https://github.com/Baseflow/flutter-permission-handler#list-of-available-permissions) are supported.

You can remove permissions you don't use by:

> 1. Add the following to your `Podfile` file:
>
>    ```ruby
>    post_install do |installer|
>      installer.pods_project.targets.each do |target|
>        target.build_configurations.each do |config|
>          ... # Here are some configurations automatically generated by flutter
>    
>          # You can remove unused permissions here
>          # for more infomation: https://github.com/BaseflowIT/flutter-permission-handler/blob/develop/ios/Classes/PermissionHandlerEnums.h
>          # e.g. when you don't need camera permission, just add 'PERMISSION_CAMERA=0'
>          config.build_settings['GCC_PREPROCESSOR_DEFINITIONS'] ||= [
>            '$(inherited)',
>   
>            ## dart: PermissionGroup.calendar
>            # 'PERMISSION_EVENTS=0',
>   
>            ## dart: PermissionGroup.reminders
>            # 'PERMISSION_REMINDERS=0',
>   
>            ## dart: PermissionGroup.contacts
>            # 'PERMISSION_CONTACTS=0',
>   
>            ## dart: PermissionGroup.camera
>            # 'PERMISSION_CAMERA=0',
>   
>            ## dart: PermissionGroup.microphone
>            # 'PERMISSION_MICROPHONE=0',
>   
>            ## dart: PermissionGroup.speech
>            # 'PERMISSION_SPEECH_RECOGNIZER=0',
>   
>            ## dart: PermissionGroup.photos
>            # 'PERMISSION_PHOTOS=0',
>   
>            ## dart: [PermissionGroup.location, PermissionGroup.locationAlways, PermissionGroup.locationWhenInUse]
>            # 'PERMISSION_LOCATION=0',
>           
>            ## dart: PermissionGroup.notification
>            # 'PERMISSION_NOTIFICATIONS=0',
>   
>            ## dart: PermissionGroup.mediaLibrary
>            # 'PERMISSION_MEDIA_LIBRARY=0',
>   
>            ## dart: PermissionGroup.sensors
>            # 'PERMISSION_SENSORS=0'
>          ]
>   
>        end
>      end
>    end
>    ```
>
> 2. Delete the corresponding permission description in `Info.plist`
>
>    e.g. when you don't need camera permission, just delete 'NSCameraUsageDescription'
>
>    The following lists the relationship between `Permission` and `The key of Info.plist`:
>
>    | Permission  | Info.plist  | Macro |
>    |---|---|---|
>    | PermissionGroup.calendar  |  NSCalendarsUsageDescription | PERMISSION_EVENTS |
>    | PermissionGroup.reminders  |  NSRemindersUsageDescription | PERMISSION_REMINDERS |
>    | PermissionGroup.contacts  |  NSContactsUsageDescription | PERMISSION_CONTACTS |
>    | PermissionGroup.camera  |  NSCameraUsageDescription | PERMISSION_CAMERA |
>    | PermissionGroup.microphone  |  NSMicrophoneUsageDescription | PERMISSION_MICROPHONE |
>    | PermissionGroup.speech  |  NSSpeechRecognitionUsageDescription | PERMISSION_SPEECH_RECOGNIZER |
>    | PermissionGroup.photos  |  NSPhotoLibraryUsageDescription | PERMISSION_PHOTOS |
>    | PermissionGroup.location, PermissionGroup.locationAlways, PermissionGroup.locationWhenInUse  |  NSLocationUsageDescription, NSLocationAlwaysAndWhenInUseUsageDescription, NSLocationWhenInUseUsageDescription | PERMISSION_LOCATION |
>    | PermissionGroup.notification  |  PermissionGroupNotification | PERMISSION_NOTIFICATIONS |
>    | PermissionGroup.mediaLibrary  |  NSAppleMusicUsageDescription, kTCCServiceMediaLibrary | PERMISSION_MEDIA_LIBRARY |
>    | PermissionGroup.sensors  |  NSMotionUsageDescription | PERMISSION_SENSORS |
>
> 3. Clean & Rebuild

## API

### Requesting permission

```dart
import 'package:permission_handler/permission_handler.dart';

Map<PermissionGroup, PermissionStatus> permissions = await PermissionHandler().requestPermissions([PermissionGroup.contacts]);
```

### Checking permission

```dart
import 'package:permission_handler/permission_handler.dart';

PermissionStatus permission = await PermissionHandler().checkPermissionStatus(PermissionGroup.contacts);
```

### Checking service status

```dart
import 'package:permission_handler/permission_handler.dart';

ServiceStatus serviceStatus = await PermissionHandler().checkServiceStatus(PermissionGroup.location);
```

Checking the service status only makes sense for the `PermissionGroup.location` on Android and the `PermissionGroup.location`, `PermissionGroup.locationWhenInUse`, `PermissionGroup.locationAlways` or `PermissionGroup.sensors` on iOS. All other permission groups are not backed by a separate service and will always return `ServiceStatus.notApplicable`.

### Open app settings

```dart
import 'package:permission_handler/permission_handler.dart';

bool isOpened = await PermissionHandler().openAppSettings();
```

### Show a rationale for requesting permission (Android only)

```dart
import 'package:permission_handler/permission_handler.dart';

bool isShown = await PermissionHandler().shouldShowRequestPermissionRationale(PermissionGroup.contacts);
```

This will always return `false` on iOS.

### List of available permissions

Defines the permission groups for which permissions can be checked or requested.

```dart
enum PermissionGroup {
  /// The unknown permission only used for return type, never requested
  unknown,

  /// Android: Calendar
  /// iOS: Calendar (Events)
  calendar,

  /// Android: Camera
  /// iOS: Photos (Camera Roll and Camera)
  camera,

  /// Android: Contacts
  /// iOS: AddressBook
  contacts,

  /// Android: Fine and Coarse Location
  /// iOS: CoreLocation (Always and WhenInUse)
  location,

  /// Android: Microphone
  /// iOS: Microphone
  microphone,

  /// Android: Phone
  /// iOS: Nothing
  phone,

  /// Android: Nothing
  /// iOS: Photos
  photos,

  /// Android: Nothing
  /// iOS: Reminders
  reminders,

  /// Android: Body Sensors
  /// iOS: CoreMotion
  sensors,

  /// Android: Sms
  /// iOS: Nothing
  sms,

  /// Android: External Storage
  /// iOS: Nothing
  storage,

  /// Android: Microphone
  /// iOS: Speech
  speech,

  /// Android: Fine and Coarse Location
  /// iOS: CoreLocation - Always
  locationAlways,

  /// Android: Fine and Coarse Location
  /// iOS: CoreLocation - WhenInUse
  locationWhenInUse,

  /// Android: None
  /// iOS: MPMediaLibrary
  mediaLibrary,

  /// Android: Check notification enable
  /// iOS: Check and request notification permission
  notification,

  /// Android Q: Check and request permissions to read from the media location (ACCESS_MEDIA_LOCATION)
  /// Android pre-Q: Nothing
  /// iOS: Nothing
  accessMediaLocation,

  /// Android Q: Check and request permissions to access the Activity Recognition API
  /// Android pre-Q: Nothing 
  /// iOS: Nothing (should implement access to CMMotionActivity, see issue #219)
  activityRecognition,  
}
```

### Status of the permission

Defines the state of a permission group

```dart
enum PermissionStatus {
  /// Permission to access the requested feature is denied by the user.
  denied,

  /// Permission to access the requested feature is granted by the user.
  granted,

  /// The user granted restricted access to the requested feature (only on iOS).
  restricted,

  /// Permission is in an unknown state
  unknown
  
  /// Permission to access the requested feature is denied by the user and never show selected (only on Android).
  neverAskAgain
}
```

### Overview of possible service statuses

Defines the state of the backing service for the supplied permission group

```dart
/// Defines the state of a service related to the permission group
enum ServiceStatus {
  /// The unknown service status indicates the state of the service could not be determined.
  unknown,

  /// There is no service for the supplied permission group.
  notApplicable,

  /// The service for the supplied permission group is disabled.
  disabled,

  /// The service for the supplied permission group is enabled.
  enabled
}
```

## Issues

Please file any issues, bugs or feature request as an issue on our [GitHub](https://github.com/Baseflow/flutter-permission-handler/issues) page.

## Want to contribute

If you would like to contribute to the plugin (e.g. by improving the documentation, solving a bug or adding a cool new feature), please carefully review our [contribution guide](CONTRIBUTING.md) and send us your [pull request](https://github.com/Baseflow/flutter-permission-handler/pulls).

## Author

This Permission handler plugin for Flutter is developed by [Baseflow](https://baseflow.com). You can contact us at <hello@baseflow.com>
