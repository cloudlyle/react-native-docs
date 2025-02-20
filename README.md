# Setting Up Android App Links with Digital Asset Links

## Prerequisites

Before setting up Android App Links, ensure you have:

- An **Android app** with a **package name** and **SHA256 certificate fingerprint**
- A **valid HTTPS domain**
- A **correctly configured Digital Asset Links file**
- Your app signed with a **release keystore** (debug keystore won't work)

---

## Step 1: Enable App Links in Android Manifest

1. Open your `AndroidManifest.xml`.
2. Inside the `<activity>` section, add an intent filter:
   ```xml
   <intent-filter android:autoVerify="true">
       <action android:name="android.intent.action.VIEW" />
       <category android:name="android.intent.category.DEFAULT" />
       <category android:name="android.intent.category.BROWSABLE" />
       <data android:scheme="https" android:host="yourdomain.com" />
   </intent-filter>
   ```

---

## Step 2: Create the Digital Asset Links File

1. Generate your app's SHA256 fingerprint using:
   ```sh
   keytool -list -v -keystore YOUR_KEYSTORE_PATH -alias YOUR_KEY_ALIAS -storepass YOUR_STORE_PASSWORD
   ```
2. Create a `assetlinks.json` file with the following content:
   ```json
   [
     {
       "relation": ["delegate_permission/common.handle_all_urls"],
       "target": {
         "namespace": "android_app",
         "package_name": "com.anyapp.android",
         "sha256_cert_fingerprints": ["YOUR_SHA256_CERTIFICATE"]
       }
     }
   ]
   ```
3. Upload the file to:
   ```
   https://yourdomain.com/.well-known/assetlinks.json
   ```
4. Ensure the file is publicly accessible and served with `Content-Type: application/json`.

---

## Step 3: Enable Digital Asset Links in Firebase

<details>
  <summary>See more.....</summary>
  
1. Go to [Firebase Console](https://console.firebase.google.com/).
2. Navigate to **Project Settings** > **General**.
3. Under **Your apps**, add your SHA256 fingerprint if not already added.
4. Navigate to **Authentication** > **Settings** > **Authorized domains**.
5. Add your domain if it's not listed.
![Screenshot 2025-02-20 at 16 53 31](https://github.com/user-attachments/assets/d42e2422-d038-444e-b4d2-603ad34e158b)

6. If not using Play App Signing, create a file named firebase-config.js in your project’s root directory with the following content:
   ```javascript
   const firebaseConfig = {
     apiKey: "YOUR_API_KEY",
     authDomain: "develop.shopping-web.com",
     databaseURL: "YOUR_DATABASE_URL",
     projectId: "YOUR_PROJECT_ID",
     appId: "YOUR_APP_ID",
   };
   ```
7. Initialize Firebase in Your App
   ```javascript
    import firebase from 'firebase/app';
    import 'firebase/auth'; // Import other Firebase services as needed
    import firebaseConfig from './firebase-config.js';

    if (!firebase.apps.length) {
        firebase.initializeApp(firebaseConfig);
    } else {
        firebase.app(); // if already initialized, use that one
    }

    ```

8. Ensure your `assetlinks.json` is accessible at `https://develop.shopping-web.com/.well-known/assetlinks.json` with correct SHA-256 fingerprints.

</details>

---

## Step 4: Verify Android App Links

1. Install the app on a **real device** (App Links do not work on emulators without deep link verification).
2. Run the following command to check if the app is associated with your domain:
   ```sh
   adb shell am start -a android.intent.action.VIEW -d "https://yourdomain.com/path" com.anyapp.android
   ```
3. Run the verification check:
   ```sh
   adb shell pm verify-app-links --re-verify com.anyapp.android
   ```
4. If the app does not open, check the verification status:
   ```sh
   adb shell dumpsys package com.anyapp.android | grep android:autoVerify
   ```

---

## Step 5: Handle Deep Links in React Native

```javascript
import { Linking } from "react-native";
import { useEffect } from "react";

const handleDeepLink = (event) => {
  const url = event.url;
  console.log("Received URL: ", url);
  // Navigate accordingly
};

useEffect(() => {
  Linking.addEventListener("url", handleDeepLink);
  return () => {
    Linking.removeEventListener("url", handleDeepLink);
  };
}, []);
```

---

## Debugging Issues

If App Links do not open your app:

- Ensure `assetlinks.json` is accessible at `https://yourdomain.com/.well-known/assetlinks.json`.
- Run:
  ```sh
  adb logcat -s IntentFilterVerification
  ```
  to check verification logs.
- Ensure the app is signed with the **release keystore**.
- Reinstall the app to force Android to refresh App Links.

---

## **Reference Documentation**

- [Android App Links](https://developer.android.com/training/app-links)
- [Digital Asset Links](https://developers.google.com/digital-asset-links/v1/getting-started)
- [Handling Deep Links in React Native](https://reactnative.dev/docs/linking)

---

## **🚀 Summary**

✅ Enable **App Links** in the Android manifest.
✅ Host a **valid `assetlinks.json` file** on your domain.
✅ Ensure the file is publicly accessible.
✅ Handle deep links in your **React Native app**.
✅ Test with real devices & debug with system logs.

Now your app should properly handle **Android App Links!** 🚀
