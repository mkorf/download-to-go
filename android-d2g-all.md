# Introduction

The Predictive Content Delivery (PCD) SDK allows users to download videos intelligetntly to mobile devices. Users initiate downloads by browsing your video catalog and selecting the content to watch. 

A sophisticated Download Manager checks the battery, device storage, network, and other variables to make sure that the video transfer will succeed. Errors are thrown if the video transfer fails. 

Downloads continue whether the app is active in the foreground, running in the background, or entirely closed. In addition, the SDK acts as a data cache, providing access to content along with its current download state. 

## Audience

This document is for developers implementing the PCD SDK; deep knowledge of Android and/or iOS is assumed. 

There are separate instructions for Android and iOS. Configuring your project to support these platforms is covered in the following chapters.


# Getting Started with Android
The fastest way to get started with SDK is to try the sample app. Once you've downloaded the Zip file and changed the necessary requirements and dependencies, you're ready to go. 

After you're done playing with the sample app, the next steps are SDK initialization and registration. These steps are covered in detail in Integrating With Your Android Application. 

## Requirements and Dependencies

* Android API 9 or later.
* A PCD SDK license key is required.
* Google play services 
* [Google Cloud Messaging](http://docs.urbanairship.com/reference/glossary.html#term-google-cloud-messaging) (GCM) for Android notifications

**Required gradle dependencies:**

* ```compile 'com.google.android.gms:play-services-gcm:9.4.0'```

## SDK Contents
When you download the PCD SDK zip, the contents include: 

* A javadoc folder with documentation for API reference
* <tt>pcd-sdk-release-{version}.aar</tt> file 
* <tt>vocclient</tt> - Example voc client app 
* Integration guide

## Trying the Sample App

The easiest way to get started is to try the sample app. 

1. Open Android Studio and navigate to where teh SDK was unzipped.  
2. Import <tt>vocclient</tt>. 
3. Substitute your SDK license key for the existing key. 
4. Build/run the <tt>vocclient</tt> sample app. 


# Integrating With Your Android Application

To start using the PCD SDK with your own app, you'll need to include the SDK in your app. This is followed by initialization, and then registration. 

## Including the SDK and Updating the Client

1. Copy <tt>pcd-sdk-<i>version</i>.aar</tt> from the Zip file into the <tt>/app/libs</tt> directory in your project’s root directory.
2. Now update the client <tt>build.gradle</tt> by adding the following gradle dependencies into your application <tt>build.gradle</tt>

```compile 'com.google.android.gms:play-services-gcm:9.4.0'```
```compile(name:'pcd-sdk-release-{version}', ext:'aar')```
```useLibrary 'org.apache.http.legacy'```

### Sample <tt>build.gradle</tt>

```
android {
     compileSdkVersion 25
     buildToolsVersion '25.0.2'
     defaultConfig {
         minSdkVersion 18
         targetSdkVersion 25
         versionCode 1
         versionName "1.0"
     }
     buildTypes {
         release {
             runProguard false
             proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
         }
     }
     useLibrary 'org.apache.http.legacy'
 }

 repositories {
     mavenCentral()
     flatDir {
         dirs 'libs'
     }
 }

 dependencies {
    // Required gradle dependencies
    compile 'com.google.android.gms:play-services-gcm:9.4.0'   
    compile (name:'pcd-sdk-release-{version}', ext:'aar')
 }
```

## Updating the Client Manifest

You also need to update <tt>AndroidManifest.xml</tt> to complete the SDK integration.

```
<manifest . . . >
     <application . . . >
          . . .
          **<!-- {ApplicationId} should be replaced with the client app package name , example - com.domain.applicationname -->**
          **<!-- A receiver to listen to Voc Status messages -->**
          <receiver
               android:name="{ApplicationId}.VocBroadcastReceiver">
                    <intent-filter>
                         <action android:name="com.akamai.android.sdk.ACTION_VOC_STATUS" />
                         <category android:name="{ApplicationId}" />
                    </intent-filter>
          </receiver>

          **<!-- A provider to access cached content -->**
          <provider 
               android:name="com.akamai.android.sdk.db.AnaContentProvider"
               android:authorities="{ApplicationId}.AnaContentProvider" >
          </provider>
          . . .
     </application>
</manifest>
```

**Note:** the SDK requires these permissions for full functionality:

```
<manifest . . . >
     . . .
     <uses-permission android:name="android.permission.INTERNET" />
     <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
     <uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
     <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
     <uses-permission android:name="com.google.android.c2dm.permission.RECEIVE" />
     <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
     . . .
</manifest>
```

**Note:** The PCD SDK relies on GCM Notifications that are delivered to a GCM broadcast receiver in the SDK; if the client app has its own GCM receivers then they could also be triggered due to PCD GCM notifications.

## Initialization

After the app starts, the first step is to create an instance of ```VocService``` using ```VocService.createVocService(Context applicationContext)```, then set up the configuration using ```VocConfigBuilder```. This should be done on main application ```create``` or ```onCreate``` of the main activity.

          VocService vocService = VocService.createVocService(getApplicationContext());

For Download to Go, disable auto prefetch:

	VocConfigBuilder builder = new VocConfigBuilder(context.getApplicationContext());
        builder.disableBackgroundDownloads();
	
## Registration

The next step is to register with VOC server using ```VocService.register``` API. The register call is non-blocking, so the client needs to pass an async handler for registration status result. 

Registration is a one-time event; the client can check the registration status using ```VocService.getRegistrationStatus``` to see if its already registered. Registration needs the SDK API key. The SDK API key is provided by Akamai, and if you don't have one, contact your Akamai representative. 

For example, in the client app ```MainActivity```:

```
public class MainActivity extends Activity {
     ...
     private VocService mVocService;
     private static final String SDK_API_KEY = "Your Akamai Provided License Key";
     ...
     
     @Override
     protected void onCreate(Bundle savedInstanceState) {
          super.onCreate(savedInstanceState);
          VocService vocService = VocService.createVocService(getApplicationContext());
          VocRegistrationStatus status = mVocService.getRegistrationStatus();
          if (status.isActive()) {
               // Launch list activity to load content from ContentProvider
               ...
          } else {
	       VocConfigBuilder builder = new VocConfigBuilder(context.getApplicationContext());
	       //Disable auto prefetch Only for Download 2 Go
	       builder.disableBackgroundDownloads();
               VocRegistrationInfo regInfo = new VocRegistrationInfo(SDK_API_KEY);	   
               mVocService.register(regInfo, new VocService.VocAysncResponseHandler(new Handler()) {
                              public void onSuccess() {
                                   // Launch activity to list content
                                   ...
                              }
                         
                              public void onFailure(String message) {
                                   // Display error message 
                                   ...
                              }
                         }
                    );
          }
     }
}
```

## SDK Events

If registration succeeds, cache initialization is triggered in the background. The client listens to the cache status by registering a ```BroadCastReceiver``` with filter set to ```"com.akamai.android.sdk.ACTION_VOC_STATUS"```.

### Example in Client Manifest 

```
<receiver android:name={ApplicationId}.MyVocBroadcastReceiver">
     <intent-filter>
          <action android:name="com.akamai.android.sdk.ACTION_VOC_STATUS" />
          <category android:name="{ApplicationId}" />
     </intent-filter>
</receiver>
```

The client app can then extend the ```VocStatusReceiver``` API and override methods to listen to different status updates like - cache sync start, cache sync done, download policy status failures, new content available etc. 

For example:

```
public class MyVocBroadcastReceiver extends VocStatusReceiver {

    private static final String LOG_TAG = MyVocBroadcastReceiver.class.getSimpleName();

    /**
     * Triggered when the policy status update fails
     *
     * @param context    The application context
     * @param policyCode The policy code such as POLICY_NO_WIFI, POLICY_NO_NETWORK, POLICY_NOT_CHARGING
     */
    protected void onPolicyStatusFailure(Context context, int policyCode) {
        Log.d(LOG_TAG, " onPolicyStatusFailure " + policyCode);
    }

    /**
     * Triggered when cache sync is initiated
     *
     * @param context The application context
     */
    protected void onCacheSynchStart(Context context) {
        Log.d(LOG_TAG, " onCacheSynchStart");
    }

    /**
     * Triggered when cache sync is complete
     *
     * @param context The application context
     */
    protected void onCacheSynchDone(Context context) {
        Log.d(LOG_TAG, " onCacheSynchDone");
    }

    /**
     * Triggered when new content is available
     *
     * @param context     The application context
     * @param numNewFeeds Number of new feeds available
     */
    protected void onNewContentAvailable(Context context, int numNewFeeds) {
        Log.d(LOG_TAG, " onNewContentAvailable " + numNewFeeds);
    }

    /**
     * Triggered when content is purged from server
     *
     * @param context   The application context
     * @param purgeType Type of purge - global ,provider or id based
     * @param purgeIds  Ids of purged content
     */
    protected void onPurgeStatus(Context context, String purgeType, String[] purgeIds) {
        Log.d(LOG_TAG, " onPurgeStatus " + purgeType + " " + purgeIds);
    }

    /**
     * Called when status update to server fails
     *
     * @param context The application context
     */
    protected void onSendServerStatusFailure(Context context) {
        Log.d(LOG_TAG, " onSendServerStatusFailure");
    }

    /**
     * Triggered when a manifest is downloaded from the server.
     * Manifest contains the feed catalogue that can be cached by the client
     *
     * @param context: The application context
     */
    protected void onManifestDownloaded(Context context) {
        Log.d(LOG_TAG, " onManifestDownloaded");
    }
}

```





# Download to Go 

The PCD SDK has a number of ways users can get content, but it's suggested that you start by using Download to Go (D2G). D2G allows users to  browse your video catalog and download one or more selections to their mobile device. 

## Setting up D2G

Before you can use D2G, you need to ingest your video catalog to the PCD Server. Ingesting content isn't a self-service operation at this time, so talk to your Akamai representative about ingesting content. Each piece of content (usually a video) ingested is represented by a unique content Id. 

## Initiating a Download

To initiate a download, you need the ```contentId``` of the content that was ingested. 

- Optionally include the provider ID, but if you have only one video catalog, then the default provider ID is your own. 
- Optionally include a resolution value, the default resolution is medium. 

1. The client triggers a download using ```VocService::downloadFeeds(ArrayList<AnaDownloadPrefs>)```, providing a ```list``` with your catalog ```ContentId, ProviderId``` and ```resolution```.

2. The client can access the metadata of the content being downloaded (represented by ```AnaFeedItem```)  using the API - ```VocUtils.getAnaFeedItemFromContentId```.

3. The client can listen to the changes when data related to a ```contentId``` ( metadata or state) is updated or added, this can be acheived using the standard Android-based cursor notifications, or by extending ```VocDownloadStatusReceiver``` which is described in the Managing Downloads section.

Here is an example to initiate the download using ```ContentId```:

```
public void downloadByContentId() {
    ArrayList<AnaDownloadPrefs> list = new ArrayList<>();
    AnaDownloadPrefs prefs = new AnaDownloadPrefs();
    // Previously ingested content id 
    prefs.setContentId(<ingested content id>);
    prefs.setVideoQualityPreference(VocVideoQualityPreference.LOWRES);
    list.add(prefs);
    VocService.createVocService(this).downloadFeeds(list);
}
```


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


# Managing Downloads 

## Pause and Resume Downloading a Specific Item 

Call the API method ```VocService.pauseFeedDownload``` to pause the ongoing download of a video by passing the ```feedId``` as the argument.

```
final String url = "https://samplevideo.mp4";
// getFeedIdFromUrl method queries the DB and gets the feedId.
final String feedId = VocUtils.getFeedIdByUrl(getApplicationContext(), url);
final VocService vocService = VocService.createVocService(getApplicationContext());
vocService.pauseFeedDownload(feedId);
```
Resume the download of a paused item (or items) by using ```VocService.downloadFeeds```.

```
final String url = "https://samplevideo.mp4";
// getFeedIdFromUrl method queries the DB and gets the feedId.
final String feedId = VocUtils.getFeedIdByUrl(getApplicationContext(), url);
final VocService vocService = VocService.createVocService(getApplicationContext());
VocServiceResult result = vocService.downloadFeeds(new String[]{feedId});
```

## Delete/Cancel Content 

Call the API method ```VocService.deleteFiles``` to delete all the files related to the video from the device by passing the ```feedId``` in the argument. Optionally, the app can retain the thumbnail of the video by setting ```deleteThumbnail``` to ```false```.

```
final String url = "https://samplevideo.mp4";
// getFeedIdFromUrl method queries the DB and gets the feedId.
final String feedId = VocUtils.getFeedIdByUrl(getApplicationContext(), url);
final VocService vocService = VocService.createVocService(getApplicationContext());
VocServiceResult result = vocService.deleteFiles(feedId, false);
```

## Track Progress of a Downloading Video 

The client app can dynamically register ```BroadcastReceiver``` to listen to the download status, such as bytes downloaded, bytes remaining, and ETA for download. The client app can subclass ```VocDownloadStatusReceiver``` class and implement the callback method ```onFeedDownloadStatus``` and use the parameters of the method to show download progress in the UI.

```
public class ClientBroadcastReceiver extends VocDownloadStatusReceiver {

    private static final String LOG_TAG = ClientBroadcastReceiver.class.getSimpleName();

    /**
     * Called whenever the download progress status changes
     *
     * @param context         The application context
     * @param feedId          Feed id of the feed item getting downloaded
     * @param bytesDownloaded Bytes downloaded
     * @param bytesRemaining  Bytes remaining for download
     * @param eta             ETA for the download
     */
    protected void onFeedDownloadStatus(Context context, String feedId, long bytesDownloaded, long bytesRemaining, long eta) {
        Log.d(LOG_TAG, "onFeedDownloadStatus" + " feedId " + feedId);
    }

    /**
     * Called when a feed is queued for download. It means that it will start downloading
     * once other downloads in the download queue are finished
     *
     * @param context The application context
     * @param feedId  Feed id of the feed item to download
     */
    protected void onFeedDownloadQueued(Context context, String feedId) {
        Log.d(LOG_TAG, "onFeedDownloadQueued" + " feedId " + feedId);
    }

    /**
     * Called when a feed download starts
     *
     * @param context The application context
     * @param feedId  Feed id of the feed item to download
     */
    protected void onFeedDownloadStarted(Context context, String feedId) {
        Log.d(LOG_TAG, "onFeedDownloadStarted" + " feedId " + feedId);
    }

    /**
     * Called when a feed download is completed successfully
     *
     * @param context The application context
     * @param feedId  Feed id of the feed item downloaded
     */
    protected void onFeedDownloadCompleted(Context context, String feedId) {
        Log.d(LOG_TAG, "onFeedDownloadCompleted" + " feedId " + feedId);
    }

    /**
     * Called when a feed download is failed
     *
     * @param context   The application context
     * @param feedId    Feed id of the feed item downloaded
     * @param errorCode Error code which hints why the download failed
     */
    protected void onFeedDownloadFailed(Context context, String feedId, int errorCode) {
        Log.d(LOG_TAG, "onFeedDownloadFailed" + " feedId " + feedId + " errorcode " + errorCode);
    }

    /**
     * Callback to let the application know of the content id(s) that were not found
     *
     * @param context    The application context
     * @param contentIds The content ids that were not found
     * @param errorMsg   The error message with more details
     */
    protected void onContentNotFound(Context context, ArrayList<String> contentIds, String errorMsg) {
        Log.d(LOG_TAG, "onContentNotFound" + " contentIds " + contentIds + " errorMsg " + errorMsg);
    }
}
```
The receiver can be registered dynamically or the client manifest can be updated with the new receiver.

```
private BroadcastReceiver receiver;

@Override
public void onCreate(Bundle savedInstanceState) {
   ...
   IntentFilter filter = new IntentFilter();
   filter.addAction("com.akamai.android.sdk.ACTION_VOC_DOWNLOAD_STATUS");
   filter.addCategory(getPackageName());
   receiver = new ClientBroadcastReceiver(); 
   this.registerReceiver(receiver, filter);
   ...
}

@Override
public void onDestroy() {
   this.unregisterReceiver(receiver);
   super.onDestroy();   
}
```

Example change in Client Manifest

```
<manifest . . . >
     <application . . . >
          . . .
          **<!-- {ApplicationId} should be replaced with the client app package name , example - com.domain.applicationname -->**
          **<!-- A receiver to listen to Voc Download Status messages -->**
          <receiver
               android:name="{ApplicationId}.ClientBroadcastReceiver">
                    <intent-filter>
                         <action android:name="com.akamai.android.sdk.ACTION_VOC_DOWNLOAD_STATUS" />
                         <category android:name="{ApplicationId}" />
                    </intent-filter>
          </receiver>
          . . .
     </application>
</manifest>

```
Alternatively instead of listening, the client app can use the API method ```VocService.getDownloadStatus``` by passing the ```feedId``` to get the download information of that particular feed item whenever they need.

```
@Override
public void onCreate(Bundle savedInstanceState) {
   AnaFeedDownloadStatus downloadStatus = vocService.getDownloadStatus(feedId);
   long bytesDownloaded = downloadStatus.getBytesDownloaded();
   long bytesRemaining = downloadStatus.getBytesRemaining();
}
```

## Listening for Data Set Changes

If cursor-based notifications are needed, then you can use ```AnaProviderContract.CONTENT_URI_FEED_DOWNLOAD_STATE```

To listen to changes in download state:

```
AnaProviderContract.Feeditem.DOWNLOAD_STATE

AnaProviderContract.Feeditem.BYTES_DOWNLOADED

AnaProviderContract.Feeditem.DOWNLOAD_FAILURE_ERROR_CODE
```

Note that ```DOWNLOAD_STATE``` and ```DOWNLOAD_FAILURE_ERROR_CODE``` are described in ```VocDownloadStatusReceiver```.

```
Uri.Builder builder = AnaProviderContract.CONTENT_URI_FEED_DOWNLOAD_STATE.buildUpon();
Uri uri = builder.build();
loader = new CursorLoader (getApplicationContext(), uri, null, null, null, null);
```



# SDK Settings 

```VocConfigBuilder``` enables the client to set up the required preferences for voc content like network preference (wifi or wifi + cellular), media path to cached content, other properties that control downloads like cache size, individual file limit, minimum battery level for prefetch, etc. Clients can use this builder to change any values based on its requirement.

```VocConfigBuilder``` can also be used to disable background downloads. When disabled, only the manifest from voc server is downloaded, but the actual content is not; the client would have to use its own download manager to download content. By default background downloads are enabled.

Please see ```VocConfigBuilder``` in the API reference for all the available SDK settings.

## Network Preference 

The API provides control over whether downloads are permitted on wifi, cellular, or both. Setting the ```VocConfigBuilder  networkPreference``` preference affects downloads that have not already started.

```
// Setup SDK Config Network preferences
VocConfigBuilder builder = new VocConfigBuilder(this)
      .networkPreference(VocNetworkPreference.WIFICELLULAR);
```

## Item Download Behavior

The default behavior of the SDK is to preposition all the content from the content catalog. ```VocConfigBuilder``` has several preferences which provides the client an option to define the download behavior of the SDK. This configuration can be configured for example to restrict the auto download of the content and the user will be responsible for downloading the content. The content catalog will still be downloaded after registering the SDK. The client has to make an explicit call to trigger a content download from the catalog.

```
// Setup SDK Config Download preferences
VocConfigBuilder builder = new VocConfigBuilder(this)
      .disableBackgroundDownloads()
      .individualFileLimit(1000) // in MB
      .prefetchLimit(1); // in GB;
```

## Auto Purge

The default behavior of the SDK is to perform content purge based on certain device-specific parameters. ```VocConfigBuilder``` has a preference, ‘evictionStrategy,’ which provides the client an option to choose a strategy for the deletion of the content.

```
// Setup SDK Config Purge preferences
VocConfigBuilder builder = new VocConfigBuilder(this)
      .evictionStrategy(AkaEvictionStrategy.Least_Frequently_Used);
```

## Maximum Number of Concurrent Downloads

The default behavior of the SDK is to perform any number of concurrent file downloads. ```VocConfigBuilder``` has a preference, ‘maxThreadsForSync,’ which provides the client an option to restrict the number of simultaneous downloads. This applies to both
foreground sync and background sync. Use with caution, changing this value will have affect on performance and memory utilization.

```
// Setup SDK Config concurrent download preferences
VocConfigBuilder builder = new VocConfigBuilder(this)
      .maxThreadsForSync(2); // two simultaneous downloads
```
