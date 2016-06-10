# MdsClient-Android

MdsClient-Android is the library for accessing the Video Access Service (VAS) layer. It enables you to retrieve a video stream URL from any valid VideoID.

## Prerequisites

Android API Level 9 is required.

## Adding MdsClient to Your Project

1. Open your project in Android Studio. 
2. Click `File → New → New Module → Import .JAR/.AAR`.
3. In the _File name_ field, choose `mdsclient-VER.aar`. Replace `VER` with the actual version number.
4. Open your app's `build.gradle` file and add a dependency for the module you specified in Step 3, as shown in the sample code below. Once again, replace `VER` with the actual version number:

``` gradle
apply plugin: 'com.android.application'

android {
	...
	dependencies {
    	...
    	compile project(':mdsclient-VER')
	}
}
```

## Using MdsClient

Before starting, contact the Video Infrastructure Team at glomex to obtain the required credentials. 

### Constructing the MdsClient Object

Once you have the required credentials, use the constructor to create an object with the following parameters: 
`MdsClient(String baseUrl, String accessToken, String clientName, String salt)` 

For example:

``` java
MdsClient mdsClient = new MdsClient(
  getString(R.string.vas_base_url),
  getString(R.string.vas_token),
  getString(R.string.vas_client_name),
  getString(R.string.vas_salt)
);
```

### Using fetchStream

Use the asynchronous `fetchStream` method which returns either an `onSuccess` or an `onFailure` callback:

``` java
mdsClient.fetchStream(getString(R.string.vas_video_id), new MdsClient.Callback() {
  @Override
  public void onSuccess(Stream stream) {
      Video video = new Video(stream.getSources().get(Source.HLS), Video.Type.HLS);

      // use your Video object.
  }

  @Override
  public void onFailure(Exception e) {
      Toast.makeText(MainActivity.this, R.string.error_vas, Toast.LENGTH_LONG).show();
  }
});
```

The `Stream` object has the following fields:

``` java
public class Stream {
    private final Status               mStatus;
    private final Map<Integer, String> mSources;
    private final DrmMetaData          mDrmMetaData;
}

public static class DrmMetaData {
    public String licenseAcquisitionUrl;
    public String licenseToken;
    public String drmType;
}
```

### Handling Live Video Streams

For live video streams, please use the `MdsClientLive` class and its corresponding `fetchStream` method:

``` java
MdsClientLive mdsClientLive = new MdsClientLive(
	getString(R.string.vas_base_url),
	getString(R.string.vas_token),
	getString(R.string.vas_client_name),
	getString(R.string.vas_salt)
);

mdsClientLive.fetchStream(getString(R.string.vas_video_id), new MdsClientLive.Callback() {
    @Override
    public void onSuccess(final LiveStream stream) {
      // Get DRM metadata if needed
      LiveStream.DrmMetaData metaData = stream.getDrmMetaData();
      // Create Video object
      video = new Video(stream.getUrl(), Video.Type.DASH);
      // Play video
      playVideo(video);
    }

    @Override
    public void onFailure(final Exception exception) {…}
});
```
