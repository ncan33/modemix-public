# Modemix for Android

Public releases of the Modemix Android app.

Download the latest APK from the [Releases](https://github.com/ncan33/modemix-public/releases) page.

## MapLibre + Jetpack Compose

Modemix uses [MapLibre Native](https://maplibre.org/) for its map. Here's a minimal example of embedding a MapLibre map in Jetpack Compose using `AndroidView`:

```kotlin
@Composable
fun MapScreen() {
    val context = LocalContext.current
    val lifecycleOwner = LocalLifecycleOwner.current

    val mapView = remember {
        val options = MapLibreMapOptions.createFromAttributes(context)
            .camera(CameraPosition.Builder().target(LatLng(37.7749, -122.4194)).zoom(12.0).build())
        MapView(context, options)
    }

    AndroidView(factory = {
        mapView.apply {
            onCreate(null)
            getMapAsync { map ->
                map.setStyle("https://tiles.openfreemap.org/styles/liberty")
            }
        }
    })

    // Forward lifecycle events to MapView
    DisposableEffect(lifecycleOwner) {
        val observer = LifecycleEventObserver { _, event ->
            when (event) {
                Lifecycle.Event.ON_START -> mapView.onStart()
                Lifecycle.Event.ON_RESUME -> mapView.onResume()
                Lifecycle.Event.ON_PAUSE -> mapView.onPause()
                Lifecycle.Event.ON_STOP -> mapView.onStop()
                Lifecycle.Event.ON_DESTROY -> mapView.onDestroy()
                else -> {}
            }
        }
        lifecycleOwner.lifecycle.addObserver(observer)
        onDispose { lifecycleOwner.lifecycle.removeObserver(observer) }
    }
}
```

This uses [OpenFreeMap](https://openfreemap.org/) tiles, which require no API key.
