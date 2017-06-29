# Integrating With Your iOS Application

To start using the PCD SDK with your own app, you'll need to include the SDK in your app. If you've already set up Android to work with PCD, you'll note that iOS has a couple extra steps to enable background execution, remote notifications, and  app transport security. Once these are complete you can contiue with initialization and registration. 

## Installing the iOS SDK
1. Download the PCD SDK and unzip it in a folder under your project, e.g.<tt> ~/myproject/pcd_sdk.</tt>
2. Locate <tt>VocSdk.framework</tt> under<tt> ~/myproject/pcd_sdk/iphoneos/</tt> and copy it into your project folder, e.g.<tt> ~/myproject/VocSdk.framework.</tt> This makes the framework available before the first build so you can add it to your project. On each build afterward it will be copied here by the <tt>copy_vocsdk</tt> build phase script.
3. Add the framework to your Xcode project by opening your project in Xcode and choose **File > Add Files to...** and choose <tt>~/myproject/VocSdk.framework</tt>.
4. Click the project name in the Project navigator to open the project settings. 
5. Click the  **General** tab, and under **Embedded Binaries**, click  **+** and choose  **VocSdk.framework**. Click  **Add**.


## Adding a Build Phase 
The right framework for your platform - simulator or device - must be copied into place before building your app. This requires a build phase to be added to your project settings.

1. In Project Settings, choose your build target then click on **Build Phases**.
2. Click the **+** to add a new build phase, and choose **New Run Script Phase**.
3. Drag this build phase to position it after **Target Dependencies**.
4. Single-click its name and rename it to <tt>copy_vocsdk</tt>.
5. Click the arrow to expand the **copy_vocsdk row**.
6. Leave the default shell setting of <tt>/bin/sh</tt>.
7. Add the following text to the script area, noting that the <tt>copy_vocsdk.sh</tt> path is relative to your <tt>.xcodeproj</tt> file and may need modification depending on where you unzipped it (<tt>pcd_sdk</tt> in this example).

```
./pcd_sdk/copy_vocsdk.sh  "${PROJECT_DIR}/pcd_sdk"
```

The second parameter, <tt>${PROJECT_DIR}/pcd_sdk</tt>, is the location of the <tt>/iphoneos</tt> and <tt>/iphonesimulator</tt> folders from the distribution archive.

In this example, the folder structure is as follows:

```
/myproject/myproject.xcodeproj 
/myproject/pcd_sdk/copy_vocsdk.sh 
/myproject/pcd_sdk/iphoneos/ 
/myproject/pcd_sdk/iphonesimulator/
```

The script will copy the correct <tt>VocSdk.framework</tt> to the <tt>/myproject/</tt>, folder at the start of every build.          

## Including the SDK
Import the ```VocSdk``` header in any class files where the SDK is required. Also define a single ```VocService``` object for your app. Create your ```VocService``` object in the app delegate to make the SDK available early in the app lifecycle and to simplify access to the ```VocService``` from other classes.

```
#import <VocSdk/VocSdk.h>
@property (strong , nonatomic) id <VocService>  vocService;
```

## Enabing Background Execution
The PCD SDK downloads content while the application is running. Various factors determine when to start downloading, how much to download, and when to pause downloads. Influencing factors include the state of the mobile network and the quality state of the provider network.

When your app is in the foreground, downloads are happening without any need for changes in your code. However, in order to get best results, the PCD SDK should be able to download when your app is not in the foreground. In order to do that, you need to enable your app for background execution. 

There are two types of background execution that the SDK uses to download content. Enable these two modes within the Xcode project settings, **Capabilities** tab, and **Background Modes** section:

- Remote notifications (remote-notification)
- Background fetch (fetch)

To enable background fetch in the PCD SDK, you need to implement the system method ```UIApplicationDelegate application:performFetchWithCompletionHandler:```
and, from there, pass the message to the ```VocService``` by calling ```VocService application:performFetchWithCompletionHandler:```

Here is what that looks like:

```
- (void)application:(UIApplication *)application performFetchWithCompletionHandler:(void (^)(UIBackgroundFetchResult result))completionHandler
{
    if (![ self.vocService application:application performFetchWithCompletionHandler:completionHandler ]) {
        completionHandler(UIBackgroundFetchResultNoData);
    }
}

```

## Enabling Remote Notifications
To make remote notifications work in the PCD SDK you need to enable the SDK backend to send push notifications to your app, and make the supporting code changes.

To enable the PCD backend to send push notifications to your app, you need to upload your app push certificate (APNS) to the PCD license management portal. The PCD backend must have a valid APNS certificate for your app at all times otherwise push notifications will not work. If you revoke or renew your certificate, make sure you upload it to the PCD license management portal. Instructions on how to generate an APNS push certificate are available on Apple’s web site:
https://developer.apple.com/library/ios/documentation/IDEs/Conceptual/AppDistributionGuide/AddingCapabilities/AddingCapabilities.html#//apple_ref/doc/uid/TP40012582-CH26-SW11

The first step for your app is to register for remote notification and hand the push token to the SDK in the ```AppDelegate``` call  ```application:didRegisterForRemoteNotificationsWithDeviceToken:```.

```
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary
                                                                                *)launchOptions
{
    // the client must register for remote notifications
    [[UIApplication sharedApplication] registerForRemoteNotifications];
    // the client must call this method during launch cycle
    // to request permission from user to use notification
    UIUserNotificationType notificationType = UIUserNotificationTypeAlert;
    [[UIApplication sharedApplication] registerUserNotificationSettings:
     [UIUserNotificationSettings settingsForTypes:notificationType categories:nil]];
}

- (void)application:(UIApplication *)application
    didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken
{
    // pass push token to SDK
    [self.vocService setDevicePushToken:deviceToken];
}
        
```
Also in your application delegate, implement the system method: ```UIApplicationDelegate application:didReceiveRemoteNotification:fetchCompletionHandler:``` and pass this notification to the ```VocService:``` 

```
VocService application:didReceiveRemoteNotification:fetchCompletionHandler:
```
For example:

```
- (void)application:(UIApplication *)application didReceiveRemoteNotification:(NSDictionary *)userInfo fetchCompletionHandler:(void (^)(UIBackgroundFetchResult result))completionHandler
{
    if ([ self.vocService application:application didReceiveRemoteNotification:userInfo fetchCompletionHandler:completionHandler ]) {
        // remote notification was for VoC SDK
        return;
    }
    // remote notification is not for VoC SDK, handle remote notification
    completionHandler(UIBackgroundFetchResultNoData);
}
```

If the client is testing against the development APNS instead of production APNS, the SDK needs to be configured to use development APNS before the SDK registration.

```
vocService.config.useProductionApns = NO;
```

Remote notification and fetch are enabled in background by adding the following key-subkey to the app’s <tt>info.plist</tt> file:

```
<key>UIBackgroundModes</key>
<array>
   <string>fetch</string>
   <string>remote-notification</string>
</array>
```

## Enabling App Transport Security
Starting in iOS 9.0, a new app security feature called App Transport Security (ATS) was introduced and is enabled by default. With ATS enabled, connections must use secure HTTPS instead of HTTP. Additionally, if the app contents that PCD SDK needs to download contain non-SSL items, those downloads will fail. Application developers must ensure that either

- [preferred] HTTPS is used for all content URLs, or
- [insecure] ATS exceptions can be added for certain domains by adding the following <tt>key-subkey</tt> to the app’s <tt>info.plist</tt> file.

The optional ATS key to allow insecure (HTTP) content follows:

```
<key>NSAppTransportSecurity</key>
<dict>
    <key>NSExceptionDomains</key>
    <dict>
        <key>pvoc-anaina.com</key>
        <dict>
            <key>NSIncludesSubdomains</key>
            <true/>
            <key>NSExceptionAllowsInsecureHTTPLoads</key>
            <true/>
        </dict>
    </dict>
</dict>
```

## Initialization
The SDK is initialized by the ```VocService```Factory call ```createServiceWithDelegate:delegateQueue:options:error:```. Initialization and registration should take place early in the app lifecycle to begin prepositioning files as early as possible. 

The recommended place for this is in AppDelegate’s ```application:didFinishLaunchingWithOptions:```. The ```create``` call inputs a reference to the SDK delegate (see SDK Delegate) as well as a configuration options dictionary. The SDK delegate is the class you designate to respond to SDK activity.

The example below is inserted into <tt>AppDelegate.m</tt>. It initializes the SDK and tells it that AppDelegate will be the delegate to handle SDK messages.

```
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
{
    NSError *error = nil;
    self.vocService = [VocServiceFactory  createServiceWithDelegate:self
                       delegateQueue:[ NSOperationQueue  mainQueue ] options:nil
                       error:&error ];
    if (! self.vocService) {
        // error handling - could not start service
        return NO;
    }
    // app initialized
    return YES;
}

```

## Registration
Registration is a one-time event that activates the ```VocService``` with a particular license key. This key is linked to a content catalog, generation of usage statistics, and other SDK capabilities. It is required for most SDK features so registration should occur early within the application.

The ```VocService``` returned from initialization has a state property that indicates whether registration already succeeded on this device. If ```VocService.state``` is equal to ```VocServiceStateNotRegistered``` then registration must be called as follows. Otherwise the SDK is already running.

```
NSString *sdkLicense = @"Your Akamai Provided License Key";
[self.vocService registerWithLicense:sdkLicense
 segments:nil completion:^(NSError *__nullable error) {
    if (error) {                                                           
    // handle failed registration
    }
}];
```

Registration requires a valid SDK license. This license is associated with a particular content set defined on the server portal. The segments field is nil.

Once registration is successful, the SDK calls the ```didRegister:``` method on ```VocServiceDelegate```. This can be used to report or log any problems starting the SDK. This callback is made every time the app starts. The first time the app registers, it is called in response to ```registerWithLicense```. After that, the app is already registered so it is called in response to ```createServiceWithDelegate```.

The service delegate also receives ```didInitialize:``` to indicate that all services are actively running.

## SDK Events

The SDK notifies your app of various events throughout its life cycle. Messages are sent asynchronously to an SDK delegate in your code that implements the ```VocServiceDelegate``` protocol. All of the delegate methods are optional.

```
- (void) vocService :(nonnull VocService *) vocService didBecomeNotRegistered :(nonnull NSDictionary *) info;
- (void) vocService :(nonnull VocService *) vocService didFailToRegister :(nonnull NSError *) error;
- (void) vocService :(nonnull VocService *) vocService didRegister :(nonnull NSDictionary *) info;
- (void) vocService :(nonnull VocService *) vocService didInitialize :(nonnull NSDictionary *) info;
- (void) vocService :(nonnull VocService *) vocService itemsDiscovered :(nonnull NSArray *) items;
- (void) vocService :(nonnull VocService *) vocService itemsStartDownloading :(nonnull NSArray *) items;
- (void) vocService :(nonnull VocService *) vocService itemsDownloaded :(nonnull NSArray *) items;
- (void) vocService :(nonnull VocService *) vocService itemsEvicted :(nonnull NSArray *) items; 
- (void) vocService :(nonnull VocService *) vocService hlsServerStarted:(nonnull NSURL*)url; 
- (void) vocService :(nonnull VocService *) vocService hlsServerStopped:(nonnull NSURL*)url;
```

The SDK delegate is passed to the SDK in the initialization call. Typically, this is the app delegate since its lifetime will span that of the SDK, from registration until shutdown. Define your app delegate as follows to implement the SDK delegate protocol.

```
@interface AppDelegate : UIResponder < UIApplicationDelegate, VocServiceDelegate>
```



