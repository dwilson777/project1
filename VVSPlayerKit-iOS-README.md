# VVSPlayerKit

VVSPlayerKit is the iOS Video Player Framework.

## Features
VVSPlayerKit has these features:

- AVPlayer-backed
- Ad playback
- DRM (Fairplay)
- Airplay
- Picture-in-Picture
- tvOS
- Player Controls
- Fullscreen support

## Prerequisites

Ensure that you have the following:

* iOS v8.0
* Xcode v7.3.1

## Adding the VVSPlayerKit to Your Xcode Project <a name="adding"></a>

1. Copy `VVSPlayerKit.framework` to your project folder (e.g. `{project_root}/Libraries/`).
2. Open your project or workspace in Xcode. 
3. Open _Project Settings_ (select the name of the project in the project navigator). 
4. Select main Target from the *Targets* list. 
5. In the _General_ tab, locate the _Embedded Binaries_ section and click on the _Plus_ sign.
6. Add `VVSPlayerKit.framework`.
7. In the _Build Settings_ tab, locate the _Build Options_ tab and set _Embedded Content Contains Swift Code_ to `YES`.
8. Select Project from the *Projects* list and repeat Step 7 for it.

## Using the VVSPlayerKit

In your Storyboard or Xib, add a container UIView to your ViewController.
To use the default controls, add another UIView to the ViewController. In the Attribute Inspector, set the `Class` attribute to `VideoPlayerControls` and the `Module` attribute to `VVSPlayerKit`. Xcode then renders a live preview of the default controls. For additional styling, adjust the provided custom attributes.

After configuring the views, create Outlets for the video container and controls.

In your main ViewController class, import the `VVSPlayerKit` framework:

``` swift
import VVSPlayerKit
```

In `viewDidLoad`, set up the main classes and define the dependencies:

``` swift
let videoPlayer = VideoPlayer(streamURL: video.url)
vvsPlayer = VVSPlayer(player: videoPlayer)
vvsPlayer.delegate = self
vvsPlayer.registerControls(playerControls)

videoPlayer.videoView.addToView(videoContainerView)
```

The code snippet above has three main steps which configure the player:

* A `VideoPlayer` is initialized with a `streamURL` and then inserted into `VVSPlayer`. 
* If you want to react to player events, implement the `VVSPlayerDelegate` protocol and set the `delegate` property.
* Player controls are registered with `VVSPlayer`. You can register multiple controls that conform to the `PlayerControls` protocol.

**Note**: Call `vvsPlayer.close()` when you want to close the stream and deallocate the player. After a `close()`, VVSPlayer must be re-instantiated.

### Ad Playback
If you wish to support ad playback, insert an instance of `AdVideoPlayer` instead of `VideoPlayer` into `VVSPlayer` and add the container view displaying ads to your view hierarchy. (For standard fullscreen behavior, you should attach `adView` to `videoView`.)

``` swift
let adPlayer = AdVideoPlayer(streamURL: video.url, adURL: adURL, adDisplayThreshold: 3)
adPlayer.adView.addToView(adPlayer.videoView)
```

`adURL` is a URL to a Video Multiple Ad Playlist (VMAP). For testing, you can use [sample VMAPs](https://developers.google.com/interactive-media-ads/docs/sdks/html5/tags) which provide different configurations for prerolls, midrolls and postrolls. `adDisplayThreshold` sets the minimum duration value for ads to be visible. This property can be used for situations where ads have to be tracked but should not be visible to user.

### DRM with Fairplay

You can implement your own custom DRM handling by inserting an object that implements `AVAssetResourceLoaderDelegate` into the video player classes. This delegate is then used internally by the player for resource loading. Please refer to the Apple documentation and contact your key server provider on how to implement Fairplay DRM for your specific use case.

VVSPlayer comes with an implementation of Fairplay (Apple's DRM solution) which is targeted at the glomex Video Access Service and license server. In order to play a DRM-protected stream, both `VideoPlayer` and `AdVideoPlayer` can be configured with an instance of `AssetLoaderDelegate`. The `AssetLoaderDelegate` parameters `licenseURL`, `certificateURL` and `token` can be retrieved using the framework `MDSClientKit`, which handles communication with the Video Access Service (VAS).

### Playback Control
`VVSPlayer` comes with standard methods for controlling the player, such as `play`, `pause` and `seekTo`. Also, see the `PlayerController` protocol for available read-only properties, e.g. for retrieving the `progress`, `duration` or `playbackState`.

### Fullscreen Presentation
If you want to use the standard fullscreen behavior, implement the delegate method: `VVSPlayerSourceViewControllerForFullscreenPresentation` 
and return the source view controller where fullscreen presentation should be initiated. When VVSPlayer is programmatically set to full screen mode, or when a user interaction sets it to fullscreen mode, it instantiates a new storyboard and moves the video view into the container view of the fullscreen storyboard.

If you want to modify the layout or appearance of the fullscreen presentation, copy `FullscreenView.storyboard` from the source files, change it, and then tell VVSPlayer to use your custom storyboard:

``` swift
func fullscreenViewController(vvsPlayer: VVSPlayer) -> FullscreenViewController? {
    return FullscreenViewController.create(vvsPlayer, storyboardName: "CustomFullscreenView", bundle: NSBundle.mainBundle())
}
```

If you need further control over fullscreen presentation, you can disable the default behaviour by returning `nil` from `vvsPlayerSourceViewControllerForFullscreenPresentation`. You can then react to VVSPlayer's fullscreen events and implement your own presentation logic.

### Custom Controls
VVSPlayer allows you to register your own controls with the `registerControls` method. This must be a class conforming to the `PlayerControls` protocol. Your controls class then receives an instance of a `PlayerController` (inserted through the property `playerController`) that exposes methods for controlling the playback. For instance, your custom play button could call `playerController.play()`.

Conversely, the `playerController` calls various update methods on your instance of `PlayerControls` (e.g., `updatePlaybackState(playbackState: PlaybackState)`, which you can use to update the display of your view components.

If you want to change the player controls during ad playback, you can react to the `canSeek` parameter provided via `updateCanSeek`. This property is set to `false` when ad playback starts and is reset back to `true` when ads have finished.

Progress updates must be handled internally by the `PlayerControls` so as to leave it up to the controls to decide when to schedule display updates. This could be done by starting a timer or DisplayLink updates and querying the current playback progress from the `playerController` property.

**Note**: Fullscreen mode uses a separate instance of the controls so that their layout and appearance can be modified for fullscreen presentation. You can change the controls by using a custom fullscreen storyboard or completely controlling fullscreen mode yourself (as described in the section above).

## Integration with Video Access Service (VAS)
glomex provides `MDSClientKit` library which is a Video Access Service (VAS) layer. To integrate it into your project, repeat steps from the [Adding the VVSPlayerKit to Your Xcode Project](#adding) section above but for `MDSClientKit.framework`.

``` swift
private var mdsClient: MDSClient = {
        let config = MDSClientConfig(
            accessToken: accessToken,
            clientName: clientName,
            salt: salt,
            baseURL: NSURL(string: baseUrl)!
        )
        return MDSClient(config: config)
    }()

mdsClient.fetch(videoID) { [weak self] source, error in
    dispatch_async(dispatch_get_main_queue(), {    

        // Setup player with given source
        let player = VideoPlayer(
            streamURL: source.streamURL,
            assetLoaderDelegate: nil
        )
        self?.setupPlayer(player)
    })
}
```

For more information, refer to the `MDSClientKit` documentation.
