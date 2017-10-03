# Introduction

The Predictive Content Delivery (PCD) SDK allows users to download videos intelligently to mobile devices. Users initiate downloads by browsing your video catalog and selecting the content to watch.

A sophisticated Download Manager checks the battery, device storage, network, and other variables to make sure that the video transfer will succeed. Errors are thrown if the video transfer fails.

Downloads continue whether the app is active in the foreground, running in the background, or entirely closed. In addition, the SDK acts as a data cache, providing access to content along with its current download state.

## Audience

This document is for developers implementing the PCD SDK; deep knowledge of iOS is assumed.

There are separate instructions for Android and iOS. Configuring your project to support these platforms is covered in the following chapters.
# Getting Started with iOS
The fastest way to get started with the SDK is to try the sample app. Once you've downloaded the Zip file and changed the necessary requirements and dependencies, you're ready to go.

After you're done playing with the sample app, the next steps are SDK initialization and registration. These steps are covered in detail in Integrating With Your iOS Application.

## Requirements and Dependencies

- iOS 8 or later.
- A PCD SDK license key is required.
- In order to cache non secure http content, the app must enable App Transport Security.
- In order for faster downloading, the app must enable Background Execution and Remote Notifications.


## SDK Contents
When you download the PCD SDK zip, the contents include: 

* A <tt>docs</tt> folder - has documentation for API reference.
* <tt>examples</tt> folder - has D2G example app based on swift.
* <tt>iphoneos</tt> folder - has device specific SDK framework.
* <tt>iphonesimulator</tt> folder - has simulator specific SDK framework.
* <tt>vocSdk.framework</tt> file - is a FAT SDK framework.
* </tt>Integration guide</tt> - walks you through PCD SDK integration.

## Trying the Sample App

The easiest way to get started is to try one of the sample apps.

1. Open <tt>D2GExample</tt> app in the <tt>examples/</tt> folder.
2. Substitute your SDK license key instead of the dummy value in the info.plist file.
```
<key>com.akamai</key>
	<dict>
            <key>vocsdk</key>
            <dict>
		<key>license</key>
		<string>your_license_key_goes_here</string>
            </dict>
	</dict>
```
3. Open the VideosViewController in the D2GExample and update the on demand content details. This includes content ids to download and the provider name.
4. Run the app.

# Integrating With Your iOS Application

To start using the PCD SDK with your own app, you'll need to include the SDK in your app. Once these are complete, you can continue with initialization and registration.

## Installing the iOS SDK
1. Download the PCD SDK and unzip it in a folder under your project, e.g.<tt> ~/myproject/pcd_sdk.</tt>
2. Locate <tt>VocSdk.framework</tt> **fat** binary under<tt> ~/myproject/pcd_sdk/</tt> and copy it into your project folder, e.g.<tt> ~/myproject/VocSdk.framework.</tt>
3. Add the framework to your Xcode project by opening your project in Xcode and choose **File > Add Files to...** and choose <tt>~/myproject/VocSdk.framework</tt>.
4. Click the project name in the Project navigator to open the project settings.
5. Click the  **General** tab, and under **Embedded Binaries**, click  **+** and choose  **VocSdk.framework**. Click  **Add**.


## Adding a Build Phase
When you attempt to submit your app to Apple’s App Store with **fat** binary, it will be automatically rejected because the **fat** binary has iPhone Simulator **slices** within it. For this reason, we supply an alternative script that will only include the binaries for the architectures that your build supports. The script will only process when the build configuration is for a **Release** build (e.g., when you archive a build).
1. In Project Settings, choose your build target then click on <tt>Build Phases</tt>.
2. Click the **+** to add a new build phase, and choose <tt>New Run Script Phase.</tt>
3. Ensure this build phase occurs after the Embed Frameworks phase. By default, your new script will be the last step, and this is fine.
Optionally single-click its name and rename it to strip_vocsdk.
4. Click the arrow to expand the strip_vocsdk row.
5. Leave the default shell setting of /bin/sh.
6. Add the following text to the script area.  The **strip_vocsdk.sh** path is relative to your .xcodeproj file and may need modification depending on where you unzipped it (pcd_sdk in this example).


## Including the SDK
Import the ```VocSdk``` header in any class files where the SDK is required. Also define a single ```VocService``` object for your app. Create your ```VocService``` object in the app delegate to make the SDK available early in the app lifecycle and to simplify access to the ```VocService``` from other classes.

```
#import <VocSdk/VocSdk.h>
@property (strong , nonatomic) id <VocService>  vocService;
```

## Enabling App Transport Security
Starting in iOS 9.0, a new app security feature called App Transport Security (ATS) was introduced and is enabled by default. With ATS enabled, connections must use secure HTTPS instead of HTTP. Additionally, if the app contents that PCD SDK needs to download contain non-SSL items, those downloads will fail. Application developers must ensure that either

- [preferred] HTTPS is used for all content URLs, or
- [insecure] ATS exceptions can be added for certain domains by adding the following <tt>key-subkey</tt> to the app’s <tt>info.plist</tt> file.

The optional ATS key to allow insecure (HTTP) content follows:

```
<key>NSAppTransportSecurity</key>
<dict>
    <key>NSExceptionDomains</key>
    <dict>
        <key>pvoc-anaina.com</key>
        <dict>
            <key>NSIncludesSubdomains</key>
            <true/>
            <key>NSExceptionAllowsInsecureHTTPLoads</key>
            <true/>
        </dict>
    </dict>
</dict>
```

## License key
During initialization, the SDK will attempt to register using the license key. This key is linked to a content catalog, generation of usage statistics, and other SDK capabilities. In this guide, we are discussing the most common way of providing license key in your app. The other options of providing the license key in your app are discussed in the integration guide.

1. Navigate to the <tt>Info.plist</tt> for the app.
2. Create a new dictionary <tt>"vocsdk"</tt> within a  <tt>"com.akamai"</tt> dictionary. If SDK finds <tt>"vocsdk"</tt> dictionary, it will be parsed for the <tt>"license"</tt> key, and if found, the value associated with that dictionary entry will be used as the license key for registration purposes.
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>CFBundleDevelopmentRegion</key>
	<string>en</string>
	<key>CFBundleDisplayName</key>
	<string>PCDSdkExample</string>

	<!-- ... other Info.plist keys omitted ... -->

	<key>com.akamai</key>
	<dict>
            <key>vocsdk</key>
            <dict>
		<key>license</key>
		<string>your_license_key_goes_here</string>
            </dict>
	</dict>
</dict>
</plist>
```


## Initialization and Registration
Initialization provides the client a running instance of the SDK service. Registration is a one-time event that activates the the SDK with a particular license key. The SDK is required to be initialized and registered for enabling most of the SDK features. Hence, both operation should take place early in the app lifecycle. The SDK is initialized and registered by the ```VocService```Factory call ```createServiceWithDelegate:delegateQueue:options:error:```.

The recommended place for this is in AppDelegate’s ```application:didFinishLaunchingWithOptions:```. The ```create``` call inputs a reference to the SDK delegate (see SDK Delegate) as well as a configuration options dictionary. The SDK delegate is the class you designate to respond to SDK activity.

The example below is inserted into <tt>AppDelegate.m</tt>. It initializes the SDK and tells it that AppDelegate will be the delegate to handle SDK messages.

```
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
{
    NSError *error = nil;
    self.vocService = [VocServiceFactory  createServiceWithDelegate:self
                       delegateQueue:[ NSOperationQueue  mainQueue ] options:nil
                       error:&error ];
    if (! self.vocService) {
        // error handling - could not start service
        return NO;
    }
    // app initialized
    return YES;
}

```

Once registration is successful, the SDK calls the ```didRegister:``` method on ```VocServiceDelegate```. This can be used to report or log any problems starting the SDK. If registration fails due to a reason other than invalid credential, it will be retried a) On every 10 mins b) On network is available c) On app relaunch.

## SDK Events

The SDK notifies your app of various events throughout its life cycle. Messages are sent asynchronously to an SDK delegate in your code that implements the ```VocServiceDelegate``` protocol. All of the delegate methods are optional.

```
- (void) vocService :(nonnull VocService *) vocService didBecomeNotRegistered :(nonnull NSDictionary *) info;
- (void) vocService :(nonnull VocService *) vocService didFailToRegister :(nonnull NSError *) error;
- (void) vocService :(nonnull VocService *) vocService didRegister :(nonnull NSDictionary *) info;
- (void) vocService :(nonnull VocService *) vocService itemsDiscovered :(nonnull NSArray *) items;
- (void) vocService :(nonnull VocService *) vocService itemsStartDownloading :(nonnull NSArray *) items;
- (void) vocService :(nonnull VocService *) vocService itemsDownloaded :(nonnull NSArray *) items;
- (void) vocService :(nonnull VocService *) vocService itemsEvicted :(nonnull NSArray *) items;
- (void) vocService :(nonnull VocService *) vocService hlsServerStarted:(nonnull NSURL*)url;
- (void) vocService :(nonnull VocService *) vocService hlsServerStopped:(nonnull NSURL*)url;
```

The SDK delegate is passed to the SDK in the initialization call. Typically, this is the app delegate since its lifetime will span that of the SDK, from registration until shutdown. Define your app delegate as follows to implement the SDK delegate protocol.

```
@interface AppDelegate : UIResponder < UIApplicationDelegate, VocServiceDelegate>
```


## Enabing Background Execution
The PCD SDK downloads content while the application is running. Various factors determine when to start downloading, how much to download, and when to pause downloads. Influencing factors include the state of the mobile network and the quality state of the provider network.

When your app is in the foreground, downloads are happening without any need for changes in your code. However, in order to get best results, the PCD SDK should be able to download when your app is not in the foreground. In order to do that, you need to enable your app for background execution.

There are two types of background execution that the SDK uses to download content. Enable these two modes within the Xcode project settings, **Capabilities** tab, and **Background Modes** section:

- Remote notifications (remote-notification)
- Background fetch (fetch)

To enable background fetch in the PCD SDK, you need to implement the system method ```UIApplicationDelegate application:performFetchWithCompletionHandler:```
and, from there, pass the message to the ```VocService``` by calling ```VocService application:performFetchWithCompletionHandler:```

Here is what that looks like:

```
- (void)application:(UIApplication *)application performFetchWithCompletionHandler:(void (^)(UIBackgroundFetchResult result))completionHandler
{
    if (![ self.vocService application:application performFetchWithCompletionHandler:completionHandler ]) {
        completionHandler(UIBackgroundFetchResultNoData);
    }
}

```

## Enabling Remote Notifications
To make remote notifications work in the PCD SDK you need to enable the SDK backend to send push notifications to your app, and make the supporting code changes.

To enable the PCD backend to send push notifications to your app, you need to upload your app push certificate (APNS) to the PCD license management portal. The PCD backend must have a valid APNS certificate for your app at all times otherwise push notifications will not work. If you revoke or renew your certificate, make sure you upload it to the PCD license management portal. Instructions on how to generate an APNS push certificate are available on Apple’s web site:
https://developer.apple.com/library/ios/documentation/IDEs/Conceptual/AppDistributionGuide/AddingCapabilities/AddingCapabilities.html#//apple_ref/doc/uid/TP40012582-CH26-SW11

The first step for your app is to register for remote notification and hand the push token to the SDK in the ```AppDelegate``` call  ```application:didRegisterForRemoteNotificationsWithDeviceToken:```.

```
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary
                                                                                *)launchOptions
{
    // the client must register for remote notifications
    [[UIApplication sharedApplication] registerForRemoteNotifications];
    // the client must call this method during launch cycle
    // to request permission from user to use notification
    UIUserNotificationType notificationType = UIUserNotificationTypeAlert;
    [[UIApplication sharedApplication] registerUserNotificationSettings:
     [UIUserNotificationSettings settingsForTypes:notificationType categories:nil]];
}

- (void)application:(UIApplication *)application
    didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken
{
    // pass push token to SDK
    [self.vocService setDevicePushToken:deviceToken];
}

```
Also in your application delegate, implement the system method: ```UIApplicationDelegate application:didReceiveRemoteNotification:fetchCompletionHandler:``` and pass this notification to the ```VocService:```

```
VocService application:didReceiveRemoteNotification:fetchCompletionHandler:
```
For example:

```
- (void)application:(UIApplication *)application didReceiveRemoteNotification:(NSDictionary *)userInfo fetchCompletionHandler:(void (^)(UIBackgroundFetchResult result))completionHandler
{
    if ([ self.vocService application:application didReceiveRemoteNotification:userInfo fetchCompletionHandler:completionHandler ]) {
        // remote notification was for VoC SDK
        return;
    }
    // remote notification is not for VoC SDK, handle remote notification
    completionHandler(UIBackgroundFetchResultNoData);
}
```

If the client is testing against the development APNS instead of production APNS, the SDK needs to be configured to use development APNS before the SDK registration.

```
vocService.config.useProductionApns = NO;
```

Remote notification and fetch are enabled in background by adding the following key-subkey to the app’s <tt>info.plist</tt> file:

```
<key>UIBackgroundModes</key>
<array>
   <string>fetch</string>
   <string>remote-notification</string>
</array>
```


# Download to Go

The PCD SDK provides a number of ways to download content and <tt>Download to Go(D2G)</tt> is one such approach. D2G use case allows users to browse your video catalog and download one or more selections on to their mobile device. D2G approach is taken when the content downloaded is decided by the user and the SDK predictive caching is disabled. On registration of the SDK, the content metadata is not available locally. Each content is identified by a unique <tt>Content Id</tt>. The client can request content information or download a content based on this <tt>Content Id</tt>.

## Setting up D2G

Before you can use D2G, you need to ingest your video catalog to the PCD Server. Ingesting content isn't a self-service operation at this time, so talk to your Akamai representative about ingesting content. Each content( example a video) ingested is represented by a unique <tt>Content Id</tt>.

## Configure SDK for D2G
VocService has a property, ```‘config,’``` which is an instance of <tt>VocConfig.h</tt> and provides a rich set of API to configure the SDK behavior. Default values for some of the configurations in <tt>VocConfig.h</tt> are set at the global level (set to server values) and you could override those at the client level. You should discuss with your Akamai representative about the default value preferences for the configurations such as permitted network types for download (wifi, cellular, or both), content items per category, daily download size limit, maximum size allowed per file download, permitted time of day for downloading, etc. **Some of the mandatory configurations for D2G that needs to be set at the client level are the following. The client should setup these configurations right after the SDK initialization/registration.**

### Item Download Behavior
Most common D2G download behavior is to download no content unless requested explicitly by the user. The default behavior of the SDK is to preposition all the content from the content catalog. VocConfig has a property, ```itemDownloadBehavior```, which provides the client an option to define the download behavior of the PCD SDK. For D2G use case, the behavior of the SDK needs to be set as no download(```VOCItemDownloadNone```). The other SDK download behaviors are auto download(```VOCItemDownloadFullAuto```) and download thumbnail files only(```VOCItemDownloadThumbnailOnly```).
```
self.vocService.config.itemDownloadBehavior = VOCItemDownloadNone ;
```
### Auto Purge
The deletion of content in case of D2G is handled by the user. The default behavior of the SDK is to perform content purge based on certain device specific parameters. ```VocConfig``` has a property, ```enableAutoPurge```, which provides the client an option to disable auto purge and the user will be responsible for the deletion of the content.
```
self.vocService.config.enableAutoPurge = NO ;
```

### Maximum File Size
The D2G use cases generally have content that has large file size. The default max file size for which the SDK is allowed to download is 104 MB. ```VocConfig``` has a property, ```fileSizeMax```, which provides the client an option to increase the max file size.
```
self.vocService.config.fileSizeMax = 10*1024*1024*1024 ;//(10 GB)
```


## Initiating a Download

### Download Option 1:
There are multiple ways you can initiate a content download. The simplest of all is to use ```VocService::downloadItemsWithContentIds:provider:options:```.
* To use this API, the client needs to provide the following information
   * contentIds - A list of content ids that needs to be downloaded.
   * provider - The content provider.
   * options -  The options are optional as they hold a default value.
     * contentType - The type of content requested (Refer VOCItemTypeEnum). The default value is VOCItemTypeVideo. If the request content is an HLS video, the optional value must be provided as VOCItemTypeHLSVideo.
     * downloadBehavior - The download behavior for the content. Whether SDK needs to download the content completely (VOCItemDownloadFullAuto) or just the thumbnail (VOCItemDownloadThumbnailOnly). Default behavior is to follow the SDK download behavior set on VocConfig::itemDownloadBehavior.
     * quality - The quality requested for downloading. The default value is VOCQualitySelectionSdkBehavior.
		 * videoPartialDownloadLength - How much of an HLS video to download. Valid values are 0 to 32767 seconds. 0 meaning download all and is the default value.

WorkFlow
* Call on this API readily provides the client an item even before content metadata is fetched from PCD Server. The state of the item will be then "VOCItemMetadataNotReceived".
* The SDK will fetch and receives the content metadata. The item state will be "VOCItemDiscovered". If the fetch fails due to a server or device state, three retry attempts will be made.
* The SDK will start downloading the item. The item state moves to VOCItemQueued and VOCItemDownloading. If there is a device policy restricting the download, the state of the item moves to VOCItemIdle.
* The SDK retires the download for three more times if each download attempt results into a permanent failure. After third failed attempt, the item will be marked as "VOCItemFailed". The client can access the download error code from the item.
```
[appDelegate().vocService downloadItemsWithContentIds:@[@"contentId_1, contentId_2"]
provider:@"content_provider"
options:@{
				@"contentType":@(VOCItemTypeHLSVideo)
				@"downloadBehavior":@(VOCItemDownloadFullAuto),
				@"quality":@(VOCQualitySelectionMedium) }
completion:^(NSError*_Nullable error) {
					if (error) {
						//do something
					}
		}];
```
### Download Option 2:
1. The client can access the item using the API - ```VocService::getItemsWithContentIds: sourceName:completion:```.
2. If the items that match the ```contentIds``` and ```sourceName``` exists locally, the completion block receives a ```VocItemSet``` instance with those items. ```VocItemSet``` has an array property -”items” which includes those items.
3. If the items that match the ```contentId``` and ```sourceName``` don't exist locally, a server lookup is performed to get the metadata. When the lookup is successful, the items are available locally and ```VocItemSet::items``` holds those new items. On any error, the server fetch will be retried 3 more times after each 10 seconds.
4. The client receives the ```VocItemSet::items``` in the completion block of the API.
5. Perform download on the received item list using the API - ```VocService::downloadItems:options:```.

```

- (void)downloadContent
{
    NSSet *contentIds = [[NSSet alloc] initWithObjects:@"<ingested content id>", nil];

    [appDelegate().vocService getItemsWithContentIds:contentIds
     sourceName:@"ProviderName" completion:^(NSError *error, id<VocItemSet>  itemSet)
    {
        if (error) {
            // If the item is not found locally, it will report a not found error
            if ([error.domain isEqualToString:VOCSDKErrorDomain] && error.code == VocErrItemNotFound) {
                NSSet<NSString *> *missing = error.userInfo[@"notfound"];
            }
        }

				//download
				[appDelegate.vocService downloadItems:itemSet.items
                              options:@{@"downloadBehavior" : @(VOCItemDownloadFullAuto)}
															completion:^(NSError * _Nullable error) {
															}];
    }];
}

```
# Pause and Resume Downloading a Specific Item

The client can pause a specific content item download by making the ```VocService``` call  ```pauseItemDownload:``` . ```VocItem``` has a readonly property, ```paused``` which the client can use to identify items that are paused. Alternatively, the same pause operation can be performed on one or more ```VocItems``` by making the ```VocService``` call ```pauseDownloadForItems:completion:```. A paused download will not resume until a specific download request has been made.


```[appDelegate.vocService pauseItemDownload :vocItemVideo  completion :^(NSError*  _Nullable  error) {}]; ```


The client can resume the download of an item or items by making the ```VocService``` call ```downloadItems:```.


```[appDelegate.vocService downloadItems:@[vocItemVideo]  options:nil completion:^(NSError *_Nullable error) {}];```


## Content Item States
Each item progresses through different states before getting cached. VocItem has a  ```state``` property which can take the following values.


|    VOCItemState    |                                                                                                                                           Description                                                                                                                                           |
|:----------------------------:|:-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------:|
| VOCItemMetadataNotReceived   | Item place holder has been created locally. Waiting for metadata update from Server. |
| VOCItemDiscovered  				   | Item has been identified. Various device policies dictate whether the item gets queued for download.                                                                                                                                                                                            |
| VOCItemQueued   				     | Item has been added to downloading queue; it is currently not downloading. It will start downloading.                                                                                                                                                                                           |
| VOCItemDownloading					 | Item is being downloaded.                                                                                                                                                                                                                                                                       |
| VOCItemIdle        					 | Item is currently not downloading. Item has been previously queued for downloading but did not finish. Download will resume automatically at some point. Reason for stopping download can be found in ```VocItem downloadError```. |
| VOCItemPaused     					 | Item is currently not downloading. It has been explicitly paused with ```VocService pauseItems```                                                                                                                                                                                                    |
| VOCItemCached     					 | Item was downloaded successfully. Cached content is ready for use. |
| VOCItemPartiallyCached 			 | Item is configured for partial download. Requested part was download successfully. Cached content is ready for use.  |
| VOCItemFailed     					 | If PCD SDK is unsuccessful to download any main content file after a number of attempts, the item will be marked as failed and the downloaded content will be removed. No more download attempts will be made on the item, unless an explicit download for this item is triggered again. ```VocItem downloadError```, can be used to verify the reason for download failure, error codes are provided in the table below. |
| VOCItemDeleted     					 | Item has been deleted and cannot be used any more. There are no cached files.                                                                                                                                                                                                                   |

### Download Failure and Idle Error Codes

|    Error code    |                                                                                                                                           Description                                                                                                                                           |
|:------------------:|:-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------:|
| VOCErrContentIdDoesNotExistInServer | Item requested for download does not have metadata available in PCD Server. |
| VocErrDownloadNoNetworkError| Network is not available for download. |
| VocErrTimeOut|  Download was cancelled because it did not finish in the allocated time. |
| VocErrDownloadNetworkError      | Download failed due to invalid http response. |
| VocErrDownloadHttpError | Download failed due to other http errors. |
| VocErrDownloadItemCorrupted | Downloaded file cannot be parsed or cannot be accessed from/stored in the file system.  |
| VocErrCancelled | File download had to be interrupted because of item deletion, or no more background time or download order optimizations. |
| VocErrDownloadFailed      | After a number of attempts the file cannot be downloaded due to problems that are not related to network connection like permissions, missing file, wrong url, server not responding, etc.|
| VOCErrDownloadCacheLimit | Error code indicating no space left in cache to download file. |
| VOCErrDownloadDailyLimit | Error code indication download cannot be proceeded since SDK has exceeded the daily download limit. |
| VOCErrDownloadOutOfPolicy| Download gets cancelled when device is out of policy |




# Accessing Cached Content

## Playing HLS Video Content
HTTP Live Streaming (HLS) video can be played from fully- or partially-downloaded content. Playback requires starting the SDK’s on-device HLS server. From that server, a playback link is then requested for the content.

The client starts the HLS server by calling ```VocService::startHLSServerWithCompletion:completion``` before playing the HLS content. The completion block is executed only after the HLS server is started. In the passed completion block, the client can access the HLS server URL and append to it the content item’s relative URL. Feed this URL to the media player to play the HLS content. Once playback is finished, stop the media server by calling ```VocService::stopHLSServer```. The client controls the starting and stopping of the HLS server.

```
id<VocItemHLSVideo> hlsVideo = ...;

if (hlsVideo.state == VOCItemCached || hlsVideo.state == VOCItemPartiallyCached)  { //play from local cache
	[appDelegate.vocService startHLSServerWithCompletion:^(BOOL success) { //start the local HLS server
	    if (success) {
				//form local server video url
	        NSURL *url = [appDelegate.vocService.hlsServerUrl URLByAppendingPathComponent:hlsVideo.hlsServerRelativePath];
	        [self playVideo:url];
	    }
	}];
} else {
	[self playVideo:hlsVideo.file.url];//play from remote url
}
```

## Playing Cached Non-HLS Video
Clients can play content from a URL if the item is not yet cached, or from the local path if the item is cached.

To play a mp4 prepositioned video:

```
id<VocItemVideo> video =  ...;
NSURL   *url = video.file.url; //remote url
if (video.state == VOCItemCached || video.state == VOCItemPartiallyCached)  {
    url = [NSURL fileURLWithPath: video.localPath]; //local path
}

//Set the media player's content URL to this URL
[self playVideo:url];
```

## Accessing content items with/without Filters

The client can access items and videos based on filters. Below are the APIs used to access content based on Filter
| Object Type | Class | API |
|-------------|-----------------|-------------------|
| VocItem | VocDataQuery.h | getItemsWithFilter:completion: |
| VocItemVideo | VocDataQuery.h | getVideosWithFilter:completion: |

Also, the client can access items based on URL, unique ID and Content IDs.

| Object Type | Class | API |
|-------------|-----------------|-------------------|
| VocItem | VocDataQuery.h | getItemWithId:completion:  getItemWithURL:completion: getItemsWithContentIds:sourceName:completion: |

In the example below, we used the “itemsInStates:sort:” filter to get a specific set of items whose state is <tt>VOCItemCached</tt>. Specific subsets can also be created by using alternative filters. These sets will behave as before, but will notify their listeners only when the filter is matched.

| Object Type | Filter Protocol | Available Filters |
|-------------|-----------------|-------------------|
| VocItem | VocItemFilter | allWithSort:	itemsInStates:sort: itemsWithContentIds:sourceName:sortDescriptors: itemsPartiallyDownloadedWithSort: itemsDownloadingWithSort:  itemsFailedWithSort: itemsPausedWithSort: itemsIdleWithSort: itemsQueuedWithSort: itemsDiscoveredWithSort: itemsDownloadedWithSort: itemsWithCategory:sortDescriptors:  itemsDownloadedWithCategory:sortDescriptors:  itemsSavedWithSort: itemsDownloadedNotViewedWithSort  itemsInStates:withCategory:sortDescriptors: itemsInStates:notViewedWithSort:|
| VocItemVideo (subset of Item) | VocItemVideoFilter | allWithSort: videosInStates:sort: videosDownloadingWithSort:  videosWithCategory:sortDescriptors:  videosDownloadedWithCategory:sortDescriptors:  videosDownloadedOrDownloadingWithCategory:sortDescriptors:  videosSavedWithSort: videosDownloadedNotViewedWithSort: |

Most filters also accept an array of sort descriptors. The sort descriptor defines the order of those arrays. Pass a nil descriptor if order is unimportant.


```
NSArray *sortDesc =   @[[[ NSSortDescriptor  alloc ] initWithKey:@"name"  ascending:YES selector:@selector(caseInsensitiveCompare:)]];

[ vocService getItemsWithFilter:[ VocItemSourceFilter itemsInStates:[NSSet setWithArray:@[@(VOCItemCached)]] sort:sortDesc ] completion:^(NSError *__nullable error,  id < VocSourceSet >  __nullable set) {
    // set.sources is ordered by each source's "name" property
}];
```

## Listening for Data Set Changes

Data sets send notifications when items are added, changed, or removed. This is the best way to detect when individual items have finished downloading. To listen for object set changes a class must subscribe to the  ```VocObjSetChangeListener``` protocol.

``` @interface MyClass() <VocObjSetChangeListener> ```

Setting the data set listener is done via ```addListener:``` and is demonstrated in the sections below for items. Listeners receive callbacks before and after any changes.

```
- (void) getContent
{
    [ vocService getVideosWithFilter:[ VocItemSourceFilter allWithSort:sortDesc] completion:^(NSError *__nullable error,  id < VocSourceSet >  __nullable set) {
        self.itemSet = itemSet;
        [self.itemSet addListener:self];
    }];
}

-(void) vocService:(nonnull id < VocService >)vocService objSetWillChange:(nonnull id < VocObjSet >) objSet added:(nonnull NSSet *) added updated:(nonnull NSSet *)updated removed:(nonnull NSSet *) removed objectsAfterChanges:(nonnull NSArray *)objectsAfterChanges
{
    if ([ objSet isEqual:self.itemSet ]) {
        NSLog(@"%ld items will change",   (unsigned long)updated.count);
    }
} 

(void)vocService:(nonnull id < VocService >)vocService objSetDidChange:(nonnull id < VocObjSet >)objSet
             added:(nonnull NSSet *)added updated:(nonnull NSSet *)updated removed:(nonnull NSSet *)removed
     objectsBefore:(nonnull NSArray *)objectsBefore
{
    if ([ objSet isEqual:self.itemSet ]) {
        NSLog(@"%ld items have changed",  (unsigned long)updated.count);
    }
}

```

## Track Progress of a Downloading Video

There are two properties in VocItem that will allow the client app to calculate download progresss.

- ```bytesDownloaded``` - total number of bytes downloaded for the item.
- ```size``` - total size of the item.

The client app can listen for ```bytesDownloaded``` changes by subscribing to the ```VocObjSetChangeListener``` protocol (refer to Listening for Data Set Changes) or using key-value observing (KVO). Alternatively, the client app can use a timer to periodically poll ```bytesDownloaded``` and update the progress.

## Delete and Lock

The SDK provides two delete options for each item:

- ```VocItem::deleteItem:``` deletes both the metadata and files associated with the item.
- ```VocItem::deleteFiles:``` deletes just the files associated with the item. Since the metadata is not removed, the client can re-download the video at any point after deleting the files. Alternatively, the same operation can be performed on one or more VocItems by making the VocService call ```deleteFilesForItems:options:completion:```. Optionally, the app can retain the video’s thumbnail by setting ```deleteThumbnail``` parameter to ```NO```.

**Note:** Some content consumers exhibit unexpected behavior if the item gets deleted while it is consumed. The SDK doesn’t know whether the item is currently consumed unless it is notified explicitly. It is recommended to call ```VocItem::lock``` before the item gets consumed and ```VocItem::unlock``` after the consumption is completed.

## Content Sources
A content source identifies the provider (e.g., a channel, company, or aggregator) of content items. Sources are defined on the SDK Web portal. The SDK allows sources to be toggled on or off.

To track changes to a source set, subscribe your interface to the ```VocObjSetChangeListener``` protocol. Also define a property to reference the source set.

```
@interface   MyViewController () <VocObjSetChangeListener>
@property (strong, nonatomic)  id<VocSourceSet>  vocItemSourceSet;
@end
```
Create the source list in your implementation. This needs to be called only once per set, such as in -viewDidLoad for a view controller that uses sources.

```
[ vocService getSourcesWithFilter:[ VocItemSourceFilter allWithSort:nil ]
  completion:^(NSError *__nullable error,  id < VocSourceSet >  __nullable sourceSet) {
    if (error) {
        NSLog(@"Error getting sources: %@",  error);
        return;
    }
    self.vocItemSourceSet =  sourceSet;
    [ sourceSet addListener:self ];
}];
```
This will notify the listener when the set changes. You can also access individual sources by stepping through the source container, ```vocItemSourceSet.sources```.

```
for (id < VocItemSource >  source  in   self.vocItemSourceSet.sources) {
    if (source.subscribed) {
        NSLog(@"Subscribed to %@",  source.displayName);
    }
}
```

## Content Categories

Categories are named groups of related content, such as “family,” “trending,” or “news.” Working with a category makes it easy to display dedicated views for broad content types, or to toggle the display of an entire group.

To create and follow a category set, subscribe your interface to the ```VocObjSetChangeListener``` protocol (the same as for sources). Also define a property to reference the category set.

```
@interface   MyViewController() <VocObjSetChangeListener>
@property (strong,  nonatomic)  id<VocCategorySet> vocItemCategorySet;
@end
```

Create the category list in your implementation. This needs to be called only once per category set, such as in a view controller’s  -```viewDidLoad``` method. As shown for sources, this example adds our self object as a listener and retains a reference to the category set.

```
[ vocService getCategoriesWithFilter:[ VocItemCategoryFilter allWithSort:nil ]
  completion:^(NSError *__nullable error,  id < VocCategorySet >  __nullable categorySet) {
    if (error) {
        NSLog(@"Error getting categories: %@",  error);
        return;
    }

    self.vocItemCategorySet = categorySet;
    [ categorySet addListener:self ];
}];
```

Categories may be iterated over by using ```vocItemCategorySet.categories```.

```
for (id < VocItemCategory >  category  in   self.vocItemCategorySet.categories) {
    if (category.subscribed) {
        NSLog(@"Subscribed category %@",  category.displayName);
    }
}
```

## Source and Category Filters

| Object Type | Filter Protocol | Available Filters |
|-------------|-----------------|-------------------|
| Source | VocSourceFilter | allWithSort: |
| Category | VocItemCategoryFilter | allWithSort:     categoriesSelected: categoriesSelectedWithContent:  subCategories: SearchField:  suggestions: |

```
NSArray *sortDesc =   @[[[ NSSortDescriptor  alloc ] initWithKey:@"name"  ascending:YES selector:@selector(caseInsensitiveCompare:)]];

[ vocService getSourcesWithFilter:[ VocItemSourceFilter allWithSort:sortDesc ] completion:^(NSError *__nullable error,  id < VocSourceSet >  __nullable set) {
    // set.sources is ordered by each source's "name" property
}];
```


## Record Video Consumption by the User

When a user watches a video partially or fully, the client reports the consumption to the PCD SDK.

```
id<VocItemVideo> video =  ...;

// When user navigates back from the video player
// The total watched duration of the video
NSTimeInterval end = CMTimeGetSeconds(player.currentItem.currentTime);

// If playback did not start at the beginning put the position in start.
NSTimeInterval start = 0;
[video recordConsumption:[NSDate date] startingAt:start endingAt:end];
```

# Optional SDK Settings

## Network Preference
The API provides control over whether downloads are permitted on wifi, cellular, or both. Setting the ```VocConfig networkSelection``` property affects downloads that have not already started.

```
self.vocService.config.networkSelection = VOCNetworkSelectionWifiAndCellular;
```

## Maximum Number of Concurrent Downloads
The default behavior of the SDK is to perform any number of concurrent file downloads. ```VocConfig``` has a property, ```maxConcurrentDownloads```, which provides  the client an option to restrict the number of simultaneous downloads.
```
self.vocService.config.maxConcurrentDownloads = 2 ;
```
