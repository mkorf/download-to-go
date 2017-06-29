# Accessing Content

Content begins loading onto the device once the client has initiated a download. Downloading also happens automatically in background.

## Playing HLS Video Content
HTTP Live Streaming (HLS) video can be played from fully- or partially-downloaded content. Playback requires starting the SDK’s on-device HLS server. From that server, a playback link is then requested for the content.

The client starts the HLS server by calling ```VocService::startHLSServerWithCompletion:completion``` before playing the HLS content. The completion block is executed only after the HLS server is started. In the passed completion block, the client can access the HLS server URL and append to it the content item’s relative URL. Feed this URL to the media player to play the HLS content. Once playback is finished, stop the media server by calling ```VocService::stopHLSServer```. The client controls the starting and stopping of the HLS server.

```
id<VocItemHLSVideo> hlsVideo = ...;

[appDelegate.vocService startHLSServerWithCompletion:^(BOOL success) {
    if (success) {
        NSURL *url = [appDelegate.vocService.hlsServerUrl URLByAppendingPathComponent:hlsVideo.hlsServerRelativePath];

        [self playVideo:url];
    }
}];
```

## Playing Cached Non-HLS Video
Clients can play content from a URL if the item is not yet cached, or from the local path if the item is cached.

To play a mp4 prepositioned video:

```
id<VocItemVideo> video =  ...;
NSURL   *url = video.file.url;
if (video.state == VOCItemCached || video.state == VOCItemPartiallyCached)  {
    url = [NSURL fileURLWithPath: video.localPath];
}

//Set the media player's content URL to this URL
[self playVideo:url];
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

## Content Item States
Each item progresses through different states before getting cached. VocItem has a  ```state``` property which can take the following values.


|    VOCItemState    |                                                                                                                                           Description                                                                                                                                           |
|:------------------:|:-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------:|
| VOCItemDiscovered  | Item has been identified. Various device policies dictate whether the item gets queued for download.                                                                                                                                                                                            |
| VOCItemQueued      | Item has been added to downloading queue; it is currently not downloading. It will start downloading.                                                                                                                                                                                           |
| VOCItemDownloading | Item is being downloaded.                                                                                                                                                                                                                                                                       |
| VOCItemIdle        | Item is currently not downloading. Item has been previously queued for downloading but did not finish. Download will resume automatically at some point. Reason for stopping download can be found in ```VocItem downloadError```. |
| VOCItemPaused      | Item is currently not downloading. It has been explicitly paused with ```VocService pauseItems```                                                                                                                                                                                                    |
| VOCItemCached      | Item was downloaded successfully. Cached content is ready for use. |
| VOCItemPartiallyCached | Item is configured for partial download. Requested part was download successfully. Cached content is ready for use.  |
| VOCItemFailed      | If PCD SDK is unsuccessful to download any main content file after a number of attempts, the item will be marked as failed and the downloaded content will be removed. No more download attempts will be made on the item, unless an explicit download for this item is triggered again. ```VocItem downloadError```, can be used to verify the reason for download failure, error codes are provided in the table below. |
| VOCItemDeleted     | Item has been deleted and cannot be used any more. There are no cached files.                                                                                                                                                                                                                   |

### Download Failure and Idle Error Codes

|    Error code    |                                                                                                                                           Description                                                                                                                                           |
|:------------------:|:-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------:|
| VocErrDownloadFailed      | After a number of attempts the file cannot be downloaded due to problems that are not related to network connection like permissions, missing file, wrong url, server not responding, etc.
| VocErrDownloadNetworkError      | Network connection error, download will resume automatically. |
| VocErrCancelled | File download had to be interrupted because of PCD policy, or no more background time or download order optimizations. |
| VOCErrDownloadCacheLimit      | Error code indicating no space left in cache to download file. |

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

## Content Filters

Each of the set-building methods accepts a filter that defines the set. In each example above we used the “all” filter to get a complete set of items. Specific subsets can also be created by using alternative filters. These sets will behave as before, but will notify their listeners only when the filter is matched.

| Object Type | Filter Protocol | Available Filters |
|-------------|-----------------|-------------------|
| Source | VocSourceFilter | allWithSort: |
| Category | VocItemCategoryFilter | allWithSort: categoriesSelected: categoriesSelectedWithContent:  subCategories: SearchField:  suggestions: |
| Item | VocItemFilter | allWithSort:  itemsDownloadingWithSort:  itemsWithCategory:sortDescriptors:  itemsDownloadedWithCategory:sortDescriptors:  itemsSavedWithSort: itemsDownloadedNotViewedWithSort |
| Video (subset of Item) | VocItemVideoFilter | allWithSort:  videosDownloadingWithSort:  videosWithCategory:sortDescriptors:  videosDownloadedWithCategory:sortDescriptors:  videosDownloadedOrDownloadingWithCategory:sortDescriptors:  videosSavedWithSort: videosDownloadedNotViewedWithSort: |

Most filters also accept an array of sort descriptors. The ```.sources```, ```.categories```, and ```.items``` properties within their respective sets are each an ordered array. The sort descriptor defines the order of those arrays. Pass a nil descriptor if order is unimportant.


```
NSArray *sortDesc =   @[[[ NSSortDescriptor  alloc ] initWithKey:@"name"  ascending:YES selector:@selector(caseInsensitiveCompare:)]];

[ vocService getSourcesWithFilter:[ VocItemSourceFilter allWithSort:sortDesc ] completion:^(NSError *__nullable error,  id < VocSourceSet >  __nullable set) {
    // set.sources is ordered by each source's "name" property
}];
```
