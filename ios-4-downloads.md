# Managing Downloads

## Pause and Resume Downloading a Specific Item

The client can pause a specific content item download by making the ```VocService``` call  ```pauseItemDownload:``` . ```VocItem``` has a readonly property, ```paused``` which the client can use to identify items that are paused. Alternatively, the same pause operation can be performed on one or more ```VocItems``` by making the ```VocService``` call ```pauseDownloadForItems:completion:```. A paused download will not resume until a specific download request has been made.


```[appDelegate.vocService pauseItemDownload :vocItemVideo  completion :^(NSError*  _Nullable  error) {}]; ```


The client can resume the download of an item or items by making the ```VocService``` call ```downloadItems:```.


```[appDelegate.vocService downloadItems:@[vocItemVideo] completion:^(NSError *_Nullable error) {}];```


## Delete, Lock, or Save an Item

The SDK provides two delete options for each item:

- ```VocItem::deleteItem:``` deletes both the metadata and files associated with the item.
- ```VocItem::deleteFiles:``` deletes just the files associated with the item. Since the metadata is not removed, the client can re-download the video at any point after deleting the files. Alternatively, the same operation can be performed on one or more VocItems by making the VocService call ```deleteFilesForItems:options:completion:```. Optionally, the app can retain the video’s thumbnail by setting ```deleteThumbnail``` parameter to ```NO```.

**Note:** Some content consumers exhibit unexpected behavior if the item gets deleted while it is consumed. The SDK doesn’t know whether the item is currently consumed unless it is notified explicitly. It is recommended to call ```VocItem::lock``` before the item gets consumed and ```VocItem::unlock``` after the consumption is completed.

The SDK initiates a purge mechanism based on certain device parameters. To avoid an item getting deleted using the purge mechanism, the user can mark the item as saved by calling ```VocItem::save```.

## Track Progress of a Downloading Video

There are two properties in VocItem that will allow the client app to calculate download progresss.

- ```bytesDownloaded``` - total number of bytes downloaded for the item.
- ```size``` - total size of the item.

The client app can listen for ```bytesDownloaded``` changes by subscribing to the ```VocObjSetChangeListener``` protocol (refer to Listening for Data Set Changes) or using key-value observing (KVO). Alternatively, the client app can use a timer to periodically poll ```bytesDownloaded``` and update the progress.

## Listening for Data Set Changes

Data sets send notifications when items are added, changed, or removed. This is the best way to detect when individual items have finished downloading. To listen for object set changes a class must subscribe to the  ```VocObjSetChangeListener``` protocol.

``` @interface MyClass() <VocObjSetChangeListener> ```

Setting the data set listener is done via ```addListener:``` and is demonstrated in the sections below for sources, categories, and items. Listeners receive callbacks before and after any changes.



```
(void) vocService:(nonnull id < VocService >)vocService objSetWillChange:(nonnull id < VocObjSet >) objSet added:(nonnull NSSet *) added updated:(nonnull NSSet *)updated removed:(nonnull NSSet *) removed objectsAfterChanges:(nonnull NSArray *)objectsAfterChanges
{
    if ([ objSet isEqual:self.itemSet ]) {
        NSLog(@"%ld items will change",   (unsigned long)updated.count);
    }
} (void)vocService:(nonnull id < VocService >)vocService objSetDidChange:(nonnull id < VocObjSet >)objSet
             added:(nonnull NSSet *)added updated:(nonnull NSSet *)updated removed:(nonnull NSSet *)removed
     objectsBefore:(nonnull NSArray *)objectsBefore
{
    if ([ objSet isEqual:self.itemSet ]) {
        NSLog(@"%ld items have changed",  (unsigned long)updated.count);
    }
} 

```
