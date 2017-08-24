# Introduction

The Predictive Content Delivery (PCD) SDK allows users to download videos intelligetntly to mobile devices. Users can download videos in three ways:

* Predictive - Users set preferences and/or subscribe, and video content is automatically downloaded to their devive. 
* Download to Go - Users initiate downloads by browsing your video catalog.
* Peer-to-Peer (P2) - Android users can share a Wifi Hotspot and download videos from each other. 

Downloads  continue whether the app is active in the foreground, running in the background, or entirely closed. In addition, the SDK acts as a data cache, providing access to content along with its current download state. A sophisticated Download Manager checks the battery, device storage, network, and other variables to make sure that the video transfer will succeed. Errors are thrown if the video transfer fails. 

## Before You Begin

Before you can begin using the SDK, you need to get your PCD SDK License Key. This is obtained from your Akamai representative. 

Once you have your SDK License Key, you can get started right away using the sample app. In order to use your own content, you need to ingest content via the API. Talk to your Akamai representative about this as well. 

## Organization of this Guide

The chapters in this guide follow our preferred development process, and we think you'll get the most from the PCD SDK by doing the steps in order. We have no preference for Android or iOS, we've chosen to order them alphabetically, and so Android is first. A good development strategy follows these steps:

1. First download the zip file and try the sample apps for both systems, so you can get a feel for how the SDK works. 
2. Next you'll want to integrate the SDK into your native mobile app. 
3. Now we suggest implementing Download to Go, which allows users to download and watch your videos. 
4. At this point your video catalog may be growing and this can cause delays loading your homepag, as there are now a large number of thumbnail images. This is a great opportunity to use prepositioning for the video thumbnails. 
5. Now that you know how prepositioning works, you can add video prepositioning as well. 
6. Optionally you may want to add peer-to-peer video sharing to your app. 

## Audience

This document is for developers implementing the PCD SDK; deep knowledge of Android and/or iOS is assumed. 

There are separate instructions for Android and iOS. Using the PCD SDK with both platforms is covered in the following chapters. 
