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
    <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
 
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

### 3. 예제
### 3.1.
```dart
import 'package:flutter/material.dart';

void main() {
  runApp(MyApp());
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter Maps',
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      home: MapView(),
    );
  }
}

class MapView extends StatefulWidget {
  @override
  _MapViewState createState() => _MapViewState();
}

class _MapViewState extends State<MapView> {
  @override
  Widget build(BuildContext context) {
    // Determining the screen width & height
    var height = MediaQuery.of(context).size.height;
    var width = MediaQuery.of(context).size.width;

    return Container(
      height: height,
      width: width,
      child: Scaffold(
        body: Stack(
          children: <Widget>[
            // TODO: Add Map View
          ],
        ),
      ),
    );
  }
}
```

### 3.2.
```dart
// Import the Google Maps package
import 'package:google_maps_flutter/google_maps_flutter.dart';

// Initial location of the Map view
CameraPosition _initialLocation = CameraPosition(target: LatLng(0.0, 0.0));

// For controlling the view of the Map
GoogleMapController mapController;

// Replace the "TODO" with this widget
GoogleMap(
  initialCameraPosition: _initialLocation,
  myLocationEnabled: true,
  myLocationButtonEnabled: false,
  mapType: MapType.normal,
  zoomGesturesEnabled: true,
  zoomControlsEnabled: false,
  onMapCreated: (GoogleMapController controller) {
    mapController = controller;
  },
),
```

* initialCameraPosition : 초기 시작 시 지도 보기를 로드하는 데 사용되는 필수 매개변수입니다.
* myLocationEnabled : 지도에 파란색 점으로 현재 위치를 표시합니다.
* myLocationButtonEnabled : 사용자 위치를 카메라 뷰의 중앙으로 가져오는 데 사용되는 버튼입니다.
* mapType : 표시할 지도의 유형(일반, 위성, 하이브리드 및 지형)을 지정합니다.
* zoomGesturesEnabled : 지도 보기가 확대/축소 제스처에 응답해야 하는지 여부입니다.
* zoomControlsEnabled : 확대/축소 컨트롤을 표시할지 여부(Android 플랫폼에만 해당).
onMapCreated : 지도를 사용할 준비가 되었을 때 콜백합니다.

### 3.3.
```dart
// Design for current location button
 
ClipOval(
  child: Material(
    color: Colors.orange[100], // button color
    child: InkWell(
      splashColor: Colors.orange, // inkwell color
      child: SizedBox(
        width: 56,
        height: 56,
        child: Icon(Icons.my_location),
      ),
      onTap: () {
        // TODO: Add the operation to be performed
        // on button tap
      },
    ),
  ),
),
```

### 3.4.
```dart
// Zoom In action
mapController.animateCamera(
  CameraUpdate.zoomIn(),
);

// Zoom Out action
mapController.animateCamera(
  CameraUpdate.zoomOut(),
);
```

### 3.5.
```dart
// Move camera to the specified latitude & longitude
mapController.animateCamera(
  CameraUpdate.newCameraPosition(
    CameraPosition(
      target: LatLng(
        // Will be fetching in the next step
        _currentPosition.latitude,
        _currentPosition.longitude,
      ),
      zoom: 18.0,
    ),
  ),
);
```

### 3.6.
`pubspec.yaml`파일에 추가

```yaml
geolocator: ^5.3.1
```

```dart
final Geolocator _geolocator = Geolocator();

// For storing the current position
Position _currentPosition;
```

```dart
// Method for retrieving the current location
_getCurrentLocation() async {
  await _geolocator
      .getCurrentPosition(desiredAccuracy: LocationAccuracy.high)
      .then((Position position) async {
    setState(() {
      // Store the position in the variable
      _currentPosition = position;

      print('CURRENT POS: $_currentPosition');

      // For moving the camera to current location
      mapController.animateCamera(
        CameraUpdate.newCameraPosition(
          CameraPosition(
            target: LatLng(position.latitude, position.longitude),
            zoom: 18.0,
          ),
        ),
      );
    });
  }).catchError((e) {
    print(e);
  });
}
```

```dart
@override
void initState() {
  super.initState();
  _getCurrentLocation();
}
```

### 3.7.
```dart
final startAddressController = TextEditingController();
```

```dart
// Method for retrieving the address

_getAddress() async {
  try {
    // Places are retrieved using the coordinates
    List<Placemark> p = await _geolocator.placemarkFromCoordinates(
        _currentPosition.latitude, _currentPosition.longitude);

    // Taking the most probable result
    Placemark place = p[0];

    setState(() {
    
      // Structuring the address
      _currentAddress =
          "${place.name}, ${place.locality}, ${place.postalCode}, ${place.country}";
      
      // Update the text of the TextField
      startAddressController.text = _currentAddress;

      // Setting the user's present location as the starting address
      _startAddress = _currentAddress;
    });
  } catch (e) {
    print(e);
  }
}
```
```dart
// Getting the placemarks
List<Placemark> startPlacemark =
    await _geolocator.placemarkFromAddress(_startAddress);
List<Placemark> destinationPlacemark =
    await _geolocator.placemarkFromAddress(_destinationAddress);

// Retrieving coordinates
Position startCoordinates = startPlacemark[0].position;
Position destinationCoordinates = destinationPlacemark[0].position;
```

### 3.8. 마커
```dart
Set<Marker> markers = {};
```

```dart
// Start Location Marker
Marker startMarker = Marker(
  markerId: MarkerId('$startCoordinates'),
  position: LatLng(
    startCoordinates.latitude,
    startCoordinates.longitude,
  ),
  infoWindow: InfoWindow(
    title: 'Start',
    snippet: _startAddress,
  ),
  icon: BitmapDescriptor.defaultMarker,
);

// Destination Location Marker
Marker destinationMarker = Marker(
  markerId: MarkerId('$destinationCoordinates'),
  position: LatLng(
    destinationCoordinates.latitude,
    destinationCoordinates.longitude,
  ),
  infoWindow: InfoWindow(
    title: 'Destination',
    snippet: _destinationAddress,
  ),
  icon: BitmapDescriptor.defaultMarker,
);
```

```dart
// Add the markers to the list
markers.add(startMarker);
markers.add(destinationMarker);
```

### 3.9. 두 지점을 기준으로 지도 방향 변경
```dart
// Add the markers property to the widget
GoogleMap(
  markers: markers != null ? Set<Marker>.from(markers) : null,
  // ...
),
```


```dart
// Define two position variables
Position _northeastCoordinates;
Position _southwestCoordinates;

// Calculating to check that
// southwest coordinate <= northeast coordinate
if (startCoordinates.latitude <= destinationCoordinates.latitude) {
  _southwestCoordinates = startCoordinates;
  _northeastCoordinates = destinationCoordinates;
} else {
  _southwestCoordinates = destinationCoordinates;
  _northeastCoordinates = startCoordinates;
}

// Accommodate the two locations within the
// camera view of the map
mapController.animateCamera(
  CameraUpdate.newLatLngBounds(
    LatLngBounds(
      northeast: LatLng(
        _northeastCoordinates.latitude,
        _northeastCoordinates.longitude,
      ),
      southwest: LatLng(
        _southwestCoordinates.latitude,
        _southwestCoordinates.longitude,
      ),
    ),
    100.0, // padding 
  ),
);
```

### 4. 경로 그리기
`pubspec.yaml`파일에 패키지를 추가 합니다.

```yaml
flutter_polyline_points: ^0.2.1
```

```dart
// Object for PolylinePoints
PolylinePoints polylinePoints;

// List of coordinates to join
List<LatLng> polylineCoordinates = [];

// Map storing polylines created by connecting
// two points
Map<PolylineId, Polyline> polylines = {};
```

```dart
// Create the polylines for showing the route between two places

_createPolylines(Position start, Position destination) async {
  // Initializing PolylinePoints
  polylinePoints = PolylinePoints();

  // Generating the list of coordinates to be used for
  // drawing the polylines
  PolylineResult result = await polylinePoints.getRouteBetweenCoordinates(
    Secrets.API_KEY, // Google Maps API Key 
    PointLatLng(start.latitude, start.longitude),
    PointLatLng(destination.latitude, destination.longitude),
    travelMode: TravelMode.transit,
  );

  // Adding the coordinates to the list
  if (result.points.isNotEmpty) {
    result.points.forEach((PointLatLng point) {
      polylineCoordinates.add(LatLng(point.latitude, point.longitude));
    });
  }

  // Defining an ID
  PolylineId id = PolylineId('poly');

  // Initializing Polyline
  Polyline polyline = Polyline(
    polylineId: id,
    color: Colors.red,
    points: polylineCoordinates,
    width: 3,
  );

  // Adding the polyline to the map
  polylines[id] = polyline;
}
```

### 4.1. 지도에 ployline 그리기
```dart
// Add the polylines property to the widget
GoogleMap(
  polylines: Set<Polyline>.of(polylines.values),
  // ...
),
```

### 4.2. 거리계산

```dart
// Calculating the distance between the start and the end positions
// with a straight path, without considering any route

double distanceInMeters = await Geolocator().distanceBetween(
  startCoordinates.latitude,
  startCoordinates.longitude,
  destinationCoordinates.latitude,
  destinationCoordinates.longitude,
);
```

```dart
import 'dart:math' show cos, sqrt, asin;

double _coordinateDistance(lat1, lon1, lat2, lon2) {
    var p = 0.017453292519943295;
    var c = cos;
    var a = 0.5 -
        c((lat2 - lat1) * p) / 2 +
        c(lat1 * p) * c(lat2 * p) * (1 - c((lon2 - lon1) * p)) / 2;
    return 12742 * asin(sqrt(a));
  }
```

```dart
double totalDistance = 0.0;

// Calculating the total distance by adding the distance
// between small segments
for (int i = 0; i < polylineCoordinates.length - 1; i++) {
  totalDistance += _coordinateDistance(
    polylineCoordinates[i].latitude,
    polylineCoordinates[i].longitude,
    polylineCoordinates[i + 1].latitude,
    polylineCoordinates[i + 1].longitude,
  );
}

// Storing the calculated total distance of the route
setState(() {
  _placeDistance = totalDistance.toStringAsFixed(2);
  print('DISTANCE: $_placeDistance km');
});
```

### 4.2. 계산한 거리 지도에 표시
```dart
Visibility(
  visible: _placeDistance == null ? false : true,
  child: Text(
    'DISTANCE: $_placeDistance km',
    style: TextStyle(
      fontSize: 16,
      fontWeight: FontWeight.bold,
    ),
  ),
),
```