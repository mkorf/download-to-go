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
