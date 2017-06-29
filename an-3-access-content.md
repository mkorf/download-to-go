# Accessing Content

Content begins loading onto the device once the client has initiated a download. Downloading also happens automatically based on background GCM notifications the SDK receives.

The client app can use ```AnaContentProvider``` to manage  its lists of cached content based on categories and providers. ```AnaContentProvider``` can be accessed using the public contract provided by ```AnaProviderContract```, which supports a cursor/adapter framework for CRUD operations and notifications. 

An additional Model API - ```AnaFeedItem, AnaFeedCategory, AnaContentSource``` are provided to be used as data models. See javadocs provided for the API reference

<table>
  <tr>
    <th>Data Object</th>
    <th><b>Description</th>
  </tr>
  <tr>
    <td><tt>AnaFeedItem</tt></td>
    <td>A single video, song, HTML page, image, etc.</td>
  </tr>
  <tr>
    <td><tt>AnaContentSource</tt></td>
    <td>Content providers configured at the Web portal.  Every content item belongs to a source.</td>
  </tr>
  <tr>
    <td><tt>AnaFeedCategory</tt></td>
    <td>A named group of related content items (e.g., "Sports").</td>
  </tr>
</table>

For example 
```
// Get a list of all content metadata - client would use 
 Uri uri  = AnaProviderContract.CONTENT_URI_FEEDS.buildUpon().build();
 Cursor cursor = context.getContentResolver().query( uri,
 null, null, null, null);
 ArrayList<AnaFeedItem> items = new ArrayList<AnaFeedItem>()
 //See AnaFeedItem for the properties exposed
if(cursor != null){
   cursor.moveToFirst();
   while (!cursor.isAfterLast()) {
       AnaFeedItem feedItem = new AnaFeedItem(cursor);
       items.add(feedItem);
       cursor.moveToNext();
   }
   cursor.close();
}
```

To show content in a list activity:
```
public class MyVideoListActivity extends Activity implements LoaderManager.LoaderCallbacks<Cursor>{

  .....
  
   @Override
    public Loader<Cursor> onCreateLoader(int i, Bundle bundle) {
	Uri.Builder builder = AnaProviderContract.CONTENT_URI_FEED_DOWNLOAD_STATE.buildUpon();
	Uri uri = builder.build();
	CursorLoader loader = new CursorLoader(getApplicationContext(), uri,
	    null, null, null, AnaProviderContract.FeedItem.MARK_FOR_DOWNLOAD_TIMESTAMP + " DESC");
        return loader;
    }

....
}

public class MyFeedItemAdapter extends RecyclerView.Adapter<MyFeedItemViewHolder>{
.....

  @Override
    public void onBindViewHolder(MyFeedItemViewHolder holder, int position) {
	if (mCursor.moveToPosition(position)) {
		final AnaFeedItem feedItem = new AnaFeedItem(mCursor);
		//Use AnaFeedItem properties using getters to paint the list
		if (!feedItem.isResourceReady()) {
		    //content is not available locally
		    playOverlay.setBackgroundColor(Color.GRAY);
	        } else {
		   //Enable play button
        	}		
        }
        ....	
	
   }
   
   
}
  
```

## Playing HLS Video Content

HTTP Live Streaming (HLS) video can be played from fully- or partially-prepositioned content.  

Playback requires starting the HLS server using ```VocService.startMediaServer()```. This starts media server on a random port to serve cached HLS segments. Once the playback is finished, stop the media server by calling ```VocService.stopMediaServer()```.  The client controls the starting and stopping of the media server. 

```VocUtils``` is a utility class that helps client to get media/data/resource path. Once the media server is started, client must call ```VocUtils.getResourcePath()``` to get appropriate HTTP URL to play locally cached HLS content. Note – The path returned by ```VocUtils.getResourcePath``` becomes invalid once the media server is stopped. 

```
// To play locally cached HLS content 
// Example in client VocPlayerMainActivity and VocVideoListActivity
AnaFeedItem feedItem = VocUtils.getAnaFeedItemFromContentId(getApplicationContext(), "<Your content Id>");
if (feedItem != null && feedItem.getType().equals(AnaFeedItem.CONTENT_TYPE_HLS)) {
    // Start media server to serve hls content over http.
    VocServiceResult result = VocService.createVocService(getApplicationContext()).startMediaServer();
    if (result.isSuccess()) {
	// Get the local URL to pass on to media player to play cached content.
	String cachedFeedUrl = VocUtils.getResourcePath(feedItem, getApplicationContext());
	// For more detail info - see the sample app 
	mMediaPlayer.setDataSource(cachedFeedUrl);
    }
}

// Stop the media server once cached HLS playback is finished.
vocService.stopMediaServer();

// And report consumption stats using 
vocService.updateFeedConsumptionStats(feedItem.getId(),  startTime,  stopPosition);
```

## Playing Non-HLS Video Content

To play downloaded content (non-HLS content) use the following:

```
AnaFeedItem feedItem = VocUtils.getAnaFeedItemFromContentId(getApplicationContext(), "<Your content Id>");
if (feedItem != null ) {
	mMediaPlayer.setDataSource( filePath );
	// Video playing  
} 

// On video play completion client can update consumption stats to the server using 
vocService.updateFeedConsumptionStats(feedItem.getId(),  startTime,  stopPosition);
```

## Record Video Consumption by User

When the user watches a video partially or fully, the client  reports the consumption to the PCD SDK.

```
/**
* Update the consumption statistics
*
* @param item      - representing the video that was played
* @param startTime - time when the video playing started
*/
private void updateConsumptionStatistics(AnaFeedItem item, long startTime) {
   VideoPlayerController videoController = getActivity().getVideoPlayerController();
   int stopPosition = videoController.getContentProgress();
   VocService.createVocService(getActivity()).updateFeedConsumptionStats(item.getId(), startTime, stopPosition);
}
```

## Content Item States 
Each item progresses through different states before getting cached. ```VocDownloadStatusReceiver``` has constants which can take the following values:


|    DownloadState    |                                                                                                                                           Description                                                                                                                                           |
|:------------------:|:-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------:|
| TYPE_DOWNLOAD_STATE_INITIATED      | Download state indicating download is initiated. This event will be called only once per feed item when it is first initiated to download. 
| TYPE_DOWNLOAD_STATE_QUEUED      | Download state indicating download is queued; it is currently not downloading. It will start downloading.                                                                                                                                                                                           |
| TYPE_DOWNLOAD_STATE_PROGRESS | Download state indicating download is in progress.                                                                                                                                                                                                                                                                       |
| TYPE_DOWNLOAD_STATE_STARTED        | Download state indicating download is started.                                                                                                                                        |
| TYPE_DOWNLOAD_STATE_PARTIAL      | Download state indicating content is partially downloaded.                                                                                                                                                                                                    |
| TYPE_DOWNLOAD_STATE_COMPLETED      | Download state indicating download is successfully completed. Cached content is ready for use.                                                                                                                                                                                                                      |
| TYPE_DOWNLOAD_STATE_FAILED      | Download state indicating download failed either because user paused it or because of some failure condition indicated by error codes. |

**Note:** When the download fails, the following error codes can be used to verify the reason for failure or download suspension.

|    Error code    |                                                                                                                                           Description                                                                                                                                           |
|:------------------:|:-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------:|
| DOWNLOAD_ERROR_CODE_INTERRUPTED      | Error code indicating that download was interrupted by user. This is same as user pausing the download. 
| DOWNLOAD_ERROR_CODE_NOT_DOWNLOADABLE      | Error code indicating that this feed doesn't allow download e.g. if the response header for the content url specifies "no-store" directive.                                                                                                                                                                                           |
| DOWNLOAD_ERROR_CODE_NETWORK_UNAVAILABLE | Error code indicating that download failed because of network connection loss.                                                                                                                                                                                                                                                                       |
| DOWNLOAD_ERROR_CODE_UNKNOWN        | Error code indicating some unknown error occurred.                                                                                                                                        |
| DOWNLOAD_ERROR_OUT_OF_POLICY      | Error code indicating PCD policy failure.                                                                                                                                                                                                    |
| DOWNLOAD_ERROR_CACHE_FULL      | Error code indicating no space left on device to download content.                                                                                                                                                                                                                      |
| DOWNLOAD_ERROR_HTTP_ERROR      | Error code indicating Http error. |
| DOWNLOAD_ERROR_DISK_SPACE_NA      | Error code indicating that the disk space for download is no longer available. Typically happens when storage is filled by some other app during download. |


## Content Sources 

A content source identifies the provider (e.g., a channel, company, or aggregator) of content items. Sources are defined on the SDK Web portal. The SDK allows sources to be toggled on or off.

```
// AnaProviderContract.CONTENT_URI_SOURCES - Provides list of content sources present   
Cursor cursor = getApplicationContext().getContentResolver().query(
	AnaProviderContract.CONTENT_URI_SOURCES,
	null, AnaProviderContract.SELECTION_SUBSCRIBED_PROVIDER, new String[]{"1"}, null);
ArrayList<AnaContentSource> sources = new ArrayList<AnaContentSource>();
if (cursor != null) {
    cursor.moveToFirst();
    while (!cursor.isAfterLast()) {
	AnaContentSource feedSource = new AnaContentSource(cursor);
	sources.add(feedSource);
	cursor.moveToNext();
    }
    cursor.close();
}
```

## Content Categories 

Categories are named groups of related content, such as "family," “trending,” or “news.” Working with a category makes it easy to display dedicated views for broad content types, or to toggle the display an entire group.

```
// AnaProviderContract.CONTENT_URI_FEEDCATEGORY - provides Categories of content (Entertainment , sports etc )
Cursor cursor = getApplicationContext().getContentResolver().query(
	AnaProviderContract.CONTENT_URI_FEEDCATEGORY,
	null, AnaProviderContract.SELECTION_SUBSCRIBED_CATEGORY, new String[]{"1"}, null);
ArrayList<AnaFeedCategory> categories = new ArrayList<>();
if (cursor != null) {
    cursor.moveToFirst();
    while (!cursor.isAfterLast()) {
	AnaFeedCategory feedCategory = new AnaFeedCategory(cursor);
	categories.add(feedCategory);
	cursor.moveToNext();
    }
    cursor.close();
}
```

## Content Filters

Additionally you can filter content by using standard content provider queries and selection mechanism based on ```AnaProviderContract```.

```
//Filter items based on Category
Uri.Builder builder;
String category = "News";
builder = AnaProviderContract.CONTENT_URI_FEEDS_FILTER.buildUpon();
builder.appendQueryParameter(AnaProviderContract.FeedItem.CATEGORIES, category);
Cursor cursor = getApplicationContext().getContentResolver().query(builder.build(),
	null, null,
	null, null);


//Filter items that are pre-positioned 
Uri.Builder builder = AnaProviderContract.CONTENT_URI_FEEDS_FILTER.buildUpon();
builder.appendQueryParameter(AnaProviderContract.FeedItem.RESOURCEREADY, "true");
Cursor cursor = getApplicationContext().getContentResolver().query(builder.build(),
	null, null,
	null, null);


//Filter items based on type example images
String selection = AnaProviderContract.FeedItem.FEEDTYPE + "=?";
String[] selectionArgs = new String[]{"image/jpeg"};
Cursor cursor = getApplicationContext().getContentResolver().query(
	Uri.parse(AnaProviderContract.CONTENT_URI_FEEDS.toString()),
	null, selection, null, null);

```
