#Table of Contents
- [1. System Requirements](#user-content-1-system-requirements)
- [2. SDK Initialization](#user-content-2-sdk-initialization)
  - [a. Import Files](#user-content-a-import-files)
  - [b. Link the Static Library](#user-content-b-link-the-static-library)
  - [c. Initialize the SDK](#user-content-c-initialize-the-sdk)
  - [d. Advanced Settings](#user-content-d-advanced-settings)
- [3. Feature Activation](#user-content-3-feature-activation)
  - [a. Engage Activation (optional)](#user-content-a-engage-activation)
  - [b. Analytics Activation (optional)](#user-content-b-analytics-activation)
    - [i. Settings](#user-content-i-settings)
    - [ii. Automatic Events](#user-content-ii-automatic-events)
    - [iii. Standard Events](#user-content-iii-standard-events)
    - [iv. Custom Events](#user-content-iv-custom-events)
    - [v. Best Practices](#user-content-v-best-practices)
  - [c. Push Notification Activation (optional)](#user-content-c-push-notification-activation)
    - [i. Initialization](#user-content-i-initialization)
    - [ii. Setup Apple Push Notification Services](#user-content-ii-setup-apple-push-notification-services)
    - [iii. Segment your Audience](#user-content-iii-segment-your-audience)
  - [d. Attribution Tracking Activation (optional)](#user-content-d-attribution-tracking-activation)
    - [i. Track RadiumOne Campaigns](#user-content-i-track-radiumone-campaigns)
    - [ii. Track 3rd party Campaigns](#user-content-ii-track-3rd-party-campaigns)
  - [e. Geofencing Activation (optional)](#user-content-e-geofecing-activation)

#1. System Requirements
The R1ConnectEngage SDK supports all mobile and tablet devices running iOS 6.0 or newer. The downloadable directory (see below "[a. Import Files](#a-import-files)") contains the library and headers for the R1ConnectEngage SDK. 

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
(https://github.com/radiumone/Social-iOS/blob/d59fd6c2070a5a08df0e0397d62f5e973fd5be61/Doc_Images/addfiles.png)

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

See image below:


![Image of framework]
(https://github.com/radiumone/Social-iOS/blob/53e01df902e8dafe3db302e814aa8c29e596c27c/Doc_Images/framework.png)


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

R1ConnectEngage SDK supports iOS 6.0 and above. It supports full-screen products (Offerwall, Interstitial, Video) and Banners.

#### SDK Initialization

You will need to enable Engage in your App Delegate. In your `application: didFinishLaunchingWithOptions:` method insert the following code:
```objc	
R1SDK *sdk = [R1SDK sharedInstance];
sdk.engageEnabled = YES;
```

#### Integration of full-screen views (Offerwall, Interstitial, Video)

Engage has three full-screen view controllers: 

* R1EngageOfferwallViewController
* R1EngageInterstitialViewController
* R1EngageVideoViewController

As an example, the following demonstrates an integration of the offerwall view. Other
Engage full-screen views are integrated in the same way.

####Setup

Set up your interface file ([Your view controller].h) with `offerwallViewController`

```objc
#import <UIKit/UIKit.h>
#import "R1EngageSDK.h"

@interface ViewController : UIViewController<R1EngageContentViewControllerDelegate>

@property (nonatomic, strong) R1EngageOfferwallViewController *offerwallViewController;

@end
```

####Block-based Full-screen View Handling

There are two different approaches to using the full-screen views:

1. Using a block-based API
2. Using delegate-based API with callback methods

Using the block-based API gives you lower level control to implement error handling and display of the full-screen view.

```objc
- (void)loadViewController
{
  self.offerwallViewController = [R1EngageOfferwallViewController viewController];
  // Also you can setup other optional info in any place of application
  [R1EngageSDK sharedInstance].userId = @"OPTIONAL USER ID";
  [R1EngageSDK sharedInstance].trackId = @"OPTIONAL TRACK ID";
  self.offerwallViewController.delegate = self;
  [self.offerwallViewController load:^(R1EngageLoadingResult result, NSError *loadError) {
        switch (result)
        {
          case R1EngageLoadingResultHasOffers:
          // server has offer(s), your code here to show the offerwall controller
          // Note - showFromViewController is the required method for displaying
          // the loaded full-screen view.
            [self.offerwallViewController showFromViewController:self];
          break;

          case R1EngageLoadingResultNoOffers:

          break;
          case R1EngageLoadingResultError:

          break;

          default:
          break;
        }
  }];
 }
```	

In the above code replace:
* \<Optional User ID\> with your own user ID (if you remove the line we’ll use the iOS Advertising Identifier in its place.
* \<Optional Track ID\> with a custom value for your own tracking needs (we will save and return back up to 100 characters of it).

If not using the block-based method, the following convenience method is provided to automatically show available offers, but will fail silently if an error occurs or if no offers are found.  However, use of the aforementioned block-based method is recommended.

```objc
- (void)loadAndShowViewController
{
  self.offerwallViewController = [R1EngageOfferwallViewController viewController];
  self.offerwallViewController.delegate = self;
  [self.offerwallViewController loadAndShowFromViewController:self];
}
```

####Delegate-based Full-screen View Handling

Setup for Full-screen View handling is as easy as instantiating the desired view controller, setting the delegate and then calling the load method on the target view controller.

```objc
self.offerwallViewController = [R1EngageOfferwallViewController viewController];
self.offerwallViewController.delegate = self;
[self.offerwallViewController load:nil];
```

The advertising loading process results in three events:

1. No errors, at least one offer will be shown
2. No errors, but no offers found
3. Some error occurred

You can respond to these situations by setting an appropriate delegate for the EngageContentViewController and implementing the `R1EngageContentViewControllerDelegate` methods.

```objc
- (void)engageContentViewControllerDidStartLoading:(R1EngageContentViewController *)engageContentViewController
{
  // fullscreen controller did start loading
}

- (void)engageContentViewController:(R1EngageContentViewController *)engageContentViewController didFinishLoadingWithResult:(R1EngageLoadingResult)loadResult loadError:(NSError *)loadError
{
  // fullscreen controller did finish loading
  switch (loadResult)
  {
    case R1EngageLoadingResultHasOffers:
    // has offer

    break;
    
    case R1EngageLoadingResultNoOffers:
    break;

    case R1EngageLoadingResultError:
    break;
    
    default:
    break;
  }
}
```

You can also respond to the appearing/disappearing of the offerwall using these: 

```objc
- (void)engageContentViewControllerWillAppear:(R1EngageContentViewController *)engageContentViewController;
- (void)engageContentViewControllerDidAppear:(R1EngageContentViewController *)engageContentViewController;
- (void)engageContentViewControllerWillDisappear:(R1EngageContentViewController *)engageContentViewController;
- (void)engageContentViewControllerDidDisappear:(R1EngageContentViewController *)engageContentViewController;
```

####Integration of Banner


Engage supports several predefined banner classes:

* R1Engage300x50BannerView for 300x50
* R1Engage320x50BannerView for 320x50
* R1Engage300x250BannerView for 300x250
* R1Engage728x90BannerView for 728x90
* R1Engage480x50BannerView for 480x50
* R1Engage1024x90BannerView for 1024x90

Alternately, you can use R1EngageCustomBannerView class if you would like to specifiy a custom banner size.

####Setup with Interface Builder

Set up your Interface file ([Your view controller].h) with “adView” as an outlet:

```objc
#import <UIKit/UIKit.h>
#import "R1EngageSDK.h"
@interface PDBannerViewController : UIViewController

@property (nonatomic, weak) IBOutlet R1Engage320x50BannerView *bannerView;

@end
```

Set up your implementation file([Your view controller].m):

```objc
- (void)viewDidLoad
{
  [super viewDidLoad];

  [self.bannerView load:^(R1EngageLoadingResult result, NSError *loadError) {
      case R1EngageLoadingResultHasOffers:
      // You can add any appropriate actions or logic here in response to the ad
      // succcessfully loading. e.g., you could unhide the bannerView if you had
      // set it up as initially hidden, or animate it in some useful way.
      break;

      case R1EngageLoadingResultNoOffers:

      break;
      case R1EngageLoadingResultError:

      break;

      default:
      break;
  }];
}
```

Set up your Interface Builder file ([Your view controller].xib):

1. Add new view into main view
2. Set up class for this view

    ![Image of ibclass]
    (https://github.com/radiumone/Social-iOS/blob/25c134816a94a5e00a5aab2c483c9247891457a0/Doc_Images/ibclass.png)

3. Set up ad view size and layout

    ![Image of iblayout]
    (https://github.com/radiumone/Social-iOS/blob/25c134816a94a5e00a5aab2c483c9247891457a0/Doc_Images/iblayout.png)

4. Connect this view with bannerView object

    ![Image of ibconnection]
    (https://github.com/radiumone/Social-iOS/blob/25c134816a94a5e00a5aab2c483c9247891457a0/Doc_Images/ibconnection.png)

####Setup without Interface Builder

Set up your Interface file ([Your view controller].h) with “adView” as an outlet:

```objc
#import <UIKit/UIKit.h>
#import "R1EngageSDK.h"
@interface PDBannerViewController : UIViewController

@property (nonatomic, strong) R1Engage320x50BannerView *bannerView;

@end
```

Set up your implementation file ([Your view controller].m):

```objc
- (void)viewDidLoad
{
  [super viewDidLoad];
  
  self.bannerView = [[R1Engage320x50BannerView alloc] initWithFrame:CGRectMake(0, 0, 320, 50)];
  [self.view addSubview:self.bannerView];
  
  [self.bannerView load:^(R1EngageLoadingResult result, NSError *loadError) {
      case R1EngageLoadingResultHasOffers:
      // You can add any appropriate actions or logic here in response to the ad
      // succcessfully loading. e.g., you could unhide the bannerView if you had
      // set it up as initially hidden, or animate it in some useful way.
      break;

      case R1EngageLoadingResultNoOffers:

      break;
      case R1EngageLoadingResultError:

      break;

      default:
      break;
  }];
}
```

####Delegate-based Banner Handling

After initializing your desired BannerView, set an appropriate delegate and call the 'load' method.

```objc
// Just as with the block-based example, we allocate and add the view to the view hierarchy
self.bannerView = [[R1Engage320x50BannerView alloc] initWithFrame:CGRectMake(0, 0, 320, 50)];
[self.view addSubview:self.bannerView];

// then set up the delegate and initiate the advertisement loading
self.bannerView.delegate = self;
[self.bannerView load:nil];
```

You can respond to banner loading states by setting an appropriate delegate for the BannerView and implementing the `R1EngageBannerViewDelegate` methods.

```objc
- (void)engageBannerViewDidStartLoading:(R1EngageBannerView *)engageBannerView;
- (void)engageBannerView:(R1EngageBannerView *)engageBannerView didFinishLoadingWithResult:(R1EngageLoadingResult) loadResult loadError:(NSError *)loadError;
```

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
One common mistake is to parametrize event names (with user data for example). Event names should be hard-coded values that you use to segement data on a specific category of event. 

Example: "ProfileViewing"

Avoid: "Profile Viewing - Lady Gaga's profile"

As you may have thousands of user profiles in your database, it is preferable to keep the event name high level ("ProfileViewing") so you can run interesting anaytics on it. High level events help answer questions like "how many profiles does a user visit every day on average?" 

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

###i. Initialization

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
```
 
Push is disabled by default. You can enable it in the *application:didFinishLaunchingWithOptions* method or later.

```objc
[[R1Push sharedInstance] setPushEnabled:YES];
```


NOTE: If you enabled it in the *application:didFinishLaunchingWithOptions* method, the Push Notification AlertView will be showed at first application start.


###ii. Setup Apple Push Notification Services

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


###iii. Segment your Audience    

You can specify Tags for *R1ConnectEngage SDK* to send *Push Notifications* to certain groups of users.

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

