# Map

The map is provided via fragment. This fragment has to be placed in a `FrameLayout`. To work with the map you have to subscribe to the map observable. The map can be animated via `CameraUpdate`, use the `CameraUpdateFactory` to create an instance. Several map types are supported by GMS, unfortunately only two are supported by HMS:  
- MAP_TYPE_NONE
- MAP_TYPE_NORMAL
- MAP_TYPE_SATELLITE -> not provided by HMS yet
- MAP_TYPE_TERRAIN   -> not provided by HMS yet
- MAP_TYPE_HYBRID    -> not provided by HMS yet

## Setup
Be sure to enable "Maps SDK for Android" in the Google Cloud Console and create an API key. This API key needs to be declared in your Manifest:
```xml
<meta-data
    android:name="com.google.android.geo.API_KEY"
    android:value="..." />
```

If the app accesses the user's current location by enabling the `MyLocation` layer, you must request location permission as well (see [Location](./location.md)).

## Usage

Create the map fragment and add it to a `FrameLayout` with id `mapContainer`:
```kotlin
val fragmentManager: FragmentManager = supportFragmentManager
val fragmentTransaction: FragmentTransaction = fragmentManager.beginTransaction()

val mapFragment: MapFragment = MapFragment.newInstance()
fragmentTransaction.add(R.id.mapContainer, mapFragment)
fragmentTransaction.commit()
```

Working with the map:
```kotlin
val mapDisposable: Disposable =
  mapFragment.getMapObservable().subscribeWith(object : DisposableObserver<Map>() {
      override fun onComplete() {}

      override fun onNext(map: Map) {
          val uiSettings = map.getUiSettings()
          uiSettings.isCompassEnabled = true
          uiSettings.isMyLocationButtonEnabled = true
          uiSettings.isRotateGesturesEnabled

          map.isMyLocationEnabled = true
          map.mapType = Map.MAP_TYPE_NORMAL
      }

      override fun onError(e: Throwable) {}
  })
```

Using the location to animate the camera to the current position:
```kotlin
val fused: FusedLocationProviderClient = FusedLocationProviderFactory.getFusedLocationProviderClient(this)
fused.getLastLocation()
  .addOnSuccessListener(object : OnSuccessListener<Location?> {
      override fun onSuccess(result: Location?) {
          if (result != null) {
            val location = LatLng(result.latitude, result.longitude)
            map.animateCamera(
              CameraUpdateFactory.instance().newLatLngZoom(location, 13f),
              4000,
              null
            )
          }
      }
  })
```

Adding a marker:
```kotlin
map.addMarker(
    MarkerOptions.create()
        .defaultIcon()
        .position(LatLng(47.871026, 13.548118))
)
```

Adding a circle:
```kotlin
map.addCircle(
    Circle.CircleOptions()
        .center(location)
        .radius(100.0)
        .fillColor(Color.YELLOW)
)
```

Adding a polygon:
```kotlin
val locationHole1: LatLng = LatLng.midPoint(location2, location3)
val locationHole2: LatLng = location1

map.addPolygon(
    PolygonOptions()
        .add(location1)
        .add(location2)
        .add(location3)
        .fillColor(Color.RED)
        .strokeColor(Color.GREEN)
        .strokeWidth(3f)
        .addHole(listOf(locationHole1, locationHole2))
)
```

Adding a polyline:
```kotlin
map.addPolyline(
    PolylineOptions()
        .add(location1, location2)
        .strokeWidth(15f)
        .startCap(Cap.RoundCap())
        .endCap(Cap.SquareCap())
        .zIndex(2f)
)
```

## Links
- [GMS Maps](https://developers.google.com/maps/documentation/android-sdk/overview)
- [HMS Maps](https://developer.huawei.com/consumer/en/hms/huawei-MapKit)