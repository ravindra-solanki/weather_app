*   
  Start: Fresh  
  * Proceed to the next step  
* Language Selection  
  * Select preferred language  
  * Proceed to permissions  
* Permissions and Consent  
  * SMS Permission  
    * If granted, proceed to the next step  
    * If not granted, go to step: "Not Logged In"  
  * Set Consent API  
    * To update content  
* CT (Event)  
  * User preference selected  
  * Proceed to next step  
* Login Screen  
  * Login Success  
    * Create Finbox user  
    * Sync data  
    * Call Insights API  
* Ready for Sync  
  * Proceed to SMS Mandatory  
  * Sync data  
* Updated user  
  * Request Permission  
    * If granted, proceed to sync  
    * If not granted, go to "Not Given"  
* Loans section permission screen  
  * SMS Permission  
  * Consent api  
  * Sync data  
  * Insight api ,if not called at login screen  
* Consent setting screen  
  * Enable and disable app installed consent

* End

# **Finbox SDK Integration and User Consent Management Implementation**

## **Introduction**

This document details the integration of the Finbox SDK into an Android application, emphasizing user consent management and event tracking. The SDK manages permissions for SMS consent and app installation, and connects with the Finbox Risk Management API for user verification and data synchronization.

## **1\. Finbox SDK Integration**

### **1.1 SDK Dependencies**

To integrate the Finbox SDK, include the following dependencies in your build.gradle file:

`1implementation ("in.finbox:mobileriskmanager:$DC_SDK_VERSION:$DC_FLAVOR-release@aar") { transitive = true }`  
`2implementation ("in.finbox:common:$COMMON_SDK_VERSION:$COMMON_FLAVOR-release@aar") { transitive = true }`

`3implementation ("in.finbox:logger:$LOGGER_SDK_VERSION:parent-release@aar") { transitive = true }`

### **1.2 SDK Versions**

Ensure the following SDK versions are defined in your build.gradle file:

groovy

Run

Copy code

`1DC_SDK_VERSION='6.6.4'`  
`2DC_FLAVOR='ind'`  
`3COMMON_SDK_VERSION='2.6.0'`  
`4COMMON_FLAVOR='moneycontrol'`

`5LOGGER_SDK_VERSION='2.6.2'`

### **1.3 API Keys**

Add the appropriate API keys in the strings.xml file:

xml

Run

Copy code

`1<string name="finbox_api_key">X4x9ZmUl4o2zTQ1kfZfi77vc05F7x47g9RaXzasV</string>`

`2<string name="finbox_stg_api_key">XOgnTNH6Nu9QSBmDRyn7Qjxa6orq3NM9G1Y8Gwk3</string>`

## **2\. User Consent Flow Implementation**

### **2.1 Consent for Permissions**

The application will manage user consent for SMS permissions and app installation through a permission screen upon app launch.

### **2.2 Initial Screen**

Users are prompted for consent. If granted, consent information is stored in CleverTap and the Finbox API is notified.

### **2.3 Proceed Button Action**

Upon clicking "Proceed," the following occurs:

* Consent data is stored in CleverTap for SMS and app installation.  
* The setSmsConsentData API method is called to record consent data in Finbox.

kotlin

Run

Copy code

`1fun collectConsentData(consent: Boolean, deviceID: String) {`  
`2    CoroutineScope(Dispatchers.IO).launch {`  
`3        try {`  
`4            // Store SMS consent`  
`5            permissionData?.setSmsConsent?.let {`  
`6                finBoxViewModel.setSmsConsentData(it, deviceID, if (consent) "1" else "0", "sms")`  
`7            }`  
`8            // Store app installation consent`  
`9            permissionData?.setSmsConsent?.let {`  
`10                finBoxViewModel.setSmsConsentData(it, deviceID, if (consent) "1" else "0", "app_installed")`  
`11            }`  
`12        } catch (e: Exception) {`  
`13            FirebaseCrashlytics.getInstance().recordException(e)`  
`14            e.printStackTrace()`  
`15        }`  
`16    }`

`17}`

### **2.4 Skip Button Action**

When users skip the consent prompt, the consent data is updated accordingly.

### **2.5 Tracking Permission Events**

Events are tracked for each permission granted or denied. The triggerPermissionResultEvent method sends results.

## **3\. Consent Management in Settings**

### **3.1 Settings Page**

A settings page allows users to toggle app installation consent using a switch.

kotlin

Run

Copy code

`1binding.consentSwitch.isChecked = sharedPrefConsent.getBoolean(AppConstants.INSTALLED_APP_CONSENT, false)`

### **3.2 Handling Consent Changes**

The handleFinboxConsent method is triggered when toggling consent to update the status.

### **3.3 API Call for Consent**

The callSetApi method sends consent data to the Finbox server.

kotlin

Run

Copy code

`1private fun callSetApi(isChecked: Boolean) {`  
`2    val url = AppData.getInstance().extra_url["set_sms_consent"] ?: return`  
`3    viewLifecycleOwner.lifecycleScope.launch {`  
`4        withContext(Dispatchers.IO) {`  
`5            try {`  
`6                finBoxViewModel.setSmsConsentData(url, ParseCall.getDeviceUidOnlyForSpecialCalls(requireContext()), if (isChecked) "1" else "0", "app_installed")`  
`7            } catch (e: Exception) {`  
`8                FirebaseCrashlytics.getInstance().recordException(e)`  
`9                Log.e("ConsentSetting", "Error in API call: ${e.localizedMessage}", e)`  
`10            }`  
`11        }`  
`12    }`

`13}`

## **4\. Lending Flow Consent**

In the lending flow, the application checks for user consent before allowing progression:

kotlin

Run

Copy code

`1if (AppData.getInstance().isLoggedIn && sharedPrefConsent.getBoolean(INSTALLED_APP_CONSENT, false) && sharedPrefConsent.getBoolean(PERMISSION_SCREEN_CONSENT, false)) {`  
`2    finBoxViewModel.createUser(getString(R.string.finbox_api_key), Utility.getInstance().useR_ID)`

`3}`

## **5\. Handling New Build Updates**

When a new build is installed, the app prompts for permissions after the splash screen. Following "Proceed," the createUser method registers the user.

## **6\. Insights API**

The Insights API is called post-Finbox sync to send relevant data to the server.

kotlin

Run

Copy code

Explain

`1fun callInsightsQueue(url: String, token: String) {`  
`2    CoroutineScope(Dispatchers.IO).launch {`  
`3        mNetworkManager.callInsightsApi(object : ResponseListener<String?> {`  
`4            override fun onSuccessResponse(response: String?) {`  
`5                Log.e("FINBOX", "call insights api: $response")`  
`6            }`  
`7`  
`8            override fun onErrorResponse(errorCode: Int) {}`  
`9            override fun onErrorResponse(message: String?) {}`  
`10            override fun onResponseCode(errorCode: Int) {}`  
`11        }, url, token)`  
`12    }`

`13}`

## **Conclusion**

By following this document, the integration of Finbox SDK will ensure effective management of user consent and provide a seamless experience across the application. Use the provided code snippets and guidelines to implement the necessary functionalities.  
