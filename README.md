# Android SDK 0.7

## Getting started

### Account Setup

To ensure the authenticity of the interactions between your app and Truecaller, you need to generate a partner key from the truecaller developer portal ( https://developer.truecaller.com/auth/login ) by providing us with your package name and SHA-1 signing-fingerprint.

You package name corresponds to the applicationId in your app level build.gradle file. Use the following command to get the fingerprint:

```java
keytool -list -v -keystore mystore.keystore
```

Once we have received the package name and the SHA-1 signing-fingerprint, we will provide you with a unique "PartnerKey" which you need to include in your project to authorize all verification requests.

### Understanding how the user verification flow works

 - Truecaller app needs to be present on the user's device. User starts by clicking on your Login / Signup CTA and would be shown the standard Truecaller profile view asking for the user's consent. The user authorizes by clicking verify and their Truecaller profile would be shared with you


### Using the SDK with your Android Studio Project

1. Ensure that your Minimum SDK version is atleast API level 16 or above ( Android 4.1 ). In case your android project compiles for API level below 16, you can include the following line in your AndroidManifest.xml file to avoid any compilation issues :
```java
<uses-sdk tools:overrideLibrary="com.truecaller.android.sdk"/>
```
Using this would ensure that the sdk works normally for API level 16 & above, and would be disabled for API level < 16
Please make sure that you put the necessary API level checks before accessing the SDK methods in case compiling for API level < 16

2. Add the provided truesdk-0.7-releasePartner.aar file into your libs folder. Example path: /app/libs/ 
3. Open the build.gradle of your application module and ensure that your lib folder can be used as a repository :

    ```java
    repositories {
        flatDir {
            dirs 'libs'
        }
    }
    ```
    
    Secondly add the compile dependency with the latest version of the TrueSDK aar :

    ```java
    dependencies {
        implementation(name: "truesdk-0.7-releasePartner", ext: "aar")
    }
    ```
    Add the following dependencies within your gradle file :
    ```java
    implementation 'com.squareup.retrofit2:retrofit:2.3.0'
    implementation 'com.squareup.okhttp3:okhttp:3.7.0'
    ```

4. Open your strings.xml file. Example path: /app/src/main/res/values/strings.xml and add a new string with the name "partnerKey" and value as your "PartnerKey"

5. Open your AndroidManifest.xml and add a meta-data element to the application element:
 
    ```java
    <application android:label="@string/app_name" ...>
    ...
    <meta-data android:name="com.truecaller.android.sdk.PartnerKey" android:value="@string/partnerKey"/>
    ...
    </application>
    ```
   
 6. Check if the truecaller app is present on the user's device or not by using the following method
    ```java
    TrueSDK.getInstance().isUsable()
    ```
    
    Accordingly, if the truecaller app is present on the device - Create a TrueSdkScope object by using the appropriate configurational settings and use it to initialize the TrueSDK in your android activity's onCreate method:
    
     ```java
     TrueSdkScope trueScope = new TrueSdkScope.Builder(this, sdkCallback)
		.consentMode(TrueSdkScope.CONSENT_MODE_FULLSCREEN )
		.consentTitleOption( TrueSdkScope.SDK_CONSENT_TITLE_VERIFY )
		.footerType( TrueSdkScope.FOOTER_TYPE_SKIP )
		.build();
                
     TrueSDK.init(trueScope);	
     ```
    
    TrueSDK v0.7 provides you with capabilities to configure the following settings -

    ##### - Consent Mode 
	To switch between a full screen view or an overlay view of the profile consent
	
	Possible values for Consent Mode :
	
	```java
	// To display the user's Truecaller profile in a popup view
	TrueSdkScope.CONSENT_MODE_POPUP
	
	// To display the user's Truecaller profile in a full screen view
	TrueSdkScope.CONSENT_MODE_FULLSCREEN
	```
	
    ##### - Footer Type
	To configure the CTA present at the bottom
	
	Possible values for Footer Type :
	
	```java
	// To use "USE DIFFERENT NUMBER" CTA at the bottom of the user profile view
	TrueSdkScope.FOOTER_TYPE_CONTINUE
	
	// To use "SKIP" CTA at the bottom of the user profile view
	TrueSdkScope.FOOTER_TYPE_SKIP
	```
	
    ##### - Consent Title Options
	To provide appropriate context of verification to the truecaller user 
	
	Possible values for Consent Title option :
	
	```java
	// To use "Login" as the contextual text in the user profile view title
	TrueSdkScope.SDK_CONSENT_TITLE_LOG_IN
	
	// To use "Signup" as the contextual text in the user profile view title
	TrueSdkScope.SDK_CONSENT_TITLE_SIGN_UP
	
	// To use "Sign in" as the contextual text in the user profile view title
	TrueSdkScope.SDK_CONSENT_TITLE_SIGN_IN
	
	// To use "Verify" as the contextual text in the user profile view title
	TrueSdkScope.SDK_CONSENT_TITLE_VERIFY
	
	// To use "Register" as the contextual text in the user profile view title
	TrueSdkScope.SDK_CONSENT_TITLE_REGISTER
	
	// To use "Get Started" as the contextual text in the user profile view title
	TrueSdkScope.SDK_CONSENT_TITLE_GET_STARTED
	```
       
 7. Add the following condition in the onActivityResult method:

      ```java
      TrueSDK.getInstance().onActivityResultObtained( this,resultCode, data);
      ```
      
 8. In your selected Activity

   - Either make the Activity implement ITrueCallback or create an instance. 
	This interface has 2 methods: onSuccesProfileShared(TrueProfile) and onFailureProfileShared(TrueError)
   
   ```java
       private final ITrueCallback sdkCallback = new ITrueCallback() {
       
        @Override
        public void onSuccessProfileShared(@NonNull final TrueProfile trueProfile) {
	
		// This method is invoked when the truecaller app is installed on the device and the user gives his
		// consent to share his truecaller profile 
		
		Log.d( TAG, "Verified Successfully : " + trueProfile.firstName );
        }

        @Override
        public void onFailureProfileShared(@NonNull final TrueError trueError) {
		// This method is invoked when some error occurs or if an invalid request for verification is made 
	
		Log.d( TAG, "onFailureProfileShared: " + trueError.getErrorType() );
        }

    };
    
   ```
    
   Write all the relevant logic in onSuccesProfileShared(TrueProfile) for displaying the information you have just received and onFailureProfileShared(TrueError) for handling the error and notify the user.

 9. You can trigger the Truecaller profile verification dialog anywhere in your app flow by calling the following method -        ```java TrueSDK.getInstance().getUserProfile() 
    ```
   
 10. (Optional) 
    You can set a unique requestID for every profile request with
    `TrueSDK.getInstance().setRequestNonce(customHash);`
    
    Note : The customHash must be a base64 URL safe string with a minimum character length of 8 and maximum of 64 characters


### Advanced and Optional

#### A. Server side Truecaller Profile authenticity check

Inside TrueProfile class there are 2 important fields, payload and signature. Payload is a Base64 encoding of the json object containing all profile info of the user. Signature contains the payload's signature. You can forward these fields back to your backend and verify the authenticity of the information. 

For details on the verification flow and sample code snippets, please refer the following link :
https://github.com/truecaller/backend-sdk-validation

IMPORTANT: Truecaller SDK already verifies the authenticity of the response before forwarding it to the your app.

#### B. Request-Response correlation check

Every request sent via a Truecaller app that supports truecaller SDK 0.7 has a unique identifier. This identifier is bundled into the response for assuring a correlation between a request and a response. If you want you can check this correlation yourself by:

1. You can use your own custom request identifier via the TrueClient with `TrueSDK.getInstance().setRequestNonce(customHash);`
2. In `ITrueCallback.onSuccesProfileShared(TrueProfile)` verify that the previously generated identifier matches the one in TrueProfile.requestNonce.

IMPORTANT: Truecaller SDK already verifies the Request-Response correlation before forwarding it to the your app.
