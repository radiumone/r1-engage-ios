#Table of Contents
  - [1. System Requirements](#user-content-1-system-requirements)
  - [2. SDK Initialization](#user-content-2-sdk-initialization)
    - [a. Import Files](#user-content-a-import-files)
    - [b. Link the Static Library](#user-content-b-link-the-static-library)
    - [c. Initialize the SDK](#user-content-c-initialize-the-sdk)
  - [3. Feature Activation](#user-content-3-feature-activation)
    - [a. Engage Activation (optional)](#user-content-a-engage-activation)
    - [b. Analytics Activation (optional)](#user-content-b-analytics-activation)
      - [i. Settings](#user-content-i-settings)
      - [ii. Automatic Events](#user-content-ii-automatic-events)
      - [iii. Standard Events](#user-content-iii-standard-events)
      - [iv. Custom Events](#user-content-iv-custom-events)
      - [v. Best Practices](#user-content-v-best-practices)
    - [c. Push Notification Activation (optional)](#user-content-c-push-notification-activation)
      - [i. Setup Apple Push Notification Services](#user-content-i-setup-apple-push-notification-services)
      - [ii. Initialization](#user-content-ii-initialization)
      - [iii. Rich Push Initialization](#user-content-iii-rich-push-initialization)
      - [iv. Deep Link Initialization](#user-content-iv-deep-link-initialization)
      - [v. Segment your Audience](#user-content-v-segment-your-audience)
      - [vi. Inbox Integration](#user-content-vi-inbox-integration)
    - [d. Attribution Tracking Activation (optional)](#user-content-d-attribution-tracking-activation)
      - [i. Track RadiumOne Campaigns](#user-content-i-track-radiumone-campaigns)
      - [ii. Track 3rd party Campaigns](#user-content-ii-track-3rd-party-campaigns)
    - [e. Geofencing Activation (optional)](#user-content-e-geofencing-activation)
  - [4. Submitting your App](#user-content-4-submitting-your-app)

#1. System Requirements
  The R1ConnectEngage SDK supports all mobile and tablet devices running iOS 7.0 or newer (6.0 support coming soon!) with a base requirement of Xcode 6.0 used for development (as you must build using iOS 8 SDK or newer). The downloadable directory (see below "[a. Import Files](#a-import-files)") contains the library and headers for the R1ConnectEngage SDK. 

  The library supports the following architectures:

  * arm7
  * arm7s
  * arm64

  Support for iOS Simulator is also included for the following architectures

  * i386
  * x86_64


#2. SDK Initialization

## a. Import Files
  1.	Download the R1ConnectEngage SDK files:
  git clone git@github.com:radiumone/r1-engage-iOS.git
  2.	Open your iOS project in Xcode.
  3.	Select File -> Add Files to “[YOUR XCODE PROJECT]” project
  4.	Select all files in the "Lib" Folder from the repo you just cloned
  5.	When the dialog box appears, check the "Copy Items into destination group’s folder" checkbox.

  See image below:


  ![Image of addfiles]
(Doc_Images/addfiles.png)

## b. Link the Static Library
  Go to "Build Phases" and make sure libR1ConnectEngage.a file is set in the “Link Binary With Libraries” section. If absent, please add it.

  Make sure you add:

  * CoreLocation.framework
  * CoreBluetooth.framework
  * AdSupport.framework
  * CoreTelephony.framework
  * SystemConfiguration.framework
  * libsqlite3.dylib
  * StoreKit.framework
  * WebKit.framework - should be marked as Optional instead of Required

  See image below:


  ![Image of framework]
(Doc_Images/framework.png)

 It is important to add an entry to the 'Other Linker Flags' setting in your application's Build Settings in Xcode.  Add '-ObjC' to the 'Other Linker Flags' setting if it is not already present.  This is a common required flag when integrating static libraries with your code.


## c. Initialize the SDK
  You will need to initialize the R1ConnectEngage Library in your App Delegate.
####Import the required header files
  At the top of your application delegate include any required headers:

```objc
#import "R1SDK.h"
```

####Initialize the R1ConnectEngage SDK  Instance

```objc
  - (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions 
{     
  R1SDK *sdk = [R1SDK sharedInstance];  

  // Initialize SDK      
  sdk.applicationId = @"YOUR APPLICATION ID";  //Ask your RadiumOne contact for an app id

  return YES; 
}
```


#3. Feature Activation

##a. Engage Activation

R1ConnectEngage SDK supports iOS 6.0 and above. It supports full-screen products (Offerwall, Interstitial, Video) as well as Banners that you can place how you like in your application.  Engage also supports ad mediation allowing you to optionally pull from a larger pool of advertisements, helping you to maximize the monetization of your application.

#### Module Initialization

Engage needs to be enabled in the application's App Delegate. In the `application: didFinishLaunchingWithOptions:` method insert the following code:
```objc	
R1SDK *sdk = [R1SDK sharedInstance];
sdk.engageEnabled = YES;
```

#### Integration of ad views

Engage has support for three full-screen ad views: Offerwall, Interstitial, Video.  In addition to these full-screen ad views, banner ads (of various sizes) are available as well.

All ad views are managed via proxy objects. AdViewProxy objects provide your interface the ability to configure, load, show and respond to status updates of an ad. You can create each type of ad proxy via the 'adServerManager' object available via the 'R1EngageSDK' shared object.

The methods used to create the different AdViewProxy objects are:

* bannerAdViewProxy
* interstitialAdViewProxy
* videoAdViewProxy
* offerwallAdViewProxy

AdViewProxy objects created with these methods will be one of the corresponding types:

* R1BannerAdProxy
* R1InterstitialAdProxy
* R1VideoAdProxy
* R1OfferwallAdProxy

As an example, you can create an interstitial proxy with the following line of code:
```objc
  self.adProxy = [[R1EngageSDK sharedInstance].adServerManager interstitialAdViewProxy];
```

In the above example, self.adProxy refers to a property of type R1InterstitialAdProxy of the calling class instance.  It is required that you retain (or, when using ARC, have a strong reference to) the created adProxy instance for the lifetime of the adView as it will release all of its managed views and controllers once no other object is holding on to it.

To continue our example, our property, 'adProxy' would be declared in the owning class as:
```objc
@property (nonatomic, strong) R1InterstitialAdProxy *adProxy;  // when using ARC

or 

@property (nonatomic, retain) R1InterstitialAdProxy *adProxy;  // when using classic memory management
```

#### Configuring adViewProxy objects

After the adViewProxy has been created and retained as detailed above, it must be configured prior to loading an ad.  This is a fairly straightforward process as there are just three attributes to set:

	* where the ad will be displayed
	* attaching any placement identifiers needed to display the ad or track user engagement
	* setting up callbacks for status updates from the adViewProxy.


#### Setting the view

All of the adViewProxy objects require a root view controller to be specified for their display (for the three full-screen ad views) or for the display of a supplemental full-screen view that may be presented when a banner ad receives a user tap.  The view controller passed in should be the root view controller for whatever view is meant to show the advertisement.  i.e. if the active view is part of a navigation view controller or tab view controller, then those parent view controllers should be passed in.
```objc
  // assuming self is a view controller in a tab view
  [self.adProxy setRootViewControllerForAdPresentation:self.parentViewController];
```

#### PlacementIds

PlacementIds identify, sometimes uniquely, one ad placement from another.  Because Engage can support multiple ad networks via its mediation feature, placementIds allow the setting of an identification tag for each ad and network that is enabled in the application.  Using this setting with mediation will be discussed in more depth later.  It is advisable that a unique value be set for Engage for your own tracking needs.  It can be any custom value of your choosing (we will save and return back up to 100 characters of it).  Since it is possible to set a placement id for more than one ad network at a time, the values are set as part of a dictionary.

Continuing from the previous code example the placementIds can be set:
```objc
  NSMutableDictionary *dataDict = [NSMutableDictionary dictionaryWithCapacity:1];
  
  [dataDict setObject:@"level1Interstitial" forKey:R1AdNetworkEngage];

  self.adProxy.placementIds = dataDict;
```

#### Setting the callbacks

As the AdViewProxy completes various actions or encounters specific states, it will send notifications to registered listeners.  The messages it will send are:

* R1AdStateLoadedNotification;
	// Your application initiated the loading of an ad in a view and it has successfully completed and is ready to show
* R1AdStateFailedNotification;
	// Your application initiated the loading of an ad in a view and it failed to load due to an error
* R1AdStateNoContentNotification;
	// Your application initiated the loading of an ad in a view and it failed to load due to no advertisement being available to serve
* R1AdStateWillAppearNotification;
	// Your application initiated the display of an ad or a user tapped on a banner ad
* R1AdStateWillDisappearNotification;
	// A full-screen advertising view is about to be dismissed but is still visible
* R1AdStateDidDisappearNotification;
	// A full-screen advertising view finished being dismissed and is no longer visible - time to dispose of ad proxy

While the application may want to respond to any of these notifications, one that is particularly important is the 'R1AdStateLoadedNotification' as this can act as the trigger to show the ad or otherwise prepare to show the ad.

An example of how to register for this notification may look something like:

```objc
  [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(interstitialLoaded:) name:R1AdStateLoadedNotification object:self.adProxy];
```

In this case, 'self' is the object that will receive the 'interstitialLoaded:' callback.  Because self.adProxy was specified as the object, only R1AdStateLoadedNotifications from that AdViewProxy will be sent to our 'self' object.

An implementation of the notification handler might look like this:

```objc
- (void)interstitialLoaded:(NSNotification *)notification
{
	// Now that the ad is loaded, show it immediately
   [self.adProxy showInterstitial];
}
```

In the above example, as soon as the R1AdStateLoadedNotification is received, the application makes the call to show the interstitial ad view.  But your application might also use this notification as a point to initiate some other action before finally showing the advertisement.

Note that the three full-screen ad types: interstitial, video and offerwall, can all be handled correctly with the preceding code sample.  For a banner ad, the application would respond to the R1AdStateLoadedNotification slightly differently.  There is no explicit 'show' method for a banner ad.  Once the AdViewProxy notifies the application that a banner has loaded, it is the application's responsibility to add it to the appropriate view hierarchy as it sees fit using animation or any other available option.

```objc
- (void)bannerLoaded:(NSNotification *)notification
{
  NSDictionary  *userInfo = notification.userInfo;
  UIView        *bannerView = [userInfo objectForKey:R1BannerViewKey];
    
  if(![bannerView superview])
  {
	  //assuming self is a view controller
    [self.view addSubview:bannerView];
  }
}
```

The R1BannerAdProxy will include the newly loaded banner view in the notification callback in the userInfo dictionary.  Use the 'R1BannerViewKey' key to access it so you can make any last minutes adjustments to the view before you ad it to the appropriate view hierarchy.  Note, when your application clears its reference to a R1BannerAdProxy instance, any banner view in a view hierarchy will automatically (and immediately) remove itself from its parent view hierarchy before the AdViewProxy disposes of view.

#### Final Step - Load the Ad

In the previous sections, we've outlined how to create, configure and show a full-screen ad.  But one critical step remains - loading the ad.  Loading an advertisement is done through a distinct call on each AdProxy instance. Full-screen adViewProxy objects have one of these selectors.  Each kick off the network call to load the advertisement assets.

* loadInterstitial
* loadVideo
* loadOfferwall

R1BannerAdProxy objects are only slightly different.

* loadBannerType

This selector takes an enumerated value from the following list of values to request a banner of a certain size.  Note that the size of the banner returned is not guaranteed to match the requested dimensions - you must check the bounds of the view when your application receives the R1AdStateLoadedNotification.

```objc
  R1_300x50BannerType
  R1_320x50BannerType
  R1_480x50BannerType
  R1_300x250BannerType
  R1_728x90BannerType
  R1_1024x90BannerType
```

#### Cleaning up

After a full-screen ad has displayed or if your application has navigated away from a view that was displaying a banner ad, it is time to release your reference to the corresponding adViewProxy instance.

```objc
// unregister your notification callback first!
  [[NSNotificationCenter defaultCenter] removeObserver:self name:R1AdStateLoadedNotification object:self.adProxy];

// now it is safe to release your ad proxy reference.
	self.adProxy = nil;
```

As a very basic example, the following demonstrates the integration of a banner ad along with an interstitial ad.  Other AdViewProxys are integrated in a similar way. (Note: This demonstrates all the necessary code required to display an advertisement, but is not suggesting, for example, that you should show an interstitial ad as soon as your main view loads.)

In your main view controller header...
```objc
#import <UIKit/UIKit.h>
#import "R1EngageSDK.h"

@interface ViewController : UIViewController

@property (nonatomic, strong) R1BannerAdProxy *bannerProxy;
@property (nonatomic, strong) R1InterstitialAdProxy *interstitialProxy;

@end
```

In your main view controller implementation file...
```objc
- (void)viewDidLoad
{
  [super viewDidLoad];
  
  // Banner setup
  self.bannerProxy = [[R1EngageSDK sharedInstance].adServerManager bannerAdViewProxy];
  
  self.bannerProxy.placementIds = [NSDictionary dictionaryWithObjectsAndKeys:@"mainScreenBanner", R1AdNetworkEngage, nil];
  
  [self.bannerProxy setRootViewControllerForAdPresentation:self];
  
  [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(bannerLoaded:) name:R1AdStateLoadedNotification object:self.bannerProxy];
  [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(bannerLoadFailed:) name:R1AdStateFailedNotification object:self.bannerProxy];
  [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(bannerLoadFailed:) name:R1AdStateNoContentNotification object:self.bannerProxy];
  
  [self.bannerProxy loadBannerType:R1_320x50BannerType];
  
  
  // Interstitial setup
  self.interstitialProxy = [[R1EngageSDK sharedInstance].adServerManager interstitialAdViewProxy];
  
  self.interstitialProxy.placementIds = [NSDictionary dictionaryWithObjectsAndKeys:@"mainScreenInterstitial", R1AdNetworkEngage, nil];
  
  [self.interstitialProxy setRootViewControllerForAdPresentation:self];
  
  [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(interstitialLoaded:) name:R1AdStateLoadedNotification object:self.interstitialProxy];
  [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(interstitialDismissed:) name:R1AdStateDidDisappearNotification  object:self.interstitialProxy];
  
  [self.interstitialProxy loadInterstitial];
}

- (void)bannerLoaded:(NSNotification *)notification
{
  NSDictionary  *userInfo = notification.userInfo;
  UIView        *bannerView = [userInfo objectForKey:R1BannerViewKey];
    
  if(![bannerView superview])
  {
    [self.view addSubview:bannerView];
  }
}

- (void)bannerLoadFailed:(NSNotification *)notification
{
  NSString  *noteName = notification.name;
    
  if([noteName isEqualToString:R1AdStateFailedNotification])
  {
    [[NSNotificationCenter defaultCenter] removeObserver:self name:nil object:self.bannerProxy];

	// release our proxy
	// since it is a general error we can try again shortly
    self.bannerProxy = nil;
  }
  else if([noteName isEqualToString:R1AdStateNoContentNotification])
  {
    [[NSNotificationCenter defaultCenter] removeObserver:self name:nil object:self.bannerProxy];

	// release our proxy
	// since there is no content, we can try a different size or just wait
	// awhile to see if more ad inventory is available later
    self.bannerProxy = nil;
  }
}

- (void)interstitialLoaded:(NSNotification *)notification
{
	// The interstitial is ready to be displayed, so go ahead and show it
  [self.interstitialProxy showInterstitial];
}

- (void)interstitialDismissed:(NSNotification *)notification
{
	// The user dismissed the ad - clean up and release our reference to the proxy
  [[NSNotificationCenter defaultCenter] removeObserver:self name:nil object:self.interstitialProxy];
  
  self.interstitialProxy = nil;
}
```

#### Ad Mediation

Ad mediation allow your application to pull from a wider pool of ad networks to fulfill an ad request giving you confidence that there will be ads to display when you want them.  Engage supports mediation by enabling ad fulfillment via the following ad networks:

* Google AdMob - https://www.google.com/ads/admob/
* MoPub - http://www.mopub.com

#### Enabling ad mediation

To enable mediation in your application is a four step process.

1) Set up an account and create placement ids (adUnits) in the AdMob, MoPub web portals.  AdUnitIds are tied to each ad you place in your application.  You should create a unique AdUnitId for each ad placement in your application.

2) Add the mediation adapter libraries (of each desired network) to your project.  These mediation adapter libraries can be found in the 'Mediation Adapters" subfolder in the SDK folder that you downloaded.

* libR1AdMobMediationAdapter.a
* libR1MoPubMediationAdapter.a

  Go to the "Build Phases" tab of your target application's Xcode build settings and make sure the desired library file (from the above list) is set in the “Link Binary With Libraries” section. If absent, please add it.

3) In your code, enable mediation behavior by setting the 'mediationEnabled' flag.  This can be done anywhere but a recommended spot would be the place in your code where you enable the Engage module.
```objc
  [R1EngageSDK sharedInstance].adServerManager.mediationEnabled = YES;
```

4) The adUnit ids that you generate in the web portals must be used to initialize each adViewProxy instance created using the 'placementIds' property of the adViewProxy.
```objc
    NSMutableDictionary *dataDict = [NSMutableDictionary dictionaryWithCapacity:3];
  
    [dataDict setObject:@"mainScreenBanner" forKey:R1AdNetworkEngage];
    [dataDict setObject:@"YourAdMobAdUnitId" forKey:R1AdNetworkAdMob];
    [dataDict setObject:@"YourMoPubAdUnitId" forKey:R1AdNetworkMoPub];

    self.bannerProxy.placementIds = dataDict;
```

Okay, to support mediation via MoPub there actually is a fifth step (sorry!) 5) Add the MoPub specific resource files to your Xcode project.
Alongside the libR1MoPubMediationAdapter.a file, you should find a 'MoPubResources' folder which contains:

MPCloseButtonX.png
MPCloseButtonX@2x.png
MPCloseButtonX@3x.png
MRAID.bundle
MPAdBrowserController.xib

These files are all required to be included in your application for MoPub advertisements to operate correctly. Again, these files are only required if you have mediation enabled and MoPub setup. You can add the png image files to your Xcode project's Images.xcassets catalog. The MRAID.bundle and MPAdBrowserController.xib can be placed with any of the other auxiliary resource files in your project. e.g. application plist.



That's all that is required in your app! It is required to also configure the rules by which Engage will execute mediation.  The mediation rules can be managed via the Engage web portal (http://gwallet.net/gwallet-admin).  Now, whenever, you request an advertisement, Engage will automatically attempt to fill the request according to the rules you specified in the Engage web portal.

####Checking rewards information

To receive rewards information setup the delegate:
```objc
[R1EngageSDK sharedInstance].delegate = self;
```

And add method to the code:

```objc
- (void)engageSDK:(R1EngageSDK *)r1engage didReceiveNewReward:(NSUInteger)reward
{
  //your code here
}
```

On close of any full-screen view controller, the Engage engine will automatically check for new
rewards. But you can do it manually (for example after application launching):

```objc
  [[R1EngageSDK sharedInstance] checkCompletions];
```


##b. Analytics Activation

#### Setup your App Delegate

```objc
#import "R1SDK.h"
#import "R1Emitter.h"
```

```objc
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions 
{     
  R1SDK *sdk = [R1SDK sharedInstance];  

  // Initialize SDK     
  sdk.applicationId = @"YOUR APPLICATION ID";  //Ask your RadiumOne contact for an app id

  // Start Analytics     
  [sdk start];   

  return YES; 
}
```

### i. Settings
The following is a list of configuration parameters for the Analytics.  Most of these contain values that are sent to the tracking server to help identify your app on our platform and to provide analytics on sessions and location.

####Configuration Parameters


***applicationUserId***

Optional current user identifier.
```objc
[R1SDK sharedInstance].applicationUserId = @"12345";
```

***location***

The current user location coordinates. Use only if your application already uses location services.

```objc
[R1SDK sharedInstance].location = [locations lastObject];
```

***locationService***

If your application did not use location information before this SDK installation, you can use locationService in this SDK to enable or disable it:

```objc
[R1SDK sharedInstance].locationService.enabled = YES;
```

When enabled, such as in the example above, location information will be sent automatically. However, locationService doesn’t fetch the location constantly. For instance, when the location is received the SDK will turn off the location in CLLocationManager and wait 10 minutes (by default) before attempting to retrieve it again. You can change this value:
```objc
[R1SDK sharedInstance].locationService.autoupdateTimeout = 1200; // 20 minutes
```

N.B. - The Connect locationService uses the Location Manager in iOS.  For
deployment on iOS 8 and newer, it is required that the application's property
list (plist) file includes one of the two following keys:

NSLocationAlwaysUsageDescription
//provide location updates whether the user is actively using the application or
not via infrequent background location updates
// OR
NSLocationWhenInUseUsageDescription
//provide location updates only while the user has your application as the
foreground running app

These are string values that need to include a description suitable to present
to a user.  The description should explain the reason the application is
requesting access to the user's location.  Therefore, description you give
should try to incorporate or reference the user benefit that may be possible
through sharing location.


***appName***

The application name associated with the emitter. By default, this property is populated with the `CFBundleName` string from the application bundle. If you wish to override this property, you must do so before making any tracking calls.

```objc
[R1Emitter sharedInstance].appName = @"Custom application name";
```

***appId***

Default: nil

The application identifier associated with this emitter.  If you wish to set this property, you must do so before making any tracking calls. Note: that this is not your app's bundle id, like e.g. com.example.appname.

```objc
[R1Emitter sharedInstance].appId = @"12345678";
```

***appVersion***

The application version associated with this emitter. By default, this property is populated with the `CFBundleShortVersionString` string from the application bundle. If you wish to override this property, you must do so before making any tracking calls.

```objc
[R1Emitter sharedInstance].appVersion = @"1.0.2";
```

***sessionStart***

If true, indicates the start of a new session. Note that when a emitter is first initiated, this is initialized to true. To prevent this default behavior, set this to `NO` when the tracker is first obtained.

Setting this does not send any data. If this is true, when the next emitter call is made, a parameter will be added, resulting in the emitter information indicating the start of a session.  This flag will be cleared.

```objc
[R1Emitter sharedInstance].sessionStart = YES;
// Your code here
[R1Emitter sharedInstance].sessionStart = NO;
```

***sessionTimeout***

If positive, indicates how long (in seconds) the application must transition to the inactive or background state for before the tracker will automatically indicate the start of a new session. This occurs when the app becomes active again by setting sessionStart to true. For example, if this is set to 30 seconds, and the user receives a phone call that lasts for 45 seconds while using the app, upon returning to the app, the sessionStart parameter will be set to true. If the phone call lasted 10 seconds, sessionStart will not be modified.

To disable automatic session tracking, set this to a negative value. To indicate the start of a session anytime the app becomes inactive or backgrounded, set this to zero.

By default, this is 30 seconds.

```objc
[R1Emitter sharedInstance].sessionTimeout = 15;
```

### ii. Automatic Events


The R1ConnectEngage SDK will automatically capture some generic events in order to get the most meaningful data on how users interact with your app. These events are triggered when the state of the application is changed, and therefore do not require any additional code to work out of the box:

**Launch** - emitted when the app starts

**First Launch** - emitted when the app starts for the first time

**First Launch After Update** - emitted when the app starts for the first time after a version upgrade

**Suspend** - emitted when the app is put into the background state

**Resume** - emitted when the app returns from the background state

**Application Opened** - emitted when the app is opened and can measure when your app is opened after a message is sent

**Session Start** - emitted when a new session begins

**Session End** - emitted when a session ends; includes a Session Length attribute with the session length in seconds.


### iii. Standard Events

Standard events cover all the main user flows (login, register, share, purchase...) in a standardized format for optimized reporting on the portal, providing a great foundation to your analytics strategy. Once you set them up in your code, they unlock great insights, particularly on user lifetime value.

*Note: The last argument in all of the following emitter callbacks, otherInfo, is a dictionary of “key”,”value” pairs or nil, which enables you to customize these events as much as you want.*

**Login**

Tracks a user login within the app

```objc
[[R1Emitter sharedInstance] emitLoginWithUserID:@"userId"
userName:@"user_name"
otherInfo:@{@"custom_key":@"value"}];
```

**Registration**

Records a user registration within the app

```objc
[[R1Emitter sharedInstance] emitRegistrationWithUserID:@"userId"
userName:@"userName"
country:@"country"
state:@"state"
city:@"city"
otherInfo:@{@"custom_key":@"value"}];
```

**Facebook connect**

Allows access to Facebook services

```objc
NSArray *permissions = @[[R1EmitterSocialPermission socialPermissionWithName:@"photos" granted:YES]];

[[R1Emitter sharedInstance] emitFBConnectWithPermissions:permissions
otherInfo:@{@"custom_key":@"value"}];
```

**Twitter connect**

Allows access to Twitter services

```objc
NSArray *permissions = @[[R1EmitterSocialPermission socialPermissionWithName:@"photos" granted:YES]];

[[R1Emitter sharedInstance] emitTConnectWithUserID:@"12345"
userName:@"user_name"
permissions:permissions
otherInfo:@{@"custom_key":@"value"}];
```

**User Info**

Enables you to send user profiles to the backend.

```objc
R1EmitterUserInfo *userInfo = [R1EmitterUserInfo userInfoWithUserID:@"userId"
userName:@"userName"
email:@"user@email.com"
firstName:@"first name"
lastName:@"last name"
streetAddress:@"streetAddress"
phone:@"phone"
city:@"city"
state:@"state"
zip:@"zip"];

[[R1Emitter sharedInstance] emitUserInfo:userInfo
otherInfo:@{@"custom_key":@"value"}];
```

**Upgrade**

Tracks an application version upgrade

```objc
[[R1Emitter sharedInstance] emitUpgradeWithOtherInfo:@{@"custom_key":@"value"}];
```

**Trial Upgrade**

Tracks an application upgrade from a trial version to a full version

```objc
[[R1Emitter sharedInstance] emitTrialUpgradeWithOtherInfo:@{@"custom_key":@"value"}];
```

**Screen View**

A page view that provides info about that screen

```objc
[[R1Emitter sharedInstance] emitScreenViewWithDocumentTitle:@"title"
contentDescription:@"description"
documentLocationUrl:@"http://www.example.com/path"
documentHostName:@"example.com"
documentPath:@"path"
otherInfo:@{@"custom_key":@"value"}];
```

**Transaction**

```objc
[[R1Emitter sharedInstance] emitTransactionWithID:@"transaction_id"
storeID:@"store_id"
storeName:@"store_name"
cartID:@"cart_id"
orderID:@"order_id"
totalSale:1.5
currency:@"USD"
shippingCosts:10.5
transactionTax:12.0
otherInfo:@{@"custom_key":@"value"}];
```

**TransactionItem**

```objc
R1EmitterLineItem *lineItem = [R1EmitterLineItem itemWithID:@"product_id"
name:@"product_name"
quantity:5
unitOfMeasure:@"unit"
msrPrice:10
pricePaid:10
currency:@"USD"
itemCategory:@"category"];

[[R1Emitter sharedInstance] emitTransactionItemWithTransactionID:@"transaction_id"
lineItem:lineItem
otherInfo:@{@"custom_key":@"value"}];
```

**Create Cart**

```objc
[[R1Emitter sharedInstance] emitCartCreateWithCartID:@"cart_id"
otherInfo:@{@"custom_key":@"value"}];
```

**Delete Cart**

```objc
[[R1Emitter sharedInstance] emitCartDeleteWithCartID:@"cart_id"
otherInfo:@{@"custom_key":@"value"}];
```

**Add To Cart**

```objc
R1EmitterLineItem *lineItem = [R1EmitterLineItem itemWithID:@"product_id"
name:@"product_name"
quantity:5
unitOfMeasure:@"unit"
msrPrice:10
pricePaid:10
currency:@"USD"
itemCategory:@"category"];

[[R1Emitter sharedInstance] emitAddToCartWithCartID:@"cart_id"
lineItem:lineItem
otherInfo:@{@"custom_key":@"value"}];
```

**Delete From Cart**

```objc
R1EmitterLineItem *lineItem = [R1EmitterLineItem itemWithID:@"product_id"
name:@"product_name"
quantity:5
unitOfMeasure:@"unit"
msrPrice:10
pricePaid:10
currency:@"USD"
itemCategory:@"category"];

[[R1Emitter sharedInstance] emitDeleteFromCartWithCartID:@"cart_id"
lineItem:lineItem
otherInfo:@{@"custom_key":@"value"}];
```


###iv. Custom Events

Custom events allow you to create and track specific events that are more closely aligned with your app. If planned and structured correctly, custom events can be strong indicators of user intent and behavior. Some examples include pressing the “like” button, playing a song, changing the screen mode from portrait to landscape, and zooming in or out of an image. These are all actions by the user that could be tracked with events.

To include tracking of custom events for the mobile app, the following callbacks need to be included in the application code:

```objc
// Emits a custom event without parameters
[[R1Emitter sharedInstance] emitEvent:@"Your custom event name"];

// Emits a custom event with parameters
[[R1Emitter sharedInstance] emitEvent:@"Your custom event name"
withParameters:@{@"key":@"value"}];
```


###v. Best Practices
####Event Naming Convention
One common mistake is to parameterize event names (with user data for example). Event names should be hard-coded values that you use to segment data on a specific category of event. 

Example: "ProfileViewing"

Avoid: "Profile Viewing - Lady Gaga's profile"

As you may have thousands of user profiles in your database, it is preferable to keep the event name high level ("ProfileViewing") so you can run interesting analytics on it. High-level events help answer questions like "how many profiles does a user visit every day on average?" 

####Parameter Variance

Another common mistake is to add parameters to the event that have too many possible values. To follow up on the previous example, one may decide to add the number of profile followers as an event parameter:

```objc
[[R1Emitter sharedInstance] emitEvent:@"ProfileViewing"
withParameters:@{"profileFollowers":profileFollowers}];
```

Again, the problem here is that each profile may have any number of followers. This will fragment your data too much to extract any valuable information.

A good strategy would be to define relevant buckets for high variance parameters. In this case, it might be more relevant to separate traffic on the profiles with a high follower count from traffic on profiles with a low count. You could define 3 categories: 

- "VERY_INFLUENTIAL" for profiles > 100,000 
- "INFLUENTIAL" for profile > 10,000 and <= 100,000
- "NON_INFLUENTIAL" for profile <= 10,000

A proper event could be 

```objc
[[R1Emitter sharedInstance] emitEvent:@"ProfileViewing"
withParameters:@{"profileFollowersBucket":@"VERY_INFLUENTIAL"}];
```


This will enable you to create more insightful reports.

##c. Push Notification Activation

###i. Setup Apple Push Notification Services

####Prerequisites for Apple Push Notification Services Setup
######The Importance of Setting your App as in “Production” or in “Development”
When creating or editing an app on RadiumOne Connect you can set the status of the app to either “Production” or “Development”. “Production” status for an app is considered to be a live app that is in the hands of real users and will have notifications and other data running on live servers. A “Development” status for an app is one that you are still performing testing on and will not be viewed by any real-life audiences because it will stay on test servers.

In the context of push notifications, it is important to know this difference because Apple will treat these two servers separately. Also device tokens for “Development” will not work on “Production” and vice versa. We recommend a Development app version and Production app version for your app on RadiumOne Connect to keep Push SSL certificates separate for each. You can also continue testing and experimenting on one app without worrying about it affecting your live app audience in any way.
######iOS Developer Program Enrollment
This doc assumes that you are enrolled in the iOS Developer Program. If you are not, please enroll [here](https://developer.apple.com/programs/ios/). Being in the Apple Developer Program is a required component to have your iOS app communicate with the RadiumOne Connect service for push notifications and is necessary for the next step of setting up your app with the Apple Push Notification Service (APNs). It is also essential that you have “Team Agent” role access in the iOS Developer Program to complete this process.

####Configuring your App for Apple Push Notifications
######Apple Developer Members Center
Make sure you are logged into the [Apple Developer Members Center](https://daw.apple.com/cgi-bin/WebObjects/DSAuthWeb.woa/wa/login?&appIdKey=891bd3417a7776362562d2197f89480a8547b108fd934911bcbea0110d07f757&path=%2F%2Fmembercenter%2Findex.action). Once you are logged in you can locate your application in the “Identifiers” folder list.

![Files in xCode project](http://mcpdemo.herokuapp.com/static/img/help/ios_integration/image001.jpg)

######Registered App ID
If you have not registered an App ID yet it is important that you do so now. You will need to click the “+” symbol, fill out the form, and check the “Push Notifications” checkbox. Please keep in mind it is possible to edit these choices after the App ID is registered.


![Files in xCode project](http://mcpdemo.herokuapp.com/static/img/help/ios_integration/image002.jpg)
![Files in xCode project](http://mcpdemo.herokuapp.com/static/img/help/ios_integration/image003.jpg)

You can expand the application and when doing so you will see two settings for push notifications. They will have either yellow or green status icons like here:

![Files in xCode project](http://mcpdemo.herokuapp.com/static/img/help/ios_integration/image004.jpg)

You will need to click Edit to continue. If the Edit button is not visible it is because you do not have “Team Agent” role access. This role is necessary for getting an SSL certificate.

######Creating an SSL Certificate
To enable the Development or Production Push SSL Certificate please click Edit. (It is important to note that each certificate is limited to a single app, identified by its bundle ID and limited to one of two environments, either Development or Production. Read more info [here](https://developer.apple.com/library/ios/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/Chapters/ProvisioningDevelopment.html#//apple_ref/doc/uid/TP40008194-CH104-SW1).)

![Files in xCode project](http://mcpdemo.herokuapp.com/static/img/help/ios_integration/image005.jpg)


You will see a Create Certificate button, after clicking it you will see the “Add iOS Certificate Assistant”. Please follow the instructions presented in the assistant which includes launching the “Keychain Access” application, generating a “Certificate Signing Request (CSR)”, generating an SSL Certificate, etc.

If you follow the assistant correctly, after downloading and opening the SSL Certificate you should have it added under “My Certificates” or “Certificates” in your “Keychain Access” application. Also when you are returned to the Configure App ID page the certificate should be badged with a green circle and the label “Enabled”.

![Files in xCode project](http://mcpdemo.herokuapp.com/static/img/help/ios_integration/image006.jpg)

######Exporting the SSL Certificate
If not already in the “Keychain Access” app that contains your certificate, please open it and select the certificate that you just added. Once you select the certificate go to File > Export Items and export it as a Personal Information Exchange (.p12) file. When saving the file be sure to use the Personal Information Exchange (.p12) format.

![Files in xCode project](http://mcpdemo.herokuapp.com/static/img/help/ios_integration/image007.jpg)

######Emailing your SSL certificate
After downloading your 2 certificates (one for production, one for development), please send them to your RadiumOne account manager (with certificate passwords if you choose to add any).


###ii. Initialization

####Setup your App Delegate


```objc
#import "R1SDK.h"
#import "R1Emitter.h"
#import "R1Push.h"
```


```objc
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
{
  R1SDK *sdk = [R1SDK sharedInstance];

  // Initialize SDK
  sdk.applicationId = @"Application ID";  //Ask your RadiumOne contact for an app id

  // Initialize Push Notification
  sdk.clientKey = @"Your Client Key";  //Ask your RadiumOne contact for a client key
  [[R1Push sharedInstance] handleNotification:[launchOptions valueForKey:UIApplicationLaunchOptionsRemoteNotificationKey]
    applicationState: application.applicationState];
  [[R1Push sharedInstance] registerForRemoteNotificationTypes:(UIRemoteNotificationTypeBadge |
      UIRemoteNotificationTypeSound |
      UIRemoteNotificationTypeAlert)];

  // Start SDK
  [sdk start];
  return YES;
}
```


####Register for Remote Notifications



```objc
- (void)application:(UIApplication *)application didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken
{
  [[R1Push sharedInstance] registerDeviceToken:deviceToken];
}
- (void)application:(UIApplication *)application didFailToRegisterForRemoteNotificationsWithError:(NSError *)error
{
  [[R1Push sharedInstance] failToRegisterDeviceTokenWithError:error];
}
- (void)application:(UIApplication *)application didReceiveRemoteNotification:(NSDictionary *)userInfo
{
  [[R1Push sharedInstance] handleNotification:userInfo applicationState:application.applicationState];
}

// For iOS 8 and newer, it is required to implement the following callback method

- (void)application:(UIApplication *)application didRegisterUserNotificationSettings:(UIUserNotificationSettings *)notificationSettings
{
  if(![application isRegisteredForRemoteNotifications])
  {
    [application registerForRemoteNotifications];
  }
}
```

Push is disabled by default. You can enable it in the *application:didFinishLaunchingWithOptions* method or later.

```objc
[[R1Push sharedInstance] setPushEnabled:YES];
```


NOTE: If you enabled it in the *application:didFinishLaunchingWithOptions* method, the Push Notification AlertView will be showed at first application start.


###iii. Rich Push Initialization    

Rich *Push Notifications* send a URL that opens in a WebView when a user responds to a system notification.  No set up is required for this feature.


###iv. Deep Link Initialization    

Deep linking *Push Notifications* open up a designated view in an application when a user responds to a system notification.  To properly handle deep link push receipts, please read Apple's documentation:  https://developer.apple.com/library/ios/documentation/iPhone/Conceptual/iPhoneOSProgrammingGuide/Inter-AppCommunication/Inter-AppCommunication.html#//apple_ref/doc/uid/TP40007072-CH6-SW10
http://wiki.akosma.com/IPhone_URL_Schemes


###v. Segment your Audience    

You can specify Tags for *R1ConnectEngage SDK* to send *Push Notifications* to certain groups of users.  You can then send *Push Notifications* to users with your specific tags.

The maximum length of a Tag is 128 characters.

*R1ConnectEngage SDK* saves Tags. You do not have to add Tags every time the application is launched.

***Add a new Tag***

```objc
[[R1Push sharedInstance].tags addTag:@"NEW TAG"];
```

***Add multiple Tags***

```objc
[[R1Push sharedInstance].tags addTags:@[ @"NEW TAG 1", @"NEW TAG 2" ]];
```

***Remove existing Tag***

```objc
[[R1Push sharedInstance].tags removeTag:@"EXIST TAG"];
```

***Remove multiple Tags***

```objc
[[R1Push sharedInstance].tags removeTags:@[ @"EXIST TAG 1", @"EXIST TAG 2" ]];
```

***Replace all existing Tags***

```objc
[R1Push sharedInstance].tags.tags = @[ @"NEW TAG 1", @"NEW TAG 2" ];
```
or
```objc
[[R1Push sharedInstance].tags setTags:@[ @"NEW TAG 1", @"NEW TAG 2" ]];
```

***Get all Tags***

```objc
NSArray *currentTags = [R1Push sharedInstance].tags.tags;
```

###vi. Inbox Integration
If you want to enable inbox functionality, you need to use R1Inbox class and import *R1Inbox.h* header at the top of your class file:

```objc
#import "R1Inbox.h"
```

If you want to add some label or button with count of unread or total Inbox messages, you should implement `-(void) inboxMessageUnreadCountChanged` or `-(void) inboxMessagesDidChanged` methods from `R1InboxMessagesDelegate` protocol in your class. After that, add your class as delegate to `[R1Inbox sharedInstance].messages`

```objc
- (void) addInboxMessagesDelegate
{
	[[R1Inbox sharedInstance].messages addDelegate:self];
}

-(void) inboxMessageUnreadCountChanged
{
	NSString *btnTitle = [NSString stringWithFormat:@"Inbox (%lu unread)", (unsigned long)[R1Inbox sharedInstance].messages.unreadMessagesCount];
	
	[self.inboxButton setTitle:btnTitle forState:UIControlStateNormal];
}

```

Do not forget to remove your class from delegates when your class gets deallocated:

```objc
- (void) dealloc
{
    [[R1Inbox sharedInstance].messages removeDelegate:self];
    ...
}
```

To display the list of Inbox messages, you should create your own ViewController to provide required customization. In this case, this screen would not look foreign to your application.

You can see the sample of Inbox implementation in DemoApplication project in files *R1SampleInboxTableViewCell.h*, *R1SampleInboxTableViewCell.m*, *R1SampleInboxViewController.h*, *R1SampleInboxViewController.m*.

*R1SampleInboxTableViewCell.h*

```objc
#import <UIKit/UIKit.h>

@class R1InboxMessage;

@interface R1SampleInboxTableViewCell : UITableViewCell

// Sets or Gets R1InboxMessage object and configures cell for it
@property (nonatomic, strong) R1InboxMessage *inboxMessage;

- (id) initWithReuseIdentifier:(NSString *)reuseIdentifier;

// Calculates the height of the cell
+ (CGFloat) heightForCellWithInboxMessage:(R1InboxMessage *) inboxMessage cellWidth:(CGFloat) cellWidth;

@end
```

*R1SampleInboxTableViewCell.m*

```objc
#import "R1SampleInboxTableViewCell.h"
#import "R1Inbox.h"

@interface R1SampleInboxTableViewCell ()

@property (nonatomic, strong) UIView *unreadMarker;
@property (nonatomic, strong) UILabel *alertLabel;

@property (nonatomic, strong) NSDateFormatter *dateFormatter;

@end

@implementation R1SampleInboxTableViewCell

- (id) initWithReuseIdentifier:(NSString *)reuseIdentifier
{
    self = [super initWithStyle:UITableViewCellStyleSubtitle reuseIdentifier:reuseIdentifier];
    if (self)
    {
        self.alertLabel = [[UILabel alloc] initWithFrame:CGRectZero];
        self.alertLabel.numberOfLines = 0;
        self.alertLabel.lineBreakMode = NSLineBreakByWordWrapping;
        [self.contentView addSubview:self.alertLabel];
        
        self.unreadMarker = [[UIView alloc] initWithFrame:CGRectZero];
        self.unreadMarker.layer.cornerRadius = 3;
        self.unreadMarker.backgroundColor = [UIColor blueColor];
        [self.contentView addSubview:self.unreadMarker];
        
        self.textLabel.font = [UIFont boldSystemFontOfSize:14];
        self.alertLabel.font = [UIFont systemFontOfSize:14];
        
        self.detailTextLabel.textAlignment = NSTextAlignmentRight;
        
        self.dateFormatter = [[NSDateFormatter alloc] init];
        [self.dateFormatter setDateStyle:NSDateFormatterShortStyle];
        [self.dateFormatter setTimeStyle:NSDateFormatterMediumStyle];
    }
    return self;
}

// Configures the cell for displaying Inbox Message
- (void) setInboxMessage:(R1InboxMessage *)inboxMessage
{
    if (_inboxMessage == inboxMessage)
    {
        [self configureUnreadMarker];
        return;
    }
    
    _inboxMessage = inboxMessage;
    
    self.textLabel.text = inboxMessage.title;
    self.alertLabel.text = inboxMessage.alert;
    
    self.detailTextLabel.text = [self.dateFormatter stringFromDate:inboxMessage.createdDate];
    [self configureUnreadMarker];
    
    [self setNeedsLayout];
}

// Shows or hides unread marker view
- (void) configureUnreadMarker
{
    if ([self.unreadMarker isHidden] != _inboxMessage.unread)
        return;
    
    [self.unreadMarker setHidden:!_inboxMessage.unread];
}

- (void) layoutSubviews
{
    [super layoutSubviews];
    
    self.unreadMarker.frame = CGRectMake(4, (self.contentView.bounds.size.height - 6)/2, 6, 6);
    
    self.detailTextLabel.frame = CGRectMake(0, 0, self.contentView.bounds.size.width-10, 20);
    
    if (self.textLabel.text == nil)
    {
        self.alertLabel.frame = CGRectMake(15, 15, self.contentView.bounds.size.width-20, self.contentView.bounds.size.height-20);
    }else
    {
        self.textLabel.frame = CGRectMake(15, 15, self.contentView.bounds.size.width-20, 20);
        self.alertLabel.frame = CGRectMake(15, 35, self.contentView.bounds.size.width-20, self.contentView.bounds.size.height-45);
    }
}

// Calculates the height of the cell
+ (CGFloat) heightForCellWithInboxMessage:(R1InboxMessage *) inboxMessage cellWidth:(CGFloat) cellWidth
{
    CGFloat height = 25;
    
    if (inboxMessage.title != nil)
        height += 20;
    
    if (inboxMessage.alert != nil)
    {
        if ([inboxMessage.alert respondsToSelector:@selector(boundingRectWithSize:options:attributes:context:)])
        {
            NSMutableParagraphStyle *paragraph = [[NSMutableParagraphStyle alloc] init];
            paragraph.lineBreakMode = NSLineBreakByWordWrapping;
            
            NSDictionary *attributes = @{NSFontAttributeName : [UIFont systemFontOfSize:14],
                                         NSParagraphStyleAttributeName: paragraph};
            
            height += [inboxMessage.alert boundingRectWithSize:CGSizeMake(cellWidth-20, 100)
                                                       options:(NSStringDrawingUsesLineFragmentOrigin|NSStringDrawingUsesFontLeading)
                                                    attributes:attributes
                                                       context:nil].size.height;
        }else
        {
            height += [inboxMessage.alert sizeWithFont:[UIFont systemFontOfSize:14]
                                     constrainedToSize:CGSizeMake(cellWidth-20, 100)
                                         lineBreakMode:NSLineBreakByWordWrapping].height;
        }
        
        height += 1;
    }
    
    if (height < 50)
        return 50;
    
    return height;
}

@end
```
*R1SampleInboxViewController.h*

```objc
#import <UIKit/UIKit.h>
#import "R1Inbox.h"

@protocol R1SampleInboxViewControllerDelegate;

@interface R1SampleInboxViewController : UITableViewController <R1InboxMessagesDelegate>

@property (nonatomic, assign) id<R1SampleInboxViewControllerDelegate> inboxDelegate;

- (id) initInboxViewController;

@end

@protocol R1SampleInboxViewControllerDelegate <NSObject>

- (void) sampleInboxViewControllerDidFinished:(R1SampleInboxViewController *) sampleInboxViewController;

@end
```

*R1SampleInboxViewController.m*

```objc
#import "R1SampleInboxViewController.h"
#import "R1SampleInboxTableViewCell.h"
#import "R1Inbox.h"

@interface R1SampleInboxViewController ()

@property (nonatomic, strong) R1InboxMessages *inboxMessages;

@end

@implementation R1SampleInboxViewController

// Initialize UITableViewController
- (id) initInboxViewController
{
    self = [super initWithStyle:UITableViewStylePlain];
    if (self)
    {
        self.inboxMessages = [R1Inbox sharedInstance].messages;
        
        [self updateTitle];
        
        self.navigationItem.leftBarButtonItem = [[UIBarButtonItem alloc] initWithBarButtonSystemItem:UIBarButtonSystemItemDone
                                                                                              target:self action:@selector(closeInboxViewController)];
    }
    return self;
}

// Called when user presses 'Done' button
- (void) closeInboxViewController
{
    [self.inboxDelegate sampleInboxViewControllerDidFinished:self];
}

// Updates the title of ViewController with number of unread messages
- (void) updateTitle
{
    if (self.inboxMessages.unreadMessagesCount == 0)
        self.navigationItem.title = @"Inbox";
    else
        self.navigationItem.title = [NSString stringWithFormat:@"Inbox (%lu unread)", (unsigned long)self.inboxMessages.unreadMessagesCount];
}

- (void) viewWillAppear:(BOOL)animated
{
    [super viewWillAppear:animated];
    
    [self updateTitle];

    // Add this view controller to the list of delegates when it appears
    [self.inboxMessages addDelegate:self];
}

- (void) viewDidDisappear:(BOOL)animated
{
    [super viewDidDisappear:animated];
    
    // Remove this view controller from the list of delegates when it disappears
    [self.inboxMessages removeDelegate:self];
}

#pragma mark - Table view data source

- (NSInteger)numberOfSectionsInTableView:(UITableView *)tableView {
    return 1;
}

- (NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section
{
    // Returns the total number of Inbox messages
    return self.inboxMessages.messagesCount;
}

- (CGFloat)tableView:(UITableView *)tableView heightForRowAtIndexPath:(NSIndexPath *)indexPath
{
    // Returns the calculated height of the cell for R1InboxMessage object in row
    return [R1SampleInboxTableViewCell heightForCellWithInboxMessage:[self.inboxMessages.messages objectAtIndex:indexPath.row]
                                                           cellWidth:self.tableView.frame.size.width];
}

- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath
{
    R1SampleInboxTableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:@"Cell"];
    if (cell == nil)
    {
        cell = [[R1SampleInboxTableViewCell alloc] initWithReuseIdentifier:@"Cell"];
    }
    
    // Sets up the cell for displaying R1InboxMessage object
    cell.inboxMessage = [self.inboxMessages.messages objectAtIndex:indexPath.row];
    
    return cell;
}

- (void) tableView:(UITableView *)tableView didSelectRowAtIndexPath:(NSIndexPath *)indexPath
{
    R1InboxMessage *message = [self.inboxMessages.messages objectAtIndex:indexPath.row];

    // Shows Inbox message when user interacts to the cell
    [[R1Inbox sharedInstance] showMessage:message
                           messageDidShow:^{
                               [self.tableView deselectRowAtIndexPath:indexPath animated:YES];
                           }];
}

- (UITableViewCellEditingStyle)tableView:(UITableView *)tableView editingStyleForRowAtIndexPath:(NSIndexPath *)indexPath
{
    return UITableViewCellEditingStyleDelete;
}

- (void)tableView:(UITableView *)tableView commitEditingStyle:(UITableViewCellEditingStyle)editingStyle forRowAtIndexPath:(NSIndexPath *)indexPath
{
    R1InboxMessage *message = [self.inboxMessages.messages objectAtIndex:indexPath.row];
    
    [self.inboxMessages deleteMessage:message];
}

#pragma mark - R1InboxMessagesDelegate methods

// This method called when the list of Inbox messages gets updated
- (void) inboxMessagesWillChanged
{
    [self.tableView beginUpdates];
}

// This method called when any item in the list of Inbox messages gets changed (modified, inserted or removed)
- (void) inboxMessagesDidChangeMessage:(R1InboxMessage *) inboxMessage
                               atIndex:(NSUInteger) index
                         forChangeType:(R1InboxMessagesChangeType)changeType
                              newIndex:(NSUInteger) newIndex
{
    switch (changeType)
    {
        case R1InboxMessagesChangeInsert:
            [self.tableView insertRowsAtIndexPaths:@[ [NSIndexPath indexPathForRow:index inSection:0] ]
                                  withRowAnimation:UITableViewRowAnimationAutomatic];
            break;
        case R1InboxMessagesChangeDelete:
            [self.tableView deleteRowsAtIndexPaths:@[ [NSIndexPath indexPathForRow:index inSection:0] ]
                                  withRowAnimation:UITableViewRowAnimationAutomatic];
            break;
        case R1InboxMessagesChangeUpdate:
            ((R1SampleInboxTableViewCell *)[self.tableView cellForRowAtIndexPath: [NSIndexPath indexPathForRow:index inSection:0] ]).inboxMessage = [self.inboxMessages.messages objectAtIndex:index];

            break;
            
        default:
            break;
    }
}

// This method called when the changes to list of Inbox messages are over
- (void) inboxMessagesDidChanged
{
    [self.tableView endUpdates];
}

// This method called when the number of unread Inbox messages gets changed
- (void) inboxMessageUnreadCountChanged
{
    [self updateTitle];
}

@end
```

In this example, initialize the R1SampleInboxViewController with the following method:

 ```objc
 [[R1SampleInboxViewController alloc] initInboxViewController]
  ```

##d. Attribution Tracking Activation
###i. Track RadiumOne Campaigns
Please contact your Account Manager to setup R1 ad campaigns as well as tracking campaigns.  If you don't have one, please contact us [here](http://radiumone.com/contact-mobile-team.html) and one of our Account Managers will assist you.

Once your Account Manager has set up tracking, you will start receiving attribution tracking report automatically!

###ii. Track 3rd party Campaigns
1. Please contact your Account Manager to setup tracking URLs for your 3rd party campaigns.  If you don't have one, please contact us [here](http://radiumone.com/contact-mobile-team.html) and one of our Account Managers will assist you.
2. Send the list of all your media suppliers (anyone you run a mobile advertising campaign with).
3. For each media supplier, your account manager will send you 2 tracking URLs (one impression tracking URL, one click tracking URL).
4. Send each pair of URLs to the relevant Media Supplier so they can set these tracking URLs on the creatives.
5. You're all set and will start having access to Attribution Tracking Reports.

##e. Geofencing Activation

````
#import "R1GeofencingSDK.h"
````

Geofencing is disabled by default.  You can enable it in the `application:didFinishLaunchingWithOptions:` method or later.

sdk.geofencingEnabled = YES;

To disable geofencing, either remove the above call or set its value to NO

The R1GeofencingSDK also allows you to notify your users and drive engagements
via local notification. When a user `entered` or `exited` a region (both
    `CLGeographicalRegion` and `CLBeaconRegion`) you can obtain relevant information
using the `NSNotification` object, and the notification object has the following
keys to access the `CLRegion` object and its name string respectively:
````
kR1LocationRegionObjectKey
kR1LocationRegionNameKey
````
You can register region enter/exit notifications as needed as shown below:
````
[[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(sendLocalEnterNotification:) name:kR1GeofenceDidEnterNotification object:nil];
[[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(sendLocalExitNotification:) name:kR1GeofenceDidExitNotification object:nil];
````
In your `sendLocalEnterNotification:` and `sendLocalExitNotification:` methods,
   you can relay your event messages to `application:didReceiveLocalNotification:`
   by overriding it on your application delegate to display these local notifications using your own keys. For example:


   ```objc
   - (void)application:(UIApplication *)application didReceiveLocalNotification:(UILocalNotification *)notification
{
  if ([UIApplication sharedApplication].applicationState == UIApplicationStateActive) {
    // you may want to show an alert

    NSString *alertString = [notification.userInfo objectForKey:<Your_Own_NotificationAlertBodyKey>];
    application.applicationIconBadgeNumber = 0; // reset the badge to zero

    NSString *alertTitle = [notification.userInfo objectForKey:<Your__Own_NotificationAlertTypeKey>];

    UIAlertView *alert = [[UIAlertView alloc] initWithTitle:alertTitle message:alertString delegate:self cancelButtonTitle:@"OK" otherButtonTitles:nil, nil];
    [alert show];
  }
}

```

N.B. - The Connect locationService uses the Location Manager in iOS.  For
deployment on iOS 8 and newer, it is required that the application's property
list (plist) file includes one of the two following keys:

NSLocationAlwaysUsageDescription
//provide location updates whether the user is actively using the application or
not via infrequent background location updates
// OR
NSLocationWhenInUseUsageDescription
//provide location updates only while the user has your application as the
foreground running app

These are string values that need to include a description suitable to present
to a user.  The description should explain the reason the application is
requesting access to the user's location.  Therefore, description you give
should try to incorporate or reference the user benefit that may be possible
through sharing location.

#4. Submitting your App

When preparing to send your binary to Apple, you will set up an application target in the iTunes Connect portal (http://itunesconnect.apple.com for details).  During this process, you will be presented with the question, "Does this app use the Advertising Identifier (IDFA)?"

  ![Image of idfaCheck]
(Doc_Images/idfaCheck.png)

Your application may or may not be using this value for your own purposes, but the Connect SDK does access it (described below). So, it is required that you answer, "Yes" to the aforementioned question.

If your application is utilizing Connect's analytics, geofencing or push notification features, be sure to check the last use case option - that the application uses the IDFA to "Attribute an action taken within this app to a previously served advertisement" as advertisements that you might have served can be related to users actions within your app.

  ![Image of idfaAnalyticsOption]
(Doc_Images/idfaAnalyticsOption.png)

If your application is also using Connect's Engage (display advertisements) feature, be sure to select the option: "Serve advertisements within the app". Naturally, if your application is only using the Engage functionality, leave all other options unchecked (as related to Connect's use of the IDFA)

  ![Image of idfaAllOptions]
(Doc_Images/idfaAllOptions.png)


You will also need to confirm that your app honors a user's "Limit Ad Tracking" setting in iOS.  The Connect SDK does honor this flag and will not access or otherwise utilize the IDFA value if the user has selected the "Limit Ad Tracking" feature.  Ensure that this confirmation and the previously mentioned IDFA use options are checked to facilitate a smooth application review process.
