# trust_location

A Flutter plugin for detecting the mock location of the Android device.

## Installation

Add `trust_location` as a [dependency in your pubspec.yaml file](https://flutter.dev/docs/development/packages-and-plugins/using-packages)

## Usage

```dart
import 'package:trust_location/trust_location.dart';
...
String location = await TrustLocation.getLocation;
bool isMockLocation = await TrustLocation.isMockLocation;
...
```

> **NOTE:** The location_permissions plugin uses the AndroidX version of the Android Support Libraries. This means you need to make sure your Android project is also upgraded to support AndroidX. Detailed instructions can be found [here](https://flutter.dev/docs/development/packages-and-plugins/androidx-compatibility).
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

## Permissions

### Android

Add either the `ACCESS_COARSE_LOCATION` or the `ACCESS_FINE_LOCATION` permission to your Android Manifest. (located under android/app/src/main)

``` xml
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
```

## Example

```dart
import 'package:flutter/material.dart';
import 'dart:async';

import 'package:flutter/services.dart';
import 'package:trust_location/trust_location.dart';

import 'package:location_permissions/location_permissions.dart';

void main() => runApp(MyApp());

class MyApp extends StatefulWidget {
  @override
  _MyAppState createState() => _MyAppState();
}

class _MyAppState extends State<MyApp> with WidgetsBindingObserver {
  String _location = "None";
  bool _isMockLocation = false;
  Timer getLocationCallback;

  @override
  void initState() {
    super.initState();
    WidgetsBinding.instance.addObserver(this);
    getLocationPermission();
    executeGetLocationCallback();
  }

  void executeGetLocationCallback() {
    getLocationCallback =
        Timer.periodic(Duration(seconds: 5), (Timer t) => _getLocation());
  }

  Future<void> _getLocation() async {
    String location;
    bool isMockLocation;
    try {
      location = await TrustLocation.getLocation;
      isMockLocation = await TrustLocation.isMockLocation;
    } on PlatformException catch(e) {
      print('PlatformException $e');
    }
    setState(() {
      _location = location;
      _isMockLocation = isMockLocation;
    });
  }

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        appBar: AppBar(
          title: const Text('Trust Location Plugin'),
        ),
        body: Padding(
          padding: const EdgeInsets.all(20.0),
          child: Center(
              child: Column(
            children: <Widget>[
              Text('Mock Location: $_isMockLocation'),
              Text('$_location'),
            ],
          )),
        ),
      ),
    );
  }

  void getLocationPermission() async {
    PermissionStatus permission =
        await LocationPermissions().requestPermissions();
    print('permissions: $permission');
  }

  @override
  void didChangeAppLifecycleState(AppLifecycleState state) {
    if (state == AppLifecycleState.resumed) {
      executeGetLocationCallback();
    }
    if (state == AppLifecycleState.inactive) {
      getLocationCallback.cancel();
    }
  }

  @override
  void dispose() {
    WidgetsBinding.instance.removeObserver(this);
    super.dispose();
  }
}
```

## Credits

Detecting the mock location: [LocationAssistant](https://github.com/klaasnotfound/LocationAssistant)

Requesting location permission: [location_permissions](https://pub.dev/packages/location_permissions)

## Issues

Please file any issues, bugs or feature request as an issue on our [issues](https://github.com/wongpiwat/flutter-trust-location/issues).

## Contribute

If you would like to contribute to the plugin, send us your [pull request](https://github.com/wongpiwat/flutter-trust-location/pulls).

## License

This open source project, and the license is BSD.
