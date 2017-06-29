# Download to Go 

The PCD SDK has a number of ways users can get content, but it's suggested that you start by using Download to Go (D2G). D2G allows users to browse your video catalog and download one or more selections to their mobile device. 

## Setting up D2G

Before you can use D2G, you need to ingest your video catalog to the PCD Server. Ingesting content isn't a self-service operation at this time, so talk to your Akamai representative about ingesting content. Each content( example a video) ingested is represented by a unique Content Id. 

## Initiating a Download

To initiate a download, you need the content ID. 

- Optionally include the provider ID, but if you have only one video catalog, then the default provider ID is your own.
- Optionally include a resolution value, the default resolution is medium.

1. The client can access the item using the API - ```VocService::getItemsWithContentIds: sourceName:completion:```.
2. If the items that match the ```contentIds``` and ```sourceName``` exists locally, the completion block receives a ```VocItemSet``` instance with those items. ```VocItemSet``` has an array property -”items” which includes those items.
3. If the items that match the ```contentId``` and ```sourceName``` don't exist locally, a server lookup is performed to get the metadata. When the lookup is successful, the items are available locally and ```VocItemSet::items``` holds those new items.
4. The client needs to listen to the changes made to ```VocItemSet``` to get notified when remote lookup adds new items. This can be achieved by adopting the listener protocol ```VocObjSetChangeListener``` and implementing the method ```VocObjSetChangeListener::vocService:objSetDidChange:added:updated:removed:objectsBefore:```. 
Add this listener instance as the listener for ```VocItemSet``` instance received in the completion block.
5. The client can use ```VocItemSet::items``` to download the content.

## iOS Sample

To initiate a download using ```ContentId```, see the following example:

```
@interface ViewController () <VocObjSetChangeListener> 
@property (strong, nonatomic) id<VocItemSet>  itemSet;
 
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
        // Add the listener
        self.itemSet = itemSet;
        [itemSet addListener:self];
    }];
}

```
