---
layout: post
title:  "DRI nearby?"
date:   2015-10-18
categories: android mapping
---

Have you ever been out walking and wondered if DRI contains any objects related to your current location?
No, neither have I, but having taken a short online course in Android development I thought I'd put together
a simple app to do this.

First we need a basic Android application that will display a map. There are plenty of guides 
available on how to do this, for example [here][android-map].

Next is to find out where the user is and to receive updates on their location
as they move. How to do this is covered well in the [Android developer guide][android-location-updates].

A very brief summary is:

Give the application permission to access location services in the app manifest

{% highlight xml %}
<uses-permission 
    android:name="android.permission.ACCESS_FINE_LOCATION"/>
{% endhighlight %}

Create an API client and a LocationRequest object

{% highlight java %}
mGoogleApiClient = new GoogleApiClient.Builder(this)
                .addConnectionCallbacks(this)
                .addOnConnectionFailedListener(this)
                .addApi(LocationServices.API)
                .build();

// Create the LocationRequest object
mLocationRequest = LocationRequest.create()
               .setPriority(LocationRequest.PRIORITY_HIGH_ACCURACY)
               .setInterval(10 * 1000) // 10 seconds
               .setFastestInterval(1 * 1000); // 1 second
{% endhighlight %}

Once the client is connected (i.e., in the ```onConnected()``` callback) request location updates

{% highlight java %}
LocationServices.FusedLocationApi.requestLocationUpdates(
            mGoogleApiClient, mLocationRequest, this);
{% endhighlight %}

The current location will be updated through the ```onLocationChanged()``` callback

{% highlight java %}
@Override
public void onLocationChanged(Location location) {
     handleNewLocation(location);
}
{% endhighlight %}

Each time we receive a location update we process it in the ```handleNewLocation()``` method. In this method
the user's current latitude and longitude are retrieved, a marker is added to the map, and this location
is then passed to the code that handles finding the nearby objects from the DRI repository.

{% highlight java %}
private void handleNewLocation(Location location) {
        Log.d(TAG, location.toString());

        // Get the current latitude and longitude
        double currentLatitude = location.getLatitude();
        double currentLongitude = location.getLongitude();

        // Add a marker to the map showing the user where they are
        LatLng latLng = new LatLng(currentLatitude, currentLongitude);
        MarkerOptions options = new MarkerOptions()
                .position(latLng)
                .title("You are here!");
        mMap.addMarker(options);

        // Move and zoom the camera to their location
        mMap.animateCamera(CameraUpdateFactory
                           .newLatLngZoom(latLng, 12.0f));

        // Find and display the locations of nearby objects
        String locationUrl = String.format(url, 
                 currentLatitude, currentLongitude);
        new GetObjectLocations(locationUrl, 
                 currentLatitude, currentLongitude).execute();
}
{% endhighlight %}

In a previous post I talked about mapping in the DRI repository. Adding the mapping interfaces involved configuring Solr
to support spatial searches. We can use this functionality here to retrieve the objects close to the user's location.

The url to perform a spatial query looks like this:

```
https://repository.dri.ie/catalog?spatial_search_type=point
&coordinates=53.3483,-6.2572&format=json
```

The response is in JSON format and contains the Solr documents of all the objects matching the query (i.e., all the objects
within a certain spatial distance of the given co-ordinates). Rather than display every object the app will only add
markers at each place, with the number of objects at that place added as a label. Querying the repository and processing
the results is all performed in an [AsyncTask][android-async] separate to the MainActivity. This is done as the network request
and subsequent processing could take some time and the app should not stop responding during this.

To retrieve the response a connection is made to the search URL followed by a GET request

{% highlight java %}
URL url = new URL(requestUrl);
// http client
HttpURLConnection conn = (HttpURLConnection) url.openConnection();
conn.setRequestMethod("GET");
conn.connect();
is = conn.getInputStream();
// Convert the InputStream into a string
contentAsString = readIt(is);
{% endhighlight %}

{% highlight java %}
// Reads an InputStream and converts it to a String.
    public String readIt(InputStream stream) 
      throws IOException {
        BufferedReader reader =new BufferedReader(
            new InputStreamReader(stream, "UTF-8"));
        String response = "",data="";

        while ((data = reader.readLine()) != null){
            response += data;
        }

        return response;
}
{% endhighlight %}

```contentAsString``` contains the JSON response and this is returned for parsing.

The [org.json][android-json] API is used to parse the JSON. As stated above, as the app will
only display counts of the objects at locations we can parse the facet output contained in the response
rather than each object.

{% highlight java %}
// Create a JSONObject from the String response
JSONObject jsonObj = new JSONObject(jsonStr);

// Get the facets for each place
JSONArray places = getPlacesFromFacets(jsonObj);

if (places != null) {
  // looping through facets
  for (int i = 0; i < places.length(); i++) {
    JSONObject p = places.getJSONObject(i);
    // How many objects at this place
    String hits = p.getString(TAG_HITS);
        
    // Get the GeoJSON formatted place metadata
    String geojson_ssim = p.getString(TAG_VALUE);
    JSONObject geojsonObject = new JSONObject(geojson_ssim);

    // Get the placename
    String placename = getPlacename(geojsonObject);

    JSONObject geometry = geojsonObject
      .getJSONObject(TAG_GEOMETRY);
    String type = geometry.getString(TAG_TYPE);

    // Only consider points, not bounding boxes
    if (type.equals("Point")) {
      // Get the co-ordinates of the place
      JSONArray coordinates = geometry
        .getJSONArray(TAG_COORDINATES);

      String latitude = coordinates.getString(1);
      String longitude = coordinates.getString(0);

      // As some objects contain multiple places, only
      // use those within the required range
      if (withinRange(latitude, longitude)) {
        HashMap<String, String> placeEntry = 
          new HashMap<>();
        placeEntry.put(TAG_HITS, hits);
        placeEntry.put(TAG_LATITUDE, latitude);
        placeEntry.put(TAG_LONGITUDE, longitude);

        placeList.put(placename, placeEntry);
      }
  }
}
}
{% endhighlight %}

For each entry added to the ```placeList``` a marker is added to the map.

{% highlight java %}
private void addMarker(String placename, HashMap<String, String> placeEntry){
  String noOfObjects = placeEntry.get(TAG_HITS);
  String latitude = placeEntry.get(TAG_LATITUDE);
  String longitude = placeEntry.get(TAG_LONGITUDE);

  MarkerOptions options = new MarkerOptions()
             .icon(BitmapDescriptorFactory.fromBitmap(
                       iconFactory.makeIcon(noOfObjects)))
             .position(new LatLng(Double.valueOf(latitude),
                       Double.valueOf(longitude)))
             .anchor(iconFactory.getAnchorU(), 
                     iconFactory.getAnchorV())
             .title(placename);
  mMap.addMarker(options);
}
{% endhighlight %}

The marker shows the number of objects. 

![User's location and the place markers displayed on the map]({{ site.url }}/assets/android_map_example.png)

Clicking on the marker displays the placename.

![Place title links to Repository search]({{ site.url }}/assets/android_map_marker.png)

Clicking on the title opens the device's browser and displays the objects in the repository that contain that placename in their metadata.

![Nearby objects displayed in Repository]({{ site.url }}/assets/android_map_results.png)

The screenshots above show the working application running on an emulator of a Nexus 5.

[android-map]:                https://developers.google.com/maps/documentation/android-api/start
[android-location-updates]:   http://developer.android.com/training/location/receive-location-updates.html
[android-json]:               http://developer.android.com/reference/org/json/package-summary.html
[android-async]:              http://developer.android.com/guide/components/processes-and-threads.html#AsyncTask
