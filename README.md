# flutter_google_map_example

### 1. Google Cloud Platform
[Google Cloud](https://console.developers.google.com/)

### 1.1. Maps API 활성화
> Google 지도 SDK를 추가
> 
> API 라이브러리  
> -> Maps SDK for Android 클릭 -> 사용 누르기  
> -> Maps SDK for IOS SDK 클릭 -> 사용 누르기

 

### 1.2. API 키 생성 및 제한

> 사용자 인증정보-> 사용자 인증정보 만들기 -> API 키 -> 키 제한

* android: 패키지명 + SHA1 입력

```shell
keytool -list -v -keystore "%USERPROFILE%\.android\debug.keystore" -alias androiddebugkey -storepass android -keypass android
```
* ios: 패키지명 입력

### 1.3. 안드로이드 설정

`android/app/src/main/AndroidManifest.xml`
```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.pinkesh.google_maps_flutter">
 
    <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION"/>
    <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION"/>
 
    <application
        android:label="google_maps_flutter"
        android:icon="@mipmap/ic_launcher">
 
       <!-- TODO: Add your API key here -->
       <meta-data android:name="com.google.android.geo.API_KEY"
           android:value="YOUR KEY HERE"/>
 
        <activity>...</activity>
    </application>
</manifest>

```

### 1.4. iOS 설정
`ios/Runner/AppDelegate.swift`
```swift
import UIKit
import Flutter
import GoogleMaps
  
@UIApplicationMain
@objc class AppDelegate: FlutterAppDelegate {
  override func application(
    _ application: UIApplication,
    didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?
  ) -> Bool {
    //Add your Google Maps API Key here
    GMSServices.provideAPIKey("YOUR KEY HERE")
    GeneratedPluginRegistrant.register(with: self)
    return super.application(application, didFinishLaunchingWithOptions: launchOptions)
  }
}

```

`ios/Runner/info.plist`
```plist
<key>NSLocationWhenInUseUsageDescription</key>
<string>The app needs location permission</string>
```

`pubspec.yaml`에 `google_maps_flutter` 플러그인 추가
```yaml
dependencies:
  flutter:
    sdk: flutter
  cupertino_icons: 1.0.0
  google_maps_flutter: ^2.0.1
```

예제코드
> gradle(app) : minSdkVersion 16->20으로 수정
```dart
import 'dart:async';
 
import 'package:flutter/material.dart';
import 'package:google_maps_flutter/google_maps_flutter.dart';
 
void main() {
  runApp(MaterialApp(
    home: Scaffold(
      appBar: AppBar(
        title: Text('GoogleMap'),
      ),
      body: MapSample(),
    ),
  ),
  );
}
 
class MapSample extends StatefulWidget {
  @override
  State<MapSample> createState() => MapSampleState();
}
 
class MapSampleState extends State<MapSample> {
  Completer<GoogleMapController> _controller = Completer();
 
  static final CameraPosition _kGooglePlex = CameraPosition(
    target: LatLng(37.42796133580664, -122.085749655962),
    zoom: 14.4746,
  );
 
  static final CameraPosition _kLake = CameraPosition(
      bearing: 192.8334901395799,
      target: LatLng(37.43296265331129, -122.08832357078792),
      tilt: 59.440717697143555,
      zoom: 19.151926040649414);
 
  @override
  Widget build(BuildContext context) {
    return new Scaffold(
      body: GoogleMap(
        mapType: MapType.hybrid,
        initialCameraPosition: _kGooglePlex,
        onMapCreated: (GoogleMapController controller) {
          _controller.complete(controller);
        },
      ),
      floatingActionButton: FloatingActionButton.extended(
        onPressed: _goToTheLake,
        label: Text('To the lake!'),
        icon: Icon(Icons.directions_boat),
      ),
    );
  }
 
  Future<void> _goToTheLake() async {
    final GoogleMapController controller = await _controller.future;
    controller.animateCamera(CameraUpdate.newCameraPosition(_kLake));
  }
}

```

## 2. Marker 찍기
```dart
import 'dart:math';
import 'package:flutter/material.dart';
import 'package:google_maps_flutter/google_maps_flutter.dart';
 
void main() {
  runApp(MyApp());
}
 
class MyApp extends StatelessWidget {
  // This widget is the root of your application.
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter Demo',
      theme: ThemeData(
        primarySwatch: Colors.blue,
        visualDensity: VisualDensity.adaptivePlatformDensity,
      ),
      home: MyHomePage(title: 'Flutter Demo Home Page'),
    );
  }
}
 
class MyHomePage extends StatefulWidget {
  MyHomePage({Key key, this.title}) : super(key: key);
 
  final String title;
 
  @override
  _MyHomePageState createState() => _MyHomePageState();
}
 
class _MyHomePageState extends State<MyHomePage> {
  // controller  for move the map in application
  GoogleMapController _controller;
 
  // this value is first  position when map is start.
  final CameraPosition _initialPosition =
  CameraPosition(target: LatLng(41.017901, 28.847953));
 
  // Marker list for  places to be marked when clicking on the map
  final List<Marker> markers = [];
 
  addMarker(cordinate) {
    int id = Random().nextInt(100);
 
    setState(() {
      markers
          .add(Marker(position: cordinate, markerId: MarkerId(id.toString())));
    });
  }
 
  @override
  Widget build(BuildContext context) {
    return Scaffold(
        body: GoogleMap(
          initialCameraPosition: _initialPosition,
          mapType: MapType.normal,
          onMapCreated: (controller) {
            setState(() {
              _controller = controller;
            });
          },
          markers: markers.toSet(),
 
          //the clicked position will be centered and marked
          onTap: (cordinate) {
            _controller.animateCamera(CameraUpdate.newLatLng(cordinate));
            addMarker(cordinate);
          },
        ),
        floatingActionButton: FloatingActionButton(
          onPressed: () {
            _controller.animateCamera(CameraUpdate.zoomOut());
          },
          child: Icon(Icons.zoom_out),
        ));
  }
}

```
### 2.1. 사용자 지정 위치 zoom 보여주기
```dart
import 'dart:async';
 
import 'package:flutter/material.dart';
import 'package:google_maps_flutter/google_maps_flutter.dart';
 
void main() {
  runApp(MaterialApp(
    home: HomePage(),
    ),
  );
}
class HomePage extends StatefulWidget {
  @override
  _HomePageState createState() => _HomePageState();
}
 
class _HomePageState extends State<HomePage> {
  Completer<GoogleMapController> _controller = Completer();
 
  CameraPosition _currentPosition = CameraPosition(
    target: LatLng(13.0827, 80.2707), //사용자 지정 좌표
    zoom: 12, //확대
  );
 
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text("Google Maps"),
      ),
      body: Container(
        height: double.infinity,
        width: double.infinity,
        child: GoogleMap(
          initialCameraPosition: _currentPosition,
          onMapCreated: (GoogleMapController controller) {
            _controller.complete();
          },
        ),
      ),
    );
  }
}
```

## [예제](https://blog.codemagic.io/creating-a-route-calculator-using-google-maps/)