# MDSClientKit

MDSClientKit is the iOS Client Library for accessing the Video Access Service (VAS) layer. It enables you to retrieve a video stream URL from any valid VideoID.

## Prerequisites
Ensure that you have the following:

* iOS v8.0
* Xcode v7.3.1

## Adding MDSClient to Your XCode Project

1. Copy `MDSClientKit.framework` and `HydrogenKit.framework` to your project folder (e.g. `{project_root}/Libraries/`).
1. Open your project or workspace in Xcode. 
2. Open _Project Settings_ (select the name of the project in the project navigator). 
3. Select main Target from the *Targets* list. 
4. In the _General_ tab, locate the _Embedded Binaries_ section and click the _Plus_ sign.
5. Add both `MDSClientKit.framework` and `HydrogenKit.framework`.
6. In the _Build Settings_ tab, locate the _Build Options_ section and set _Embedded Content Contains Swift Code_ to `YES`.
7. Select Project from the *Projects* list and repeat Step 6 for it.

## Using MDSClient
The code snippet below demonstrates how to fetch a streaming URL:

``` swift
let config = MDSClientConfig(
  accessToken: "YourAccessToken",
  clientName: "YourClientName",
  salt: "YourSalt",
  baseURL: NSURL(string: "YourLinkToVAS")!
)
let mdsClient = MDSClient(config: config)
mdsClient.fetch("YourVideoID") { source, error in
  print(source.url, error)
}
```
The code snippet above has three main steps to fetch a streaming URL:

* A `config` object is created. Please contact your Technical Account Manager (TAM) to obtain the required credentials.
* An instance of the `MDSClient` main class is instantiated with the `config` object.
* A `fetch` call is performed to retrieve a streaming URL for a video id. The call takes a video id as its first parameter (`YourVideoID` in the code above) and completion closure as its second parameter. The provided `source` struct contains the video details with a streaming URL and optional DRM information.

### MDSClientSource

The source struct retrieved from a fetch call is of type `MDSClientSource` and contains the following properties:

``` swift
streamURL: NSURL
drmInfo: DRMInfo?
```

`DRMInfo` contains:
``` swift
certificateURL: NSURL?
licenseURL: NSURL?
token: String?
```

### Live streams

For live streams, please use the `MDSClientLive` class which is quite similar to `MDSClient`. Simply provide it with `MDSClientLiveConfig`:

``` swift
let liveConfig = MDSClientLiveConfig(
  accessToken: "YourAccessToken",
  clientLocation: "YourClientLocation",
  salt: "YourSalt",
  baseURL: NSURL(string: "YourLinkToVAS")!
)

let mdsLiveClient = MDSClientLive(config: liveConfig)
mdsLiveClient.fetch("YourLiveStreamID") { source, error in
  print(source.url, error)
}
```
