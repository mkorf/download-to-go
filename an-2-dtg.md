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
