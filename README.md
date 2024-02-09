## Digio Android eSign/eMandate SDK
### **How to Integrate?**
1. Add it in your root build.gradle at the end of repositories:

```
allprojects {
    repositories {
        ...
        maven { url 'https://jitpack.io' }
    }
}

plugins {
   id 'com.android.application' version '7.4.2' apply false
   id 'com.android.library' version '7.4.2' apply false
   id 'org.jetbrains.kotlin.android' version '1.7.20' apply false
   id 'androidx.navigation.safeargs' version '2.4.2' apply false
}

```

2. Add the dependency:

```
plugins {
   id 'com.android.application'
}

android {
   ...
   buildFeatures {
       viewBinding true
       dataBinding true
   }
}

dependencies {
    implementation 'com.github.digio-tech:gateway:v4.0.8'
    implementation 'com.github.digio-tech:gateway_esign:v4.0.10'
    implementation 'com.github.digio-tech:protean-esign:v3.2'
    implementation 'com.github.digio-tech:cvl_esign:v1.0.0'
    implementation 'com.github.digio-tech:cvl_rdservice:v1.0.0'


    
    // Other dependencies
    implementation 'androidx.appcompat:appcompat:1.6.1'
    implementation 'com.google.android.material:material:1.9.0'
    implementation 'androidx.navigation:navigation-fragment-ktx:2.5.3'
    implementation 'androidx.navigation:navigation-ui-ktx:2.5.3'
    // Added in version 4.0.6
    implementation 'androidx.swiperefreshlayout:swiperefreshlayout:1.1.0'
    Following dependency is also not required if org.jetbrains.kotlin.android plugin version is 1.8.*
    implementation 'androidx.core:core-ktx:1.10.0'
    implementation 'com.android.volley:volley:1.2.1'
    implementation 'com.scottyab:rootbeer-lib:0.1.0'
}
```

3. After updating dependencies click File =>  Sync Project with Gradle Files


**Digio SDK supports android version 5.0 and above (SDK level 21 above)**

### **Biometric Configuration**
1. Finger print/ IRIS / FACE authentication required following software
   1. RD services software must be installed in device (Provided by vendor)
   2. For face based authentication on SANDBOX environment use the following apk from the given link. https://digio-public-docs.s3.ap-south-1.amazonaws.com/esign/rd_service/facerd_v0.7.43_pre_prod.apk
      On PRODUCTION - app will be redirected to play store to install necessary RD Service. 

    

**Note - For hybrid applications, you need to create a channel/bridge/plugin to communicate with the native SDK.**




### **Steps to Invoke signing**
1. Configure Digio instances : should be called on activity/fragment onCreate
```
Digio digio = new Digio();

@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    DigioConfig digioConfig = new DigioConfig();
    DigioTheme theme = new DigioTheme();
    theme.setPrimaryColor(android.R.color.holo_red_dark);
    theme.setFontFamily("Unbounded");
    theme.setSecondaryColorHex("#141414");
    theme.setFontUrl("https://fonts.googleapis.com/css2?family=Unbounded:wght@200&display=swap");
    digioConfig.setTheme(theme);
    digioConfig.setLogo("https://www.digio.in/images/digio_blue.png"); // Your company logo url
    digioConfig.setEnvironment(DigioEnvironment.SANDBOX); // SANDBOX or PRODUCTION
    digioConfig.setServiceMode(DigioServiceMode.OTP);  // FP/OTP/IRIS/FACE
    try {
        digio.init(this, digioConfig);
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```


2. DigioResponseListener and import onDigioSuccess, onDigioFailure override function in your activity/fragment. Below are function signatures

```
@Override
public void onDigioSuccess(@NonNull DigioResponse digioResponse) {
    System.out.println("digioResponse = " + digioResponse);
}

@Override
public void onDigioFailure(@NonNull DigioResponse digioResponse) {
    System.out.println("digioResponse = " + digioResponse);
}

@Override
public void onGatewayEvent(@NonNull GatewayEvent gatewayEvent) {
    System.out.println("gatewayEvent = " + gatewayEvent);
}
```

3. Starting the sign flow
```
try {
    digio.start(signForm.getDocumentId(), signForm.getEmail()); //this refers DigioResponseListener
} catch (Exception e) {
    e.printStackTrace();
}
```


4. **Proguard :** 
#### It is required to test the release build for possible proguard exceptions before prod releases.
```
-keepattributes SourceFile,LineNumberTable
-keep public class * extends java.lang.Exception

-keepclassmembers class * {
    @android.webkit.JavascriptInterface <methods>;
}
-keepattributes JavascriptInterface
-keepattributes *Annotation*
-keepattributes Signature
-optimizations !method/inlining/*
-keeppackagenames

-keepnames class androidx.navigation.fragment.NavHostFragment
-keep class * extends androidx.fragment.app.Fragment{}
-keepnames class * extends android.os.Parcelable
-keepnames class * extends java.io.Serializable

-keepclassmembers enum * {
    public static **[] values();
    public static ** valueOf(java.lang.String);
}

-dontwarn androidx.databinding.**
-keep class androidx.databinding.** { *; }
-keepclassmembers class * extends androidx.databinding.** { *; }

-dontwarn org.json.**
-keep class org.json** { *; }

-keep public class org.simpleframework.**{ *; }
-keep class org.simpleframework.xml.**{ *; }
-keep class org.simpleframework.xml.core.**{ *; }
-keep class org.simpleframework.xml.util.**{ *; }
-dontwarn com.google.android.gms.**
-keep class com.google.android.gms.** { *; }
-keep class com.google.android.material.** { *; }

-dontwarn org.simpleframework.**

-keepattributes ElementList, Root
-keepclassmembers class * {
    @org.simpleframework.xml.* *;
}

-keep class org.spongycastle.** { *; }
-keep class com.ecs.rdlibrary.request.** { *; }
-keep class com.ecs.rdlibrary.response.** { *; }
-keep class com.ecs.rdlibrary.utils.** { *; }
-keep class com.ecs.rdlibrary.ECSBioCaptureActivity { *; }
-keep class org.simpleframework.xml.** { *; }
-keepattributes Exceptions, InnerClasses
```

#### **App crash after starting digio flow :**
- Make sure init is called before start and all the parameter values are proper as the documentation.
- There is no missing dependency as described above.


[Check out our demo App implementation](https://drive.google.com/file/d/1BuVZhMbLb5abfAHc4_P5VT6kmRgMJxVJ/view?usp=sharing)

Demo App Apks : [PRODUCTION](https://drive.google.com/file/d/1Gr5toJENTFUGkCQHMxXIg9h79HrSGwnq/view?usp=sharing) , [SANDBOX](https://drive.google.com/file/d/1TO2tqGcaw2b22nM6k3ln-Qy1lprXK9Kc/view?usp=sharing)



|<a name="_g2lkuhshpqkl"></a>**DigioEnvironment**  | <p>SANDBOX</p><p>PRODUCTION</p>           | Mandatory                                                                                                                                                                                                                                                                           |
| :- |:------------------------------------------|:------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|<a name="_dj5ef5818xf7"></a>**DigioServiceMode**| <p>OTP</p><p>FP</p><p>IRIS</p><p>FACE</p> | <p>Note:</p><p>**OTP** (default): Aadhaar OTP authentication based eSign</p><p></p><p>**FP**: Fingerprint authentication based Aadhaar eSign (Biometric)</p><p></p><p>**IRIS**: Iris authentication based Aadhaar eSign</p><p>**FACE**: Face authentication based Aadhaar eSign</p> |
###

### <a name="_w7hact52sg6g"></a><a name="_702inqpogtkm"></a>**Sample Success Response**


|<p></p><p>documentId</p><p></p>|<p></p><p>DID22040413040490937VNTC6LAP8KWD</p>|String format (Request ID Passed by parent app)|
| :- | :- | :- |
|<p></p><p>message</p>|<p>Signing Success</p><p></p>|<p>**Failur** = Digio sdk crash</p><p></p><p>**Webpage could not be loaded** = web page not loaded due to internet connection issue even after 3 retries.</p><p></p>|
|code|1001|<p>DigioConstants.</p><p>RESPONSE\_CODE\_SUCCESS = 1001</p><p>RESPONSE\_CODE\_CANCEL = -1000</p><p>RESPONSE\_CODE\_FAIL = 1002</p><p>RESPONSE\_CODE\_WEB\_VIEW\_CRASH = 1003</p><p>RESPONSE\_CODE\_SDK\_CRASH</p><p>= 1004</p>|
|screen|document\_preview||
|npciTxnId||String|
|esignState|<p>{</p><p>`  `"txn\_id": "0.8113660041195622-state",</p><p>`  `"doc\_id": "DID22040413040490937VNTC6LAP8KWD",</p><p>`  `"last\_state": {</p><p>`    `"screen": "user\_auth\_screen",</p><p>`    `"state\_code": "initiated"</p><p>`  `}</p><p>}</p><p></p>|Object|


### <a name="_wjtrktrxj8c5"></a>**Sample Failure/Cnacel Response**


|<p></p><p>documentId</p><p></p>|<p></p><p>DID22040413040490937VNTC6LAP8KWD</p>|String format (Request ID Passed by parent app)|
| :- | :- | :- |
|<p></p><p></p><p>message</p>|<p></p><p>Signing Failur</p>|<p></p><p></p><p>String</p>|
|code|-1000|<p>RESPONSE\_CODE\_CANCEL = -1000</p><p>RESPONSE\_CODE\_FAIL = 1002</p><p>RESPONSE\_CODE\_WEB\_VIEW\_CRASH = 1003</p><p>RESPONSE\_CODE\_SDK\_CRASH</p><p>` `= 1004</p>|
|errorCode|-1000|<p>RESPONSE\_CODE\_CANCEL = -1000</p><p>RESPONSE\_CODE\_FAIL = 1002</p><p>RESPONSE\_CODE\_WEB\_VIEW\_CRASH = 1003</p><p>RESPONSE\_CODE\_SDK\_CRASH</p><p>` `= 1004</p><p>WEB\_PAGE\_NOT\_LOADED = -1 TO -16</p>|
|esignState|<p>{</p><p>`  `"txn\_id": "0.8113660041195622-state",</p><p>`  `"doc\_id": "DID22040413040490937VNTC6LAP8KWD",</p><p>`  `"last\_state": {</p><p>`    `"screen": "user\_auth\_screen",</p><p>`    `"state\_code": "initiated"</p><p>`  `}</p><p>}</p>|Object|


**Gateway Events :**

Refer Gateway document for all posible events and error data : [Gateway Event Doc](https://docs.google.com/document/d/15LHtjGyXd_JNM0de8uH9zB7WllJikRl1d9e4qdy0-C0)



 DigioEvent<br>`    `documentId: string;<br>`    `txnId: string;<br>`    `entity: string;<br>`    `identifier: string;<br>`    `event: string;<br>`    `payload: <br>`        `type: 'error' | 'info';<br>`        `data?: HashMap<String,Any>;<br>`        `error?: <br>`            `code: string;<br>`            `message: string;



**Webview Error Codes and messages. The following webview errors are already handled by digio SDK, The name,description and message will be displayed during sdk journey if any internet connectivity issue happens.**

|**name**|**error code description from webview client**|**error code**|**messages (manually mapped, error\_code may differ)**|
| :- | :- | :- | :- |
|*ERROR\_UNKNOWN*|*Generic error*|-1|<p>net::ERR\_FAILED</p><p>net::ERR\_CACHE\_MISS</p>|
|*ERROR\_HOST\_LOOKUP*|*Server or proxy hostname lookup failed*|-2|<p>net::ERR\_INTERNET\_DISCONNECTED</p><p>net::ERR\_NAME\_RESOLUTION\_FAILED</p><p>net::ERR\_NAME\_NOT\_RESOLVED</p>|
|*ERROR\_UNSUPPORTED\_AUTH\_SCHEME*|*Unsupported authentication scheme (not basic or digest)*|-3|net::ERR\_ACCESS\_DENIED|
|*ERROR\_AUTHENTICATION*|*User authentication failed on server*|-4||
|*ERROR\_PROXY\_AUTHENTICATION*|*User authentication failed on proxy*|-5||
|*ERROR\_CONNECT*|*Failed to connect to the server*|-6|<p>net::ERR\_CONNECTION\_FAILED</p><p>net::ERR\_ABORTED</p><p>net::ERR\_CONNECTION\_ABORTED</p><p>net::ERR\_CONNECTION\_CLOSED</p>|
|*ERROR\_IO*|*Failed to read or write to the server*|-7|<p>net::ERR\_INVALID\_RESPONSE</p><p>net::ERR\_EMPTY\_RESPONSE</p><p>net::ERR\_CONTENT\_DECODING\_FAILED</p><p>net::ERR\_CONTENT\_LENGTH\_MISMATCH</p>|
|*ERROR\_TIMEOUT*|*Connection timed out*|-8|net::ERR\_CONNECTION\_TIMED\_OUT|
|*ERROR\_REDIRECT\_LOOP*|*Too many redirects*|-9|net::ERR\_TOO\_MANY\_REDIRECTS|
|*ERROR\_UNSUPPORTED\_SCHEME*|*Unsupported URI scheme*|-10|net::ERR\_UNKNOWN\_URL\_SCHEME|
|*ERROR\_FAILED\_SSL\_HANDSHAKE*|*Failed to perform SSL handshake*|-11|<p>net::ERR\_SSL\_PROTOCOL\_ERROR</p><p>net::ERR\_CERT\_DATE\_INVALID</p><p>net::ERR\_CERT\_AUTHORITY\_INVALID</p><p>net::ERR\_BAD\_SSL\_CLIENT\_AUTH\_CERT</p><p>net::ERR\_SSL\_CLIENT\_AUTH\_CERT\_NEEDED</p>|
|*ERROR\_BAD\_URL*|*Malformed URL*|-12|net::ERR\_INVALID\_URL|
|*ERROR\_FILE*|*Generic file error*|-13|net::ERR\_FILE\_TOO\_BIG|
|*ERROR\_FILE\_NOT\_FOUND*|*File not found*|-14|net::ERR\_FILE\_NOT\_FOUND|
|*ERROR\_TOO\_MANY\_REQUESTS*|*Too many requests during this load*|-15||
|*ERROR\_UNSAFE\_RESOURCE*|*Resource load was canceled by Safe Browsing*|-16|net::ERR\_INSUFFICIENT\_RESOURCES|
||||net::ERR\_INVALID\_HANDLE|
||||net::ERR\_CONTENT\_DECODING\_FAILED|
||||net::ERR\_CONTENT\_LENGTH\_MISMATCH|




### Change Logs
- **Version 4.0.10 :**
  - Using CVL eSign SDK for biometric and face based authentication


- **Version 4.0.8 :**
  - Added FACE authentication to eSign
  - Using protean eSign SDK for biometric and face based authentication
  - Removed firebase crashlytics 

- **Version 4.0.6 :**
  - Introduced webview connection error handling with in digio sdk.
  - Added internet connection observability, if internet connection get disconnected message will be displayed with in digio sdk.
  - Digio activity will run in portrait mode only and will not re-create on any system configuration changes
  - Digio Activity will not run in separate process, there is no requirement of any handling in Application class.
  - Introduced Gateway events


### Migration Guide
- **4.0.6 => 4.0.8**
  - Remove firebase crashlytics dependencies and gradle plugin if not required by app.
  - Add following dependencies ```implementation 'com.github.digio-tech:protean-esign:v3.2'```
  - Add following to pro-guard rules
```
-dontwarn org.json.**
-keep class org.json** { *; }
```
  
- **4.0.3 => 4.0.6**
  - implemented onGatewayEvent on DigioResponseListener
  - add following dependencies
  - add  ```implementation 'androidx.swiperefreshlayout:swiperefreshlayout:1.1.0'``` to build.gradle




**Note:** Digio reserves the right to modify this API document from time-to-time. If you are a business using this API, you will be notified  well in advance, prior to any change is made

© Copyright 2016-23 | [www.digio.in](http://www.digio.in) | For Limited Circulation

