# AppLovin MAX Unity 플러그인 연동 가이드

## 시작하기

### ID 발급

매니저로부터 다음을 발급 받으세요.

- _AppLovin SDK key_
- _AppLovin MAX ad unit ID_ (앱의 광고 형식마다 고유한 값)
- _AdMob app ID_ (앱마다 고유한 값)
- _PointBerry inventory ID_ (앱의 광고 단위마다 고유한 값)

### AppLovin MAX Unity 플러그인 가져오기

[Requirements](https://dash.applovin.com/documentation/mediation/unity/getting-started/integration#requirements)를 확인하세요.

AppLovin MAX Unity 플러그인 5.3.0을 [다운로드](https://artifacts.applovin.com/unity/com/applovin/applovin-sdk/AppLovin-MAX-Unity-Plugin-5.3.0-Android-11.3.1-iOS-11.3.1.unitypackage)하세요. **Assets > Import Package > Custom Package…** 에서 다운로드한 플러그인을 import 하세요.

### AppLovin MAX 어댑터 가져오기

**AppLovin > Integration Manager** 에서 **Google AdMob** 어댑터와 **Unity Ads** 어댑터를 설치 후 import 하세요. 이때 매니저로부터 발급 받은 *AdMob app ID*를 입력하세요.

### AppLovin MAX SDK 초기화

앱이 시작된 후 가능한 한 빨리 SDK를 초기화하세요. 이때 매니저로부터 발급 받은 *AppLovin SDK key*를 사용하세요.

SDK가 초기화되면 광고를 로드하세요. `InitializeBannerAds()`는 [배너 광고 로드](#배너-광고-로드)에, `InitializeInterstitialAds()`는 [전면 광고 로드](#전면-광고-로드)에, `InitializeRewardedAds()`는 [보상형 광고 로드](#보상형-광고-로드)에 정의되어 있습니다.

```c#
MaxSdkCallbacks.OnSdkInitializedEvent += (MaxSdkBase.SdkConfiguration sdkConfiguration) =>
{
    // SDK가 초기화됐습니다.

    // 광고를 로드하세요.
    InitializeBannerAds();
    InitializeInterstitialAds();
    InitializeRewardedAds();
};
MaxSdk.SetSdkKey("YOUR_APPLOVIN_SDK_KEY");
MaxSdk.InitializeSdk();
```

### PointBerry Event Tracker 가져오기

PointBerry Event Tracker 1.0.2를 [다운로드](https://jitpack.io/com/github/connect-n/pointberry-event-tracker-android/1.0.2/pointberry-event-tracker-android-1.0.2.aar)하세요. Assets/Plugins/Android/ 경로 내 아무 위치에나 다운로드한 AAR 파일을 복사하세요. 그리고 **Inspector > Import Settings** 에서 **Android** 체크 박스를 선택하세요.

`io.pointberry.eventtracker.PointBerryImpressionTracker`의 `logImpression()`을 호출하는 `LogImpression()`을 정의하세요.

```c#
public class PointBerryImpressionTracker : MonoBehaviour
{
    AndroidJavaObject activity;
    AndroidJavaObject context;
    AndroidJavaObject impTracker;

    void Awake()
    {
        using (AndroidJavaClass unityPlayer = new AndroidJavaClass("com.unity3d.player.UnityPlayer"))
        {
            activity = unityPlayer.GetStatic<AndroidJavaObject>("currentActivity");
            context = activity.Call<AndroidJavaObject>("getApplicationContext");
            impTracker = new AndroidJavaObject("io.pointberry.eventtracker.PointBerryImpressionTracker", context)
        }
    }

    public void LogImpression(string inventoryId, bool development)
    {
        activity.Call("runOnUiThread", new AndroidJavaRunnable(() =>
        {
            impTracker.Call("logImpression", inventoryId, development);
        }));
    }
}
```

## 배너 광고

### 배너 광고 로드

`MaxSdk.CreateBanner()`를 호출해서 광고를 로드하세요. 이때 매니저로부터 발급 받은 *MAX ad unit ID*를 사용하세요.

광고 이벤트를 수신하는 콜백을 연결하세요.

`OnBannerAdLoadedEvent()` 콜백에서 `LogImpression()`을 호출해서 광고 노출을 로깅하세요. 이때 매니저로부터 발급 받은 *PointBerry inventory ID*를 사용하세요.

```c#
public void InitializeBannerAds()
{
    // 광고를 로드하세요.
    // 광고를 하단 중앙에 로드하는 예시입니다.
    MaxSdk.CreateBanner("YOUR_MAX_AD_UNIT_ID", MaxSdkBase.BannerPosition.BottomCenter);

    MaxSdk.SetBannerPlacement("YOUR_MAX_AD_UNIT_ID", "YOUR_POINTBERRY_INVENTORY_ID");

    // 광고 배경색을 설정하세요.
    // 광고 배경색을 흰색으로 설정하는 예시입니다.
    MaxSdk.SetBannerBackgroundColor("YOUR_MAX_AD_UNIT_ID", "#-ffffff");

    // 배너 광고 크기는 폰에서 320×50, 태블릿에서 728×90으로 자동 조정됩니다.

    // 콜백을 연결하세요.
    MaxSdkCallbacks.Banner.OnAdLoadedEvent += OnBannerAdLoadedEvent;
    MaxSdkCallbacks.Banner.OnAdLoadFailedEvent += OnBannerAdLoadFailedEvent;
    MaxSdkCallbacks.Banner.OnAdClickedEvent += OnBannerAdClickedEvent;
    MaxSdkCallbacks.Banner.OnAdExpandedEvent += OnBannerAdExpandedEvent;
    MaxSdkCallbacks.Banner.OnAdCollapsedEvent += OnBannerAdCollapsedEvent;
}

private void OnBannerAdLoadedEvent(string adUnitId, MaxSdkBase.AdInfo adInfo)
{
    // 광고가 로드됐습니다.

    // 광고 노출을 로깅하세요.
    LogImpression("YOUR_POINTBERRY_INVENTORY_ID");

    // 개발 환경에서는 development 파라미터에 true를 전달해서 로그 메시지를 출력하세요.
    // LogImpression("YOUR_POINTBERRY_INVENTORY_ID", true);
}

private void OnBannerAdLoadFailedEvent(string adUnitId, MaxSdkBase.ErrorInfo errorInfo)
{
    // 광고가 로드에 실패했습니다.
}

private void OnBannerAdClickedEvent(string adUnitId, MaxSdkBase.AdInfo adInfo)
{
    // 광고가 클릭됐습니다.
}

private void OnBannerAdExpandedEvent(string adUnitId, MaxSdkBase.AdInfo adInfo) {}

private void OnBannerAdCollapsedEvent(string adUnitId, MaxSdkBase.AdInfo adInfo) {}
```

### 배너 광고 게재

`MaxSdk.ShowBanner()`를 호출해서 광고를 게재하세요. 이때 매니저로부터 발급 받은 *MAX ad unit ID*를 사용하세요.

```c#
MaxSdk.ShowBanner("YOUR_MAX_AD_UNIT_ID");
```

### 배너 광고 닫기

`MaxSdk.HideBanner()`을 호출해서 광고를 닫으세요. 이때 매니저로부터 발급 받은 *MAX ad unit ID*를 사용하세요.

```c#
MaxSdk.HideBanner("YOUR_MAX_AD_UNIT_ID");
```

## 전면 광고

### 전면 광고 로드

`MaxSdk.LoadInterstitial()`을 호출해서 광고를 로드하세요. 이때 매니저로부터 발급 받은 *MAX ad unit ID*를 사용하세요.

광고 이벤트를 수신하는 콜백을 연결하세요.

`OnInterstitialDisplayedEvent()` 콜백에서 `LogImpression()`을 호출해서 광고 노출을 로깅하세요. 이때 매니저로부터 발급 받은 *PointBerry inventory ID*를 사용하세요.

```c#
int retryAttempt;

public void InitializeInterstitialAds()
{
    // 콜백을 연결하세요.
    MaxSdkCallbacks.Interstitial.OnAdLoadedEvent += OnInterstitialLoadedEvent;
    MaxSdkCallbacks.Interstitial.OnAdLoadFailedEvent += OnInterstitialLoadFailedEvent;
    MaxSdkCallbacks.Interstitial.OnAdDisplayedEvent += OnInterstitialDisplayedEvent;
    MaxSdkCallbacks.Interstitial.OnAdClickedEvent += OnInterstitialClickedEvent;
    MaxSdkCallbacks.Interstitial.OnAdDisplayFailedEvent += OnInterstitialAdFailedToDisplayEvent;
    MaxSdkCallbacks.Interstitial.OnAdHiddenEvent += OnInterstitialHiddenEvent;

    // 첫 광고를 로드하세요.
    LoadInterstitial();
}

private void LoadInterstitial()
{
    MaxSdk.LoadInterstitial("YOUR_MAX_AD_UNIT_ID");
}

// 광고 이벤트

private void OnInterstitialLoadedEvent(string adUnitId, MaxSdkBase.AdInfo adInfo)
{
    // 광고가 로드됐고 이제 게재될 수 있습니다.
    // MaxSdk.IsInterstitialReady()가 true를 리턴합니다.

    // 로드 재시도 횟수를 리셋하세요.
    retryAttempt = 0;
}

private void OnInterstitialLoadFailedEvent(string adUnitId, MaxSdkBase.ErrorInfo errorInfo)
{
    // 광고가 로드에 실패했습니다.

    // 딜레이를 기하급수적으로 증가시키며 광고 로드를 재시도하세요.
    // 딜레이를 1초부터 64초까지 2배씩 증가시키는 예시입니다.
    retryAttempt++;
    double retryDelay = Math.Pow(2, Math.Min(6, retryAttempt));
    Invoke("LoadInterstitial", (float) retryDelay);
}

private void OnInterstitialDisplayedEvent(string adUnitId, MaxSdkBase.AdInfo adInfo)
{
    // 광고가 게재됐습니다.

    // 광고 노출을 로깅하세요.
    LogImpression("YOUR_POINTBERRY_INVENTORY_ID");

    // 개발 환경에서는 development 파라미터에 true를 전달해서 로그 메시지를 출력하세요.
    // LogImpression("YOUR_POINTBERRY_INVENTORY_ID", true);
}

private void OnInterstitialAdFailedToDisplayEvent(string adUnitId, MaxSdkBase.ErrorInfo errorInfo, MaxSdkBase.AdInfo adInfo)
{
    // 광고가 게재에 실패했습니다.

    // 다음 광고를 로드하세요.
    LoadInterstitial();
}

private void OnInterstitialClickedEvent(string adUnitId, MaxSdkBase.AdInfo adInfo)
{
    // 광고가 클릭됐습니다.
}

private void OnInterstitialHiddenEvent(string adUnitId, MaxSdkBase.AdInfo adInfo)
{
    // 광고가 닫혔습니다.

    // 다음 광고를 미리 로드하세요.
    LoadInterstitial();
}
```

### 전면 광고 게재

`MaxSdk.ShowInterstitial()`을 호출해서 광고를 게재하세요. 이때 매니저로부터 발급 받은 *MAX ad unit ID*와 *PointBerry inventory ID*를 사용하세요.

```c#
if (MaxSdk.IsInterstitialReady("YOUR_MAX_AD_UNIT_ID"))
{
    MaxSdk.ShowInterstitial("YOUR_MAX_AD_UNIT_ID", "YOUR_POINTBERRY_INVENTORY_ID");
}
```

## 보상형 광고

### 보상형 광고 로드

`MaxSdk.LoadRewardedAd()`를 호출해서 광고를 로드하세요. 이때 매니저로부터 발급 받은 *MAX ad unit ID*를 사용하세요.

광고 이벤트를 수신하는 콜백을 연결하세요.

`OnRewardedAdReceivedRewardEvent()` 콜백에서 `LogImpression()` 호출해서 광고 노출을 로깅하세요. 이때 매니저로부터 발급 받은 *PointBerry inventory ID*를 사용하세요.

```c#
int retryAttempt;

public void InitializeRewardedAds()
{
    // 콜백을 연결하세요.
    MaxSdkCallbacks.Rewarded.OnAdLoadedEvent += OnRewardedAdLoadedEvent;
    MaxSdkCallbacks.Rewarded.OnAdLoadFailedEvent += OnRewardedAdLoadFailedEvent;
    MaxSdkCallbacks.Rewarded.OnAdDisplayedEvent += OnRewardedAdDisplayedEvent;
    MaxSdkCallbacks.Rewarded.OnAdDisplayFailedEvent += OnRewardedAdFailedToDisplayEvent;
    MaxSdkCallbacks.Rewarded.OnAdClickedEvent += OnRewardedAdClickedEvent;
    MaxSdkCallbacks.Rewarded.OnAdReceivedRewardEvent += OnRewardedAdReceivedRewardEvent;
    MaxSdkCallbacks.Rewarded.OnAdHiddenEvent += OnRewardedAdHiddenEvent;

    // 첫 광고를 로드하세요.
    LoadRewardedAd();
}

private void LoadRewardedAd()
{
    MaxSdk.LoadRewardedAd("YOUR_MAX_AD_UNIT_ID");
}

// 광고 이벤트

private void OnRewardedAdLoadedEvent(string adUnitId, MaxSdkBase.AdInfo adInfo)
{
    // 광고가 로드됐고 이제 게재될 수 있습니다.
    // MaxSdk.IsRewardedAdReady()가 true를 리턴합니다.

    // 로드 재시도 횟수를 리셋하세요.
    retryAttempt = 0;
}

private void OnRewardedAdLoadFailedEvent(string adUnitId, MaxSdkBase.ErrorInfo errorInfo)
{
    // 광고가 로드에 실패했습니다.

    // 딜레이를 기하급수적으로 증가시키며 광고 로드를 재시도하세요.
    // 딜레이를 1초부터 64초까지 2배씩 증가시키는 예시입니다.
    retryAttempt++;
    double retryDelay = Math.Pow(2, Math.Min(6, retryAttempt));
    Invoke("LoadRewardedAd", (float) retryDelay);
}

private void OnRewardedAdDisplayedEvent(string adUnitId, MaxSdkBase.AdInfo adInfo)
{
    // 광고가 게재됐습니다.
}

private void OnRewardedAdFailedToDisplayEvent(string adUnitId, MaxSdkBase.ErrorInfo errorInfo, MaxSdkBase.AdInfo adInfo)
{
    // 광고가 게재에 실패했습니다.

    // 다음 광고를 로드하세요.
    LoadRewardedAd();
}

private void OnRewardedAdClickedEvent(string adUnitId, MaxSdkBase.AdInfo adInfo)
{
    // 광고가 클릭됐습니다.
}

private void OnRewardedAdReceivedRewardEvent(string adUnitId, MaxSdk.Reward reward, MaxSdkBase.AdInfo adInfo)
{
    // 유저가 보상을 받아야 합니다.

    // 광고 노출을 로깅하세요.
    LogImpression("YOUR_POINTBERRY_INVENTORY_ID");

    // 개발 환경에서는 development 파라미터에 true를 전달해서 로그 메시지를 출력하세요.
    // LogImpression("YOUR_POINTBERRY_INVENTORY_ID", true);
}

private void OnRewardedAdHiddenEvent(string adUnitId, MaxSdkBase.AdInfo adInfo)
{
    // 광고가 닫혔습니다.

    // 다음 광고를 미리 로드하세요.
    LoadRewardedAd();
}
```

### 보상형 광고 게재

`MaxSdk.ShowRewardedAd()`를 호출해서 광고를 게재하세요. 이때 매니저로부터 발급 받은 *MAX ad unit ID*와 *PointBerry inventory ID*를 사용하세요.

```c#
if (MaxSdk.IsRewardedAdReady("YOUR_MAX_AD_UNIT_ID"))
{
    MaxSdk.ShowRewardedAd("YOUR_MAX_AD_UNIT_ID", "YOUR_POINTBERRY_INVENTORY_ID");
}
```

## 체크리스트

- [x] 광고가 노출되나요?

  광고가 노출되지 않는다면 MAX SDK를 점검하세요.

- [x] 광고 노출이 로깅되나요?

  `LogImpression("YOUR_POINTBERRY_INVENTORY_ID", true)`을 호출하면 다음과 같은 로그 메시지가 출력되어야 합니다.

  _Successfully logged ad impression for inventory ID P_148 and advertising ID a1312407-a27a-4a2e-a52f-0a0ab89febe1_

  성공을 뜻하는 로그 메시지가 출력되지 않는다면, 즉 광고 노출이 로깅되지 않는다면 PointBerry Event Tracker를 점검하세요.

- [x] 앱 출시 전에, 광고 노출 로깅에 관한 로그 메시지가 출력되지 않도록 했나요?

  출시 버전에서는 `LogImpression("YOUR_POINTBERRY_INVENTORY_ID")` 또는 `LogImpression("YOUR_POINTBERRY_INVENTORY_ID", false)`을 호출하세요.

## 데모 앱

AppLovin에서 제공하는 [데모 앱](https://github.com/AppLovin/AppLovin-MAX-Unity-Plugin)을 참고하세요.
