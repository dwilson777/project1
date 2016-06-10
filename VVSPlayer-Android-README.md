# VVSPlayer-Android

VVSPlayer-Android is the Android Video Player Framework.

## Features

* MP4, HLS and DASH streams
* Widevine and PlayReady DRM (platform dependent)
* VAST-compliant (Linear video ads - pre/mid/postrolls)

## Adding VVSPlayer to Your Project <a name="adding"></a>

1. Open your project in Android Studio. 
2. Click `File → New → New Module → Import .JAR/.AAR`
3. In the _File name_ field, choose `vvsplayer-VER.aar`. Replace `VER` with the actual version number.
1. Open your app's `build.gradle` file and add a dependency for the module you specified in Step 3, as shown in the sample code below. Once again, replace `VER` with the actual version number:

```
apply plugin: 'com.android.application'

android {
    ...
    dependencies {
        ...
        compile project(':vvsplayer-VER')
    }
}
```

## Using VVSPlayer
VVSPlayer is a powerful tool for video and ad playback, which offers a variety of possibilities for customization.

### Playing Content

#### Basic Integration

1. Integrate either a `VvsPlayerTextureView` or a `VvsPlayerSurfaceView` into your XML layout file. The key differences between Texture and Surface views are explained [here](https://github.com/crosswalk-project/crosswalk-website/wiki/Android-SurfaceView-vs-TextureView).
``` xml
 <com.glomex.vvsplayer.widget.video.VvsPlayerSurfaceView
     android:id="@+id/vvsplayer_view"
     android:layout_width="match_parent"
     android:layout_height="wrap_content"/>
```

2. In your `onCreate` or `onCreateLayout` method, obtain a reference to the `VvsPlayerView`:
``` java
 // Find view
 VvsPlayerView vvsPlayerView = (VvsPlayerView) findViewById(R.id.vvsplayer_view);
```

3. Create a new `Video` object with the stream URL and the video format. The supported formats are HLS, MP4 and DASH.
``` java
 // Create Video object
 Video video = new Video(getString(R.string.video_url), Video.Type.MP4);
```

4. Tell VvsPlayer to load the video and start it:
``` java
 // Load video object
 vvsPlayerView.load(video);
 // Start playing when ready
 vvsPlayerView.getPlayerControl().start();
```

#### Live Streams
The procedure for playing live streams is the same as in the basic integration. However, you may want to disable seeking and pausing of the playback as shown below:

``` java
vvsPlayerView.getPlayerControl().setCanPause(false)
vvsPlayerView.getPlayerControl().setCanSeek(false, false);
```

#### DRM Content
Playing DRM video content is just as easy as playing non-protected video content. Simply use the overloaded version of the `Video` object constructor, which takes a DRM-key as an argument (`drmKey` in the code below), as well as your DRM-protected video's URL (`streamUrl` in the code below):

``` java
String streamUrl = getString(R.string.my_stream_url);
String drmKey    = getString(R.string.my_drm_key);
Video  video     = new Video(streamUrl, Video.Type.DASH, drmKey);
```

#### Playing Ads
`VvsPlayer` supports playback of pre-, mid- and postroll inline video advertisements from [VAST](https://en.wikipedia.org/wiki/Video_Ad_Serving_Template)-compliant ad servers. To include Ad integration in your player, use `VvsPlayerAdView` instead of `VvsPlayerTextureView` or `VvsPlayerSurfaceView`. You will also have to supply an `Ad` object with your ad server's address.

``` xml
<com.glomex.vvsplayer.widget.video.VvsPlayerAdView
    android:id="@+id/ad_view"
    android:layout_width="match_parent"
    android:layout_height="match_parent"/>

<com.glomex.vvsplayer.widget.control.AdControlsView
    android:id="@+id/controls"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    app:progress_bar_color="@color/colorAccent"
    app:progress_bar_marker_drawable="@drawable/custom_ad_marker"/>
```

``` java
// Find views
VvsPlayerAdView vvsPlayerAdView = (VvsPlayerAdView) findViewById(R.id.ad_view);
AdControlsView adControlsView = (AdControlsView) findViewById(R.id.controls);

// Create Video and Ad objects
Video video = new Video(getString(R.string.video_url), Video.Type.MP4);
Ad ad = new Ad(getString(R.string.ad_url_pre_mid_post));

// Setup callbacks
adControlsView.setPlayerControl(vvsPlayerAdView.getPlayerControl());
vvsPlayerAdView.addAdCallbacks(adControlsView);

// Load Video object and start playing.
// If you are using `ControlsLayout`, it will automatically detect and display your ad markers.
vvsPlayerAdView.load(video, ad);
vvsPlayerAdView.getPlayerControl().start();
```

You can also listen to ad events by setting `AdCallbacks`:

``` java
 vvsPlayerAdView.setAdCallbacks(new VvsPlayerAdView.AdCallbacks() {
     @Override
     public void onAdCuePointsLoaded(List<Float> cues) { /*...*/ }
     @Override
     public void onAdStarted(AdUnit adUnit) { /*...*/ }
     @Override
     public void onAdClicked(AdUnit adUnit) { /*...*/ }
     @Override
     public void onAdFinished(AdUnit adUnit) { /*...*/ }
     @Override
     public void onAdError(AdError adError) { /*...*/ }
 });
```

### Player Controls
`VvsPlayer` implements the Android `MediaPlayerControl` interface, which allows it to interact with most controllers, including the system's `MediaController`. Standard methods for controlling the player include `start()`, `pause()`, `seekTo(int)`.
Getters for current position, duration and playback state are also available.

#### Default Controls
`VvsPlayer` comes with default controls that support all of the common interactions: play/pause, seeking, displaying the buffer state and ad markers.

1. To integrate the default controls, add a `ControlsView` to your XML:
 ``` xml
 <com.glomex.vvsplayer.widget.control.ControlsView
         android:id="@+id/controls_view"
         android:layout_width="match_parent"
         android:layout_height="wrap_content"/>
 ```

2. Obtain your `ControlsView` and connect it to your player:
 ``` java
 // Config layouts
 ControlsView controlsView = (ControlsView) findViewById(R.id.controls_view);

 // Connect ControlsView and VideoPlayer
 controlsView.setPlayerControl(vvsPlayerView.getPlayerControl());
 ```

3. Now you can now control your video's playback.

#### Custom Controls
`VvsPlayer` allows you to use custom controls by implementing the `MediaPlayerControl` interface.

An easier option is to extend VvsPlayer's `BaseControlsView` control class and override event methods from `BaseControlsView` in order to react to events. For example, for a play button you could listen to the `onPaused` and the `onPlaying` events:
``` java
@Override
public void onPaused() {
    playButton.setImageResource(android.R.drawable.ic_media_pause);
}

@Override
public void onPlaying() {
    playButton.setImageResource(android.R.drawable.ic_media_play);
}
```

### Fullscreen video playback
For seamless fullscreen integration you can use a helper library, such as [Leanback](https://github.com/7factory/mia-Leanback). It toggles notification and action bars when necessary:

* Extend your `Activity` from `LeanbackActivity`:
``` java
public class MainActivity extends LeanbackActivity { /*...*/ }
```
* Call `toggleFullscreen()`.

* You can also override the `onFullscreenChanged(boolean, boolean)` method in order to react to fullscreen toggle events, for example, to set the correct layout parameters for your fullscreen view:
``` java
@Override
public void onFullscreenChanged(boolean isFullscreen, boolean isSystemUiVisible) {
    super.onFullscreenChanged(isFullscreen, isSystemUiVisible);

    if (isFullscreen) {
        vvsPlayerView.setLayoutParams(FULLSCREEN_LAYOUT_PARAMS);
    } else {
        vvsPlayerView.setLayoutParams(EMBEDDED_LAYOUT_PARAMS);
    }
}
```

### Integration with Video Access Service (VAS)
glomex provides an `MdsClient` library which is a VAS (Video Access Service) layer. To integrate it into your project, repeat the steps from the [Adding VVSPlayer to Your Project](#adding) section above, but substitute `mdsclient-VER.aar` library, where `VER` is the actual version number.  

Please contact the Video Infrastructure Team at glomex to get the required credentials and then use the constructor to create an object:

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

Use the asynchronous `fetchStream` method which delivers either an `onSuccess` or an `onFailure` callback.

``` java
mdsClient.fetchStream(getString(R.string.vas_video_id), new MdsClient.Callback() {
    @Override
    public void onSuccess(Stream stream) {
        Video video = new Video(stream.getSources().get(Source.HLS), Video.Type.HLS);

        // Play video
        vvsPlayerView.load(video);
        vvsPlayerView.getPlayerControl().setCanSeek(false, false);
        vvsPlayerView.getPlayerControl().start();
    }

    @Override
    public void onFailure(Exception e) {
        Toast.makeText(MainActivity.this, R.string.error_vas, Toast.LENGTH_LONG).show();
    }
});
```

For more information, refer to the `MdsClient` documentation.

### Documentation

Comprehensive JavaDoc is available in the `javadoc` sub-directory.
