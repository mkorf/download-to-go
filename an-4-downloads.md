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

