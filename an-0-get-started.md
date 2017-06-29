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
