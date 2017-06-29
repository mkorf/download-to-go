# SDK Settings
```VocService``` has a property, ‘config,’ which provides a rich set of API to configure the SDK behavior. Default values for some of the configurations in <tt>VocConfig.h</tt> are set at the global level (to server values) and you could override those at the client level. You should discuss with the PCD SDK team about your default value preferences for permitted network types for download (wifi, cellular, or both), content items per category, daily download size limit, maximum size allowed per file download, permitted time of day for downloading, etc. Some of the important configurations are the following.

## Network Preference
The API provides control over whether downloads are permitted on wifi, cellular, or both. Setting the ```VocConfig networkSelection``` property affects downloads that have not already started.

```
self.vocService.config.networkSelection = VOCNetworkSelectionWifiAndCellular;
```

## Item Download Behavior
The default behavior of the SDK is to preposition all the content from the content catalog. VocConfig has a property, ```itemDownloadBehavior```, which provides  the client an option to define the download behavior of the PCD SDK.  This configuration can be configured to restrict the auto download of the content and the user will be responsible for downloading the content. The content catalog will still be downloaded after registering the PCD SDK. The client has to make an explicit call to trigger a content download from the catalog. The SDK download behaviors are auto download, download of thumbnail files only and no download.
```
self.vocService.config.itemDownloadBehavior = VOCItemDownloadThumbnailOnly ;
```

## Auto Purge
The default behavior of the SDK is to perform content purge based on certain device specific parameters. ```VocConfig``` has a property, ```enableAutoPurge```, which provides  the client an option to disable auto purge and the user will be responsible for the deletion of the content.
```
self.vocService.config.enableAutoPurge = NO ;
```

## Maximum Number of Concurrent Downloads
The default behavior of the SDK is to perform any number of concurrent file downloads. ```VocConfig``` has a property, ```maxConcurrentDownloads```, which provides  the client an option to restrict the number of simultaneous downloads.
```
self.vocService.config.maxConcurrentDownloads = 10 ;
```



