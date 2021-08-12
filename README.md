# Open_Mediation SDK_Integration_Document

## OpenMediation SDK Integration Document

> Before You Start
>We support Android Operating Systems Version 4.1 (API Level 16) and above. Be sure to:

>Use Android Studio 3.0 and above
>Target Android API level 28
>minSdkVersion level 16 and above

#### 1. Overview

This Guideline introduces the integration of OpenMediation's SDK in Android Apps.

#### 2. Ads ID

This section introduces how to fetch and use identity information when integrating the OpenMediation SDK：APP_KEY and Placement ID. Please contact your account manager to get APP_KEY and Placement ID.

APP_KEY: APP_KEY is the sole identification of developer’s application on OpenMediation platform. Placement ID: identifies the ads placement at OpenMediation.

#### 3. Add OpenMediation SDK to your Project

Add the following to your project-level build.gradle file.


```
allprojects {
    repositories {
        google()
        jcenter()
        maven { url 'https://dl.openmediation.com/omcenter/' }
    }
}
```
Add the following to your application-level build.gradle file inside dependencies section.


```
implementation 'com.openmediation:om-android-sdk:2.3.2'
```

**Cloned GitHub Repository**

Alternatively, you can obtain the OpenMediation SDK source and Sample App form the GitHub Repository:
```
git clone git://github.com/AdTiming/OpenMediation-Android.git
```

#### 4. Update AndroidManifest.xml

Configure your AndroidManifest.xml to add the following permissions into the <manifest> tag but outside the <application> tag.

```
<!-- Required permissions -->
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
```

#### 5. Proguard

You must add the following to your Proguard configuration file (Android Studio: proguard-rules.pro or Eclipse: proguard-project.txt) if you are using Porguard in your application. Failed to do so will lead to errors.

```
-dontwarn com.openmediation.sdk.**.*
-dontskipnonpubliclibraryclasses
-keep class com.openmediation.sdk.**{*;}
#R
-keepclassmembers class **.R$* {
  public static <fields>;
}
-keepattributes *Annotation*
-keepattributes InnerClasses
-keepnames class * implements android.os.Parcelable {
  public static final ** CREATOR;
}
```

#### 6. MultiDex

If your project has a multDex operation，Add the following to your build.gradle file.

```
android {
  buildTypes {
    release {
      multiDexKeepProguard file('multidex-config.pro')
      ...
    }
  }
}
```
** multidex-config.pro **
```
-keep class com.openmediation.sdk.**{*;}
-dontwarn com.openmediation.sdk.**.*
```

#### 7. Override Your Activity Lifecycle Methods

Override the onPause(), onResume() methods in each of your activities to call the corresponding OpenMediation methods as below:

```
protected void onResume() {
     super.onResume();
     OmAds.onResume(this);
  }
protected void onPause() {
     super.onPause();
     OmAds.onPause(this);
  }
```

#### 8. Initialization

Before you can fetch ads from OpenMediation Platform, you should first initialize the OpenMediation SDK when your app started. To do so, we suggest to call OpenMediation SDK's init() method within onCreate() of an Application or Activity as below:

```
import com.openmediation.sdk.InitConfiguration;
import com.openmediation.sdk.OmAds;
import com.openmediation.sdk.InitCallback;
import com.openmediation.sdk.utils.error.Error;
...

InitConfiguration configuration = new InitConfiguration.Builder()
          .appKey("Your AppKey")
          .logEnable(false)
          .build();
OmAds.init(configuration, new InitCallback() {

    // Invoked when the initialization is successful.
    @Override
    public void onSuccess() {
    }

    // Invoked when the initialization is failed.
    @Override
    public void onError(Error error) {
    }
});
```
>**Note:**
APP KEY can only be obtained by creating apps on OpenMediation.
We suggest that the first call to method loadAd after the app starts should be in the onSuccess callback.
No need to invoke loadAd method for Rewarded Video & Interstitial Ads anymore, the SDK will fetch Ads automatic and maintain the ad inventory.
The error parameter in onError callback describes the cause of failure. Check the  Error Code for more information and guidance on errors.

#### Best Practice: Init the SDK with AdUnits

The SDK can be initialized in another way and we recommend this approach as it will only fetch the specific ad units you define in the adUnits parameter. The adUnits parameter is a string array. The following illustrates that the SDK is initiated with Rewarded_Video&Interstitial AdUnits and only these two types Ads will be fetched automatic.

```
// Ad Units should be in the type of OmAds.AD_TYPE.AdUnitName, for example:
InitConfiguration configuration = new InitConfiguration.Builder()
    .appKey("Your AppKey")
    .preloadAdTypes(OmAds.AD_TYPE.INTERSTITIAL, OmAds.AD_TYPE.REWARDED_VIDEO)
    .build();
OmAds.init(configuration, callback);
```
When using this init approach, you can initialize each ad unit separately at different touchpoints in your app flow in one session.
```
// Init with pre-load Rewarded video ads
InitConfiguration configuration = new InitConfiguration.Builder()
    .appKey("Your AppKey")
    .preloadAdTypes(OmAds.AD_TYPE.REWARDED_VIDEO)
    .build();
OmAds.init(configuration, callback);
// Init with pre-load Interstitial
InitConfiguration configuration = new InitConfiguration.Builder()
    .appKey("Your AppKey")
    .preloadAdTypes(OmAds.AD_TYPE.INTERSTITIAL)
    .build();
OmAds.init(configuration, callback);
```

> If no parameters are passed in the type, it means that the Rewarded Video and Interstitial ad types will be preloaded by default
> If you don’t want to do any preloading, please pass in the OmAds.AD_TYPE.NONE parameter

```
// Init with no pre-load
InitConfiguration configuration = new InitConfiguration.Builder()
    .appKey("Your AppKey")
    .preloadAdTypes(OmAds.AD_TYPE.NONE)
    .build();
OmAds.init(configuration, callback);
```
####  Implement the Callback

The OpenMediation SDK fires several events to inform you of your initialing SDK operation. To receive these events, register to the callback in the init() and implement the onSuccess() and onError() of it.

```
@Override
public void onSuccess() {
    // Add code here to process init success event.
}
@Override
public void onError(Error error) {
    // Add code here to process init failed event.
    // Parameter message tells the error information.
}
```

#### Report Custom User Identifier

The APP can report the customized device identifier through the SDK, and only needs to call the setUserId method to set it before initialization. This identifier will be reflected in user-level data, such as UAR report.
```
OmAds.setUserId(String userId);
```

#### 9. Add monetize SDK into the mediation

Add the following to your application-level **build.gradle** file inside **dependencies** section.

```groovy
dependencies{
	implementation 'com.flatads.sdk:flatads:1.1.16'
	implementation 'com.adtiming:adnetwork:6.9.1'
	implementation 'com.mbridge.msdk.oversea:videojs:15.5.31'
    implementation 'com.mbridge.msdk.oversea:mbjscommon:15.5.31'
    implementation 'com.mbridge.msdk.oversea:playercommon:15.5.31'
    implementation 'com.mbridge.msdk.oversea:reward:15.5.31'
    implementation 'com.mbridge.msdk.oversea:videocommon:15.5.31'
    implementation 'com.mbridge.msdk.oversea:same:15.5.31'
    implementation 'com.mbridge.msdk.oversea:interstitialvideo:15.5.31'
    implementation 'com.mbridge.msdk.oversea:mbbanner:15.5.31'
    // for using bidding
    implementation 'com.mbridge.msdk.oversea:mbbid:15.5.31'
}
```

Add the following to your project-level **build.gradle** file inside **repositories** section.

```groovy
allprojects {
    repositories {
        jcenter()
        google()
        maven {url "http://maven.flat-ads.com/repository/maven-public/"}
        maven {url 'https://dl.openmediation.com/omcenter/'}
        maven {url "https://dl.adtiming.com/android-sdk"}
        maven { url "https://dl-maven-android.mintegral.com/repository/mbridge_android_sdk_oversea"}

    }
}
```
**For Proguard Users Only**

```groovy
-keep class com.flatads.sdk.response.* {*;}
-keep class com.adtbid.sdk.** { *;}
-dontwarn com.adtbid.sdk.**
-keepattributes Signature
-keepattributes *Annotation*
-keep class com.mbridge.** {*; }
-keep interface com.mbridge.** {*; }
-keep interface androidx.** { *; }
-keep class androidx.** { *; }
-keep public class * extends androidx.** { *; }
-dontwarn com.mbridge.**
-keep class **.R$* { public static final int mbridge*; }
```
#### Add Adapter
Add the following in your application-level build.gradle file inside dependencies section.

```groovy
implementation 'com.openmediation.adapters:flatads:2.3.3'
implementation 'com.openmediation.adapters:adtiming:2.3.1'
implementation 'com.openmediation.adapters:mintegral:2.3.0'

```


#### 10. Splash Ad
The Splash Ad takes APP launch as the exposure opportunity, and provides 3s~5s of advertising display time. The user can click the ad to jump to the target page, or click the "skip" button in the upper right corner to jump to the APP content homepage.


**Step 1. Set the Splash Ad Listener**

The OpenMediation SDK fires several events to inform you of Promotion Ad activity, such as ad availability and completions, so you will know whether and when to reward your users. To serve Promotion Ad, you need to first set its listeners and process ad events. The following snippet demonstrates how to implement the SplashAdListener interface to receive video ad events.

The SDK will notify the listener of all possible events listed below:

```
import com.openmediation.sdk.splash.SplashAd;
import com.openmediation.sdk.splash.SplashAdListener;
import com.openmediation.sdk.utils.error.Error;
...
SplashAd.setSplashAdListener(placementId, new SplashAdListener() {

    /**
     * called when SplashAd loaded
     */
    @Override
    public void onSplashAdLoaded(String placementId) {
    }

    /**
     * called when SplashAd load error
     */
    @Override
    public void onSplashAdFailed(String placementId, Error error) {
    }

    /**
     * called when SplashAd clicked
     */
    @Override
    public void onSplashAdClicked(String placementId) {
    }

    /**
     * called when SplashAd showed
     */
    @Override
    public void onSplashAdShowed(String placementId) {
    }

    /**
     * called when SplashAd show failed
     *
     * @param error SplashAd show error reason
     */
    @Override
    public void onSplashAdShowFailed(String placementId, Error error) {
    }

    /**
     * called when SplashAd countdown
     * @param millisUntilFinished The time until the end of the countdown,ms
     */
    @Override
    public void onSplashAdTick(String placementId, long millisUntilFinished) {
    }

    /**
     * called when SplashAd dismissed
     */
    @Override
    public void onSplashAdDismissed(String placementId) {
    }
});
```
**Step 2. Load the Promotion Ad**
```
SplashAd.loadAd(String placementId);
```
**Step 3. Show the Splash Ad**
```
if (SplashAd.isReady(placementId)) {
    SplashAd.showAd(placementId);
}
```

#### 11. Rewarded Video

**Step 1. Set the Rewarded Video Listener**

The OpenMediation SDK fires several events to inform you of Rewarded Video Ad activity, such as ad availability and completions, so you will know whether and when to reward your users. To serve Rewarded Video Ad, you need to first set its listeners and process ad events. The following snippet demonstrates how to implement the RewardedVideoListener interface to receive video ad events.
The SDK will notify the listener of all possible events listed below:

```
import com.openmediation.sdk.utils.error.Error;
import com.openmediation.sdk.utils.model.Scene;
import com.openmediation.sdk.video.RewardedVideoAd;
import com.openmediation.sdk.video.RewardedVideoListener;
...
RewardedVideoAd.setAdListener(new RewardedVideoListener() {

    /**
     * Invoked when the ad availability status is changed.
     *
     * @param available is a boolean.
     *      True: means the rewarded videos is available and
     *          you can show the video by calling RewardedVideoAd.showAd().
     *      False: means no videos are available
     */
    @Override
    public void onRewardedVideoAvailabilityChanged(boolean available) {
        // Change the rewarded video state according to availability in app.
        // You could show ad right after it's was loaded here
    }

    /**
     * Invoked when the RewardedVideo ad view has opened.
     * Your activity will lose focus.
     */
    @Override
    public void onRewardedVideoAdShowed(Scene scene) {
        // Do not perform heavy tasks till the video ad is going to be closed.
    }

    /**
     * Invoked when the call to show a rewarded video has failed
     * @param error contains the reason for the failure:
     */
    @Override
    public void onRewardedVideoAdShowFailed(Scene scene, Error error) {
        // Video Ad show failed
    }

    /**
     * Invoked when the user clicked on the RewardedVideo ad.
     */
    @Override
    public void onRewardedVideoAdClicked(Scene scene) {
        // Video Ad is clicked
    }

    /**
     * Invoked when the RewardedVideo ad is closed.
     * Your activity will regain focus.
     */
    @Override
    public void onRewardedVideoAdClosed(Scene scene) {
        // Video Ad Closed
    }

    /**
     * Invoked when the RewardedVideo ad start to play.
     * NOTE:You may not receive this callback on some AdNetworks.
     */
    @Override
    public void onRewardedVideoAdStarted(Scene scene) {
        // Video Ad Started
    }

    /**
     * Invoked when the RewardedVideo ad play end.
     * NOTE:You may not receive this callback on some AdNetworks.
     */
    @Override
    public void onRewardedVideoAdEnded(Scene scene) {
        // Video Ad play end
    }

    /**
     * Invoked when the video is completed and the user should be rewarded.
     * If using server-to-server callbacks you may ignore this events and wait
     * for the callback from the OpenMediation server.
     */
    @Override
    public void onRewardedVideoAdRewarded(Scene scene) {
        // Here you can reward the user according to your in-app settings.
    }
});
```

**Step 2. Serve Rewarded Video Ad**

Ad Availability

OpenMediation SDK automatic loads ads for you to cache the Rewarded Video Ads during application lifecycle if only SDK is integrated and initiated successfully. By correctly implementing the RewardedVideoListener , you will be notified about the ad availability through the onRewardedVideoAvailabilityChanged callback.
```
public void onRewardedVideoAvailabilityChanged(boolean available)
```
Another way to check if the ad is available is by calling the isReady() function directly.
```
public boolean isReady()
```

Show Video Ad

Once you receive the onRewardedVideoAvailabilityChanged callback, you can invoke the showAd() method to serve a Rewarded Video Ad to your users.
```
public void showAd(String sceneName)
```

>**Note：**
Scene is optional，if you don't want to use it just ignore the sceneName parameter or use value "" as below.

```
//Use empty String as value if you don't need this functionality.
 RewardedVideoAd.showAd("")
 //Or ignore it.
 RewardedVideoAd.showAd()
```

We do not recommend showing ads in onRewardedVideoAvailabilityChanged callback directly, the Availability event only occurs when the ad availability changes, true or false, which does not necessarily mean it is the right time to serve ad. And showing ads in this callback unrestricted may result in frequent or even continuous ad pop-up, which will cause confusion for users and affect the experience. You should choose when your ads will show based on your application's ad plan.

```
//if you would like to show ad right after it's was loaded
public void onRewardedVideoAvailabilityChanged(boolean available) {
    if(available) {
        RewardedVideoAd.showAd(sceneName)
    }
}
```

>** Warning!**
Showing ads in onRewardedVideoAvailabilityChanged callback can cause unforeseen ad pop-ups. You should not do this unless it is happened only in a specific scenario and the necessary restrictions have been imposed on the invoking to showAd.

We strongly recommend checking the ad's availability by isReady method before you show Rewarded Video Ad to your users, as shown in the following code:
```
//if you would like to show ad when it's required
if (RewardedVideoAd.isReady()) {
    RewardedVideoAd.showAd(sceneName);
}
```
>**Notes:**
When you have successfully completed step 2, you will have shown a Rewarded Video Ad to your users. If you want to serve another Rewarded Video Ad, you just repeat step 2 to check ad availability and show a new Rewarded Video Ad.


**Step 3. Reward the User**

The OpenMediation SDK will fire the onRewardedVideoAdRewarded event each time the user successfully completes a video. The RewardedVideoListener will be in place to receive this event so you can reward the user successfully.

>**Note:**
Make sure to set up your listener to grant rewards event in cases where onRewardedVideoAdRewarded is fired after the onRewardedVideoAdClosed event, because the onRewardedVideoAdRewarded and onRewardedVideoAdClosed events are asynchronous.

Server-to-Server Callbacks

OpenMediation SDK supports the rewarded video ad Server-side Callbacks. You can do this by defining a unique, private endpoint for us , and specify information that you want the OpenMediation server to pass. The way to works is the app on a user’s device will contact OpenMediation servers for validation of a reward. Once a user has been validated by the OpenMediation server, we will send notification to the endpoint defined in your dashboard. Since a user’s device will never be in contact with your server, this method prevents tampering with reward events sent to your endpoint.

Video ExtId

If your Rewarded Video Ad requires server-to-server data interoperability, you need to call setExtId to pass your user identity as below. This parameter is used to verify Rewarded transactions and must be set before calling showAd.

```
RewardedVideoAd.setExtId(scene, "Your Ext Id");
```

Configure Rewarded Video Ad Server-side Callback.

Please contact your account manager to config Rewarded Video Ad Server-side Callback.

Endpoint Format

http://yourendpoint.com?variable_name_you_define={content}

{content} - the information that you want the OpenMediation server to pass. The user identity pass to setExtId will be set to {content}.

#### 12. Interstitial
The OpenMediation Interstitial is a full-screen ad unit, usually served at natural transition points during an app's lifecycle. Both static and video interstitials are supported.

**Step 1. Set the Interstitial Ad Listener**

The OpenMediation SDK fires several events to inform you of Interstitial Ad activity, such as ad availability and clicked. To serve Interstitial Ad, you need to first set its listeners and process ad events. The following snippet demonstrates how to implement the InterstitialAdListener interface to receive interstitial ad events.

The SDK will notify the Listener of all possible events listed below:

```
import com.openmediation.sdk.interstitial.InterstitialAd;
import com.openmediation.sdk.interstitial.InterstitialAdListener;
import com.openmediation.sdk.utils.error.Error;
import com.openmediation.sdk.utils.model.Scene;
...
InterstitialAd.setAdListener(new InterstitialAdListener() {

    /**
     * Invoked when the interstitial ad availability status is changed.
     *
     * @param - available is a boolean.
     *          True: means the interstitial ad is available and you can
     *              show the video by calling InterstitialAd.showAd().
     *          False: means no ad are available
     */
    @Override
    public void onInterstitialAdAvailabilityChanged(boolean available) {
        // Change the interstitial ad state in app according to param available.
    }

    /**
     * Invoked when the Interstitial ad view has opened.
     * Your activity will lose focus.
     */
    @Override
    public void onInterstitialAdShowed(Scene scene) {
        // Do not perform heavy tasks till the ad is going to be closed.
    }

    /**
     * Invoked when the Interstitial ad is closed.
     * Your activity will regain focus.
     */
    @Override
    public void onInterstitialAdClosed(Scene scene) {
    }

    /**
     * Invoked when the user clicked on the Interstitial ad.
     */
    @Override
    public void onInterstitialAdClicked(Scene scene) {
    }

    /* Invoked when the Interstitial ad has showed failed
     * @param - error contains the reason for the failure:
     */
    @Override
    public void onInterstitialAdShowFailed(Scene scene, Error error) {
        // Interstitial ad show failed
    }
});
```
>**Note:**
The error parameter in onInterstitialAdShowFailed callback describes the cause of failure. Check the Error Code for more information and guidance on errors.

**Step 2. Serve Interstitial Ad**

Ad Availability

OpenMediation SDK automatic loads ads for you to cache the Interstitial Ads during application lifecycle if only SDK is integrated and initiated successfully. By correctly implementing the InterstitialAdListener , you will be notified about the ad availability through the onInterstitialAdAvailabilityChanged callback.
```
public void onInterstitialAdAvailabilityChanged(boolean available)
```
Another way to check if the ad is avalible is by calling the isReady function directly.

```
public boolean isReady()
```
Show Interstitial Ad

Once you receive the onInterstitialAdAvailabilityChanged callback with available=TRUE, you can invoke the showAd() method to serve a Interstitial Ad to your users.

```
public void showAd(String sceneName)
```

>**Note:**
Scene is optional，if you don't want to use it just ignore the sceneName parameter or use value "" as below.

```
//Use empty String as value if you don't need this functionality.
InterstitialAd.showAd("")
//Or ignore it.
InterstitialAd.showAd()
```

We strongly recommend checking the ad's availability by isReady method before you show Interstitial Ad to your users, as shown in the following code:

```
//if you would like to show ad when it's required
if (InterstitialAd.isReady()) {
    InterstitialAd.showAd(sceneName);
}
```
When you have successfully completed step 2, you will have shown a Interstitial Ad to your users. If you want to serve another Interstitial Ad, you just repeat step 2 to check ad availability and show a new Interstitial Ad.

#### 13. Banner
Banners are rectangular, system-initiated ads that served in a designated area around your live app content.

**Step 1. Init Banner Ad**

The OpenMediation SDK fires several events to inform you of Banner Ad activity. To display Banner Ads, one needs to create a brand new BannerAd object, setup its listeners and load the ads.

The following snippet demonstrates how to use the BannerAd class to create BannerAd objects and implement the BannerAdListener interface to receive Banner Ad events. The SDK will notify the Listener of all possible events listed below:

```
import com.openmediation.sdk.banner.AdSize;
import com.openmediation.sdk.banner.BannerAd;
import com.openmediation.sdk.banner.BannerAdListener;
...
BannerAd bannerAd = new BannerAd(placementId, new BannerAdListener() {

      /**
       * Invoked when Banner Ad are available.
       */
      @Override
      public void onBannerAdLoaded(String placementId, View view) {
          // bannerAd is load success
      }

      /**
       * Invoked when the end user clicked on the Banner Ad
       */
      @Override
      public void onBannerAdClicked(String placementId) {
         // bannerAd clicked
      }

      /**
       * Invoked when the call to load a Banner Ad has failed
       * String error contains the reason for the failure.
      */
      @Override
      public void onBannerAdLoadFailed(String placementId, Error error) {
         // bannerAd fail
      }
});
// default size
bannerAd.setAdSize(AdSize.BANNER);
```

See table below for details about our supported standard banner sizes:

| AdSize |	Description |	Dimensions in dp (WxH) |
|  ----  | ----  | ----  |
|AdSize.BANNER|	Standard Banner	|320 x 50|
|AdSize.MEDIUM_RECTANGLE|	Medium Rectangular Banner|	300 x 250|
|AdSize.LEADERBOARD|	LeaderBoard Banner|	728 x 90
|AdSize.SMART	|Smart Banner(Adjusted for both mobile and tablet)	| If (screen height ≤ 720) 320 x 50; If (screen height > 720) |728 x 90

Step 2. Load BannerAd

Invoke loadAd method to request and cache the Banner Ad before it is available for display to users. It is strongly recommended to invoke this method a short while before the ad is to be shown.

```
bannerAd.loadAd();
```

>**Notes: **
The loadAd method could be called multiple times at any time, but we do not recommend continuous requests for a short period of time. Many requests in a short period of time have no added value because the opportunities of available for inventory is unlikely at this time.
Warning!: Attempting to load a new ad from the onAdFailed() methods is strongly discouraged. If you must load an ad from onAdFailed() ensure to limit ad load retries to avoid continuous failed ad requests in situations such as limited network connectivity.
Please do not request Banner ads regularly in the application, OpenMediation SDK will automatically refresh Banner ads regularly.

**Step 3. Show BannerAd**

After the Banner Ad is successfully loaded by calling the loadAd in Step 2, you will be notified when the ad is ready to be shown through the onAdReady callback which will inform you the availability of ad inventory.

Once you receive the onAdReady callback, you  can use code below to show and serve the Banner Ad to your users.
```
 private RelativeLayout adParent;
 ...
 adParent = this.findViewById(R.id.banner_ad_container);
 ...

 @Override
 public void onBannerAdLoaded(String placementId, View view) {
    // bannerAd is loaded successfully
    if (null != view.getParent()) {
        ((ViewGroup) view.getParent()).removeView(view);
    }
    adParent.removeAllViews();
    RelativeLayout.LayoutParams layoutParams = new RelativeLayout.LayoutParams(
    RelativeLayout.LayoutParams.WRAP_CONTENT, RelativeLayout.LayoutParams.WRAP_CONTENT);
    layoutParams.addRule(RelativeLayout.CENTER_IN_PARENT);
    adParent.addView(view, layoutParams);
}
```

■ R.id.banner_ad_container code as following:

```
<RelativeLayout
   android:id="@+id/banner_ad_container"
   android:layout_width="match_parent"
   android:layout_height="wrap_content">

</RelativeLayout>
```

**Step 4. Destroy BannerAd Object**

It is recommended to invoke destroy method to release BannerAd object when the Activity be destroyed.

```
@Override
public void onDestroy() {
   if (bannerAd != null) {
      bannerAd.destroy();
   }
   super.onDestroy();
}
```

#### 14. Test Your Integration
Test Placement Ids

The quickest way to enable testing is to use OpenMediation test placement ids. These test placement ids are not associated with your OpenMediation account, so there's no risk of your account generating invalid traffic when using these test placement ids.

AppKey：gwsQFNL6ef0WZoIWP1rtHg0gokviPjOX
Banner	：9039
Interstitial	：9038
RewardedVideo	：9037
Splash	：9107

#### 15. Error Code
If any exception or no_ads_fill was happened during the SDK Initing and loading or showing ads, SDK would return error code and cause with InitCallback and Listener callback of ad units, such as in onRewardedVideoAdShowFailed event of RewardedVideoListener you could get the Rewarded Video ad's error information with parameter ErrorResult.

**Error Codes**

The Error Code and Cause as below:

Init

111	invalid request:	SDK init failed for invalid argument error such as wrong AppKey or SDK Configuration error
121	network error:	No network connection during init
131	server error:	Server response code is not 200
151	unknown internal error:	Such as Activity is not available. Make sure the Activity will not be destroyed.

Load

211	Invalid request:	Load failed for invalid argument error such as wrong placementID or SDK Configuration error
221	network error:		No network connection during loading
231	server error:	Server response failure during loading, such as invalid response, timeout or the server experience failure, etc.
241	No ad fill:	Ad request success but no ad was returned
242	SDK not initialized:	SDK Initialize SDK before loading ads
243	reached request cap:	Reached ad request cap for ad loading
244	missing AdNetwork SDK/Adapter:	The instance's AdNetwork SDK or adapter is not integrated correctly
245	adapter returned an error or no fill:	The instance returns an error or no fill
251	unknown internal error:	Such as Activity is not available. Make sure the Activity will not be destroyed.

Show

311	invalid request:		Show ad failed for invalid argument error such as scene doesn't exist
341	ad not ready:		Check with isReady() before showing ads
342	SDK not initialized	SDK:	Initialize SDK before showing any ads
343	reached scene impression cap:		Reached scene cap for ad showing
345	failure:		Internal error during showing
351	unknown internal error:		Such as Activity is not available. Make sure the Activity will not be destroyed.

**Diagnosis**

1. **Error Code 111 - SDK init: invalid request**
**Diagnosis**：the request for SDK init is invalid, usually due to wrong parameters or SDK config error such as incorrect permissions inAndroidManifest.xml. Run following checks to find out which might have caused your problem:
Invalid AppKey: make sure the AppKey in your code is exactly the same as the one obtained from OpenMediation
Invalid activity: pass a legit activity object as a parameter to init(). As the best practice, initialize the SDK in the activity's onCreate event. (Please follow the instructions we provide inside Android SDK Integration on this page, under "7. Initialization".)
WebView not supported: your current device does not support WebView on which the OpenMediation SDK depends. Find a proper device before retry.
Missing required permissions in AndroidManifest.xml. Both INTERNET and ACCESS_NETWORK_STATE are required for the SDK to function properly. Make these two permissions are granted in AndroidManifest.xml. (Please follow the instructions we provide inside Android SDK Integration on this page, under "4. Update AndroidManifest.xml".)


2. **Error Code 121 - SDK init: network error**
**Diagnosis**： your current device has no network connection to the Internet.
**Remedy**：turn on your device's WiFi switch, or cellular network connection together with a valid SIM card.


3. **Error Code 151 - SDK init: unknown error**
**Diagnosis**：some unknow error, such as an activity was destroyed, or runtime exception, has occurred.
**Remedy**：
First, check if your app destroyed the activity during SDK init. Neither pass a temporary activity as a parameter to the SDK, nor finish/destroy the activity during SDK init. A legit activity is required for the SDK to initialize itself as well as any necessary 3rd party AdNetwork's SDK. Any premature destruction of the activity will cause an unknown exception.
If the problem still persists after step 1, contact our technical support by email or submitting a ticket with your logs, device and app config info.

4. **Error Code 131 - SDK init: server error**
**Diagnosis**：probable cause can be your unstable network connection.
**Remedy**：make sure to have astable connection to the Internet. If the problem still persists, contact our technical support by email or submitting a ticket with your logs, device and app config info.


5. **Error Code 241 - Ads loading: No ad fill**
**Diagnosis**：No available ads for loading. Probable causes are: waterfall weights and mediation rules setup, instance availability, device/app config.
**Remedy**：Run following checks to find out which might have caused your problem:
Your app is enabled. If it's been deleted or disabled, contact our technical support by email or submitting a ticket with your logs.
If you used our mediation feature, check your mediation rules setup to make sure there is at least one active instance whose waterfall weight is greater than 0
Service restrictions due to device(no ad tracking)/region(no available ads)/network: switch to a new device, or IP address, or WiFi/cellular connection. Or add your device to the test device list to rule out possible restrictions
Update your SDK to the latest version for some earlier versions may not be able to provide all of the features you need
Should the problem persist, contact our technical support by email or submitting a ticket with your logs, device and app config info.

6. **Error Code 242 - Ad loading: SDK not initialized**
**Diagnosis**：SDK init() was not called in your app. (This is different from the init failed error.)
**Remedy**：add/locate init() in your code and make sure it was called before ads loading. We recommend to initialize the SDK in the activity's onCreate(), and not destroy the activity or use a temporary one. (Please follow the instructions we provide inside Android SDK Integration on this page, under "7. Initialization".)

7. **Error Code 243 - Ad loading: reached impression cap**
**Diagnosis**：impression frequency control caused your loading as request to return an error.
**Remedy**：check the cap config on OpenMediation's developer platform. Increase/lift the cap before trying again.


8. **Error Code 244 - Instance loading failure: missing AdNetwork SDK/Adapter**
**Diagnosis**：the required AdNetwork SDK/Adapter for the instance you are trying to load is missing.
**Remedy**：Add the missing AdNetwork SDK/Adapter. Please follow the instructions we provide inside Add Mediation Networks on this page to add the necessary AdNetwork SDKs & Adapters.


9. **Error Code 245 - Instance loading failure: adapter returned an error**
**Diagnosis**： some mediated AdNetwork failed to return an ad to load.
**Remedy**：Use the AdNetwork's error message to locate the cause. Usually the AdNetwork's help center can provide specific solutions. Some common ones are:
No fill error: switch to a new device, or IP address, or WiFi/cellular connection. Or add your device to the test device list to rule out possible restrictions. Sometimes one can lower the instance's floor price set at the AdNetwork to increase fill rate in order to avoid this type of error.
Cap reached error: Increase/lift the cap at the AdNetwork.
Internal error such as timeout or unknown error: switch to another AdNetwork or retry after the AdNetwork recovers.
Other error such as wrong parameters or invalid request: contact our technical support by email or submitting a ticket with your logs, device and app config info.


10. Ads showing related error code.
Error Code 341 - Ad not ready: no ads in stock. Wait on onAvailable(), or check with isReady() before showing ads.
Error Code 343 - Reached scene impression cap: increase/lift the cap.
Error Code 311 - Invalid request: the scene corresponding to the PlacementId parameter doesn't exist.
Error Code 351 - Unknown error: see the load and init section above.
Error Code 345 - Adapter returned an error: see the AdNetwork's help center.
