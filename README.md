# Admitad SDK for Android
Android integration SDK of https://www.admitad.com/

Admitad help center: https://help.admitad.com/en/advertiser/topic/195-mobile-sdk/

## Table of contents

* [Example app](#example-app)
* [Basic integration](#basic-integration)
    * [Add the SDK to your project](#sdk-add)
    * [Usage](#sdk-usage)
      * [Initialized](#initialized)
      * [Registration](#registration)
      * [Returned user](#returned_user)
      * [Loyalty](#loyalty)
      * [Order](#order)
        * [Paid order](#paid_order)
        * [Confirmed purchase](#confirmed_purchase)
      * [Specific event](#specific_event)
      * [Subscribe for all events](#subscribe_for_all_events)
      * [More examples](#more_examples)
      * [Log enabled](#log_enabled)
    * [Error code](#error_code)
* [License](#license)

## <a id="example-app"></a>Example app

You can find examples of use [in this directory](app/src/main/java/ru/tachos/admitadstatistic/MainActivity.java).
You can open the Android project to see an example on how the admitad SDK can be integrated.

## <a id="basic-integration"></a>Basic integration

These are the minimal steps required to integrate the admitad SDK into your Android project. We are going to assume that you use Android Studio for your Android development and target an Android API level 14 (Ice Cream Sandwich) or later.

### <a id="sdk-add"></a>Add the SDK to your project

Add repository to the root gradle:

```gradle
allprojects {
    repositories {
        jcenter()
        google()
    }
}
```

old version:

```gradle
allprojects {
    repositories {
        jcenter()
        maven { url "https://maven.google.com" }
    }
}
```

And this to the project's gradle:

```gradle
 implementation 'ru.tachos.admitadstatisticsdk:admitadstatisticsdk:1.4.8'
```

old version:

```gradle
compile('ru.tachos.admitadstatisticsdk:admitadstatisticsdk:1.4.8') {
        transitive = true
}
```

### <a id="sdk-usage"></a>Usage
#### <a id="initialized">Initialized
  * SDK is being initialized async, so you must call AdmitadTracker#initialize before using. We'd like to reccomend to initialize in the `Application#OnCreate` or in the launcher Activity in `Activity#onCreate`. You have to pass context, postback key (non-null key is mandatory, exception is thrown otherwise), callback (optional)

  ```java
   AdmitadTracker.initialize(getApplicationContext(), YOUR_ANDROID_POSTBACK_KEY, new TrackerInitializationCallback() {
            @Override
            public void onInitializationSuccess() {
            }

            @Override
            public void onInitializationFailed(Exception exception) {
            }
        });
  ```


  * Example of handling deeplink:

  ```java
  @Override
  protected void onCreate(Bundle savedInstanceState) {
      super.onCreate(savedInstanceState);
      onNewIntent(getIntent());
  }

  @Override
  protected void onNewIntent(Intent intent) {
      super.onNewIntent(intent);
      setIntent(intent);

      if (intent.getData() != null) {
          AdmitadTracker.getInstance().handleDeeplink(intent.getData());
      }
  }
  ```


  * When `AdmitadTracker#initialize` is called, it's possible to start tracking even if sdk is not initialized, if sdk has any uid value, logs will be stored and send ASAP. There're several events sdk is able to log:
#### <a id="registration">Registration

      AdmitadTracker.getInstance().logRegistration(*USER_ID*);

#### <a id="returned_user">Returned user

      AdmitadTracker.getInstance().logUserReturn(*USER_ID*, *DAY_COUNT*);

#### <a id="loyalty">Loyalty

      AdmitadTracker.getInstance().logUserLoyalty(*USER_ID*, *OPEN_APP_COUNT*);



#### <a id="order">Order
  * To log confirmed purchase or paid order you have to create AdmitadOrder object using builder. e.g.:

  ```java
    final AdmitadOrder order = new AdmitadOrder.Builder("123", "100.00")
                .setCurrencyCode("RUB")
                .putItem(new AdmitadOrder.Item("Item1", "ItemName1", 300))
                .putItem(new AdmitadOrder.Item("Item2", "ItemName2", 500))
                .setUserInfo(new AdmitadOrder.UserInfo().putExtra("Surname", "UserSurname").putExtra("Age", "18"))
                .build();
  ```
  
  You can initialize AdmitadOrder with extra parameter *tarifCode*. Then Admitad can apply this tariff to the order as defined in your agreement. To get tariff codes ask your Admitad account manager.
  
  ```java
    final AdmitadOrder order = new AdmitadOrder.Builder("123-ISBD-123", "500.00")
              .setCurrencyCode("USD")
              .putItem(new AdmitadOrder.Item("Item2", "ItemName2", 500))
              .setTarifCode("book_promo_action")
              .build();
  ```

  * Then you can log using `order`:
##### <a id="paid_order">Paid order

      AdmitadTracker.getInstance().logOrder(order);

##### <a id="confirmed_purchase">Confirmed purchase

      AdmitadTracker.getInstance().logPurchase(order);

#### <a id="specific_event">Specific event
   * To subscribe for specific event, you can pass callbacks to any log* method, e.g.:

  ```java
     AdmitadTracker.getInstance().logRegistration(*USER_ID*, new TrackerListener() {

            @Override
            public void onSuccess(AdmitadEvent result) {
                 Log.d("AdmitadTracker", "Registration event sent successfully + " + result.toString());
            }

            @Override
            public void onFailure(@AdmitadTrackerCode int errorCode, @Nullable String errorText) {
                 Log.d("AdmitadTracker", "Failed to send registration event: errorCode = " + errorCode + ", errorText = " + errorText);
            }
        });
   ```
#### <a id="subscribe_for_all_events">Subscribe for all events
  * To subscribe for all events, you can call method `AdmitadTracker#addListener`. This method will be always called on sending.

  ```java
   AdmitadTracker.getInstance().addListener(new TrackerListener() {

            @Override
            public void onSuccess(AdmitadEvent result) {
                Log.d("AdmitadTracker", "Event sent successfully + " + result.toString());
            }

            @Override
            public void onFailure(@AdmitadTrackerCode int errorCode, @Nullable String errorText) {
                Log.d("AdmitadTracker", "Failed to send event: errorCode = " + errorCode + ", errorText = " + errorText);
            }
        });
  ```
#### <a id="more_examples">More examples

  * See more examples in the [test project](app/)

#### <a id="log_enabled">Log enabled

  * To enable logs you can call any time:
  ``` java
  AdmitadTracker.setLogEnabled(true);
  ```

### <a id="error_code">Error code
  * Error code can be one of the AdmitadTrackedCode:

  ```java
  public @interface AdmitadTrackerCode {
    int NONE = 0;
    int SUCCESS = 200;

    int ERROR_GENERIC = -100;
    int ERROR_NO_INTERNET = -200;
    int ERROR_SERVER_UNAVAILABLE = -1;
    int ERROR_SDK_NOT_INITIALIZED = -1000;
    int ERROR_SDK_POSTBACK_CODE_MISSED = -1100;
    int ERROR_SDK_GAID_MISSED = -1200;
    int ERROR_SDK_ADMITAD_UID_MISSED = -1300;
  }
  ```



## <a id="license"></a>License

The admitad SDK is licensed under the MIT License.

Copyright (c) 2017 Admitad GmbH

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

