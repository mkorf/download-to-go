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



