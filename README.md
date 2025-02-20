# Setting Up Universal Links on iOS

## Prerequisites

Before setting up Universal Links, ensure you have:

- An **iOS app** with an **App ID** and an **associated domain**
- A **valid HTTPS domain**
- A **correctly configured Apple App Site Association (AASA) file**
- An app with **Associated Domains capability enabled**

---

## Step 1: Enable Associated Domains in Xcode

1. Open **Xcode** and go to your project settings.
2. Select your **app target**.
3. Navigate to the **Signing & Capabilities** tab.
4. Click `+ Capability` and add **Associated Domains**.
5. Under Associated Domains, add entries in the following format:
   ```
   applinks:yourdomain.com
   ```
   Example:
   ```
   applinks:develop.123.com
   ```

---

## Step 2: Create the Apple App Site Association (AASA) File

1. Create a file named `apple-app-site-association` **without an extension**.
2. Add the following JSON content:
   ```json
   {
     "applinks": {
       "apps": [],
       "details": [
         {
           "appID": "1234.com.anyapp.app.dev",
           "paths": ["NOT/_/*", "/*"]
         },
         {
           "appID": "1234.com.anyapp.app.uat",
           "paths": ["NOT/_/*", "/*"]
         },
         {
           "appID": "1234.com.anyapp.app",
           "paths": ["NOT/_/*", "/*"]
         }
       ]
     }
   }
   ```
   - `appID`: This is a unique identifier for your app, formatted as `<TeamID>.<BundleIdentifier>`.
   - `paths`: This specifies which URLs should be handled by the app.
     - `"NOT/_/*"`: Excludes paths starting with `/_/` from opening the app.
     - `"/*"`: Allows all other paths to open the app.
3. Ensure the file is **served with `Content-Type: application/json`**.

---

## Step 3: Host the AASA File

1. Upload `apple-app-site-association` to your server at:
   ```
   https://yourdomain.com/.well-known/apple-app-site-association
   ```
2. Verify it is publicly accessible by running:
   ```sh
   curl -i https://yourdomain.com/.well-known/apple-app-site-association
   ```
   It should return valid JSON with `Content-Type: application/json`.
3. **Important Note:** Ensure that `https://yourdomain.com/` does not return any content before hosting the AASA file. If it does, it may conflict with your app, preventing it from opening via Universal Links.

---

## Step 4: Verify Universal Links

1. Install the app on a **real device** (Universal Links do not work in the iOS Simulator).
2. Restart the device to refresh Apple’s domain validation.
3. Test deep linking by running:
   ```sh
   xcrun simctl openurl booted https://yourdomain.com/path
   ```
4. If the link still opens in Safari, check your device logs:
   ```sh
   log stream --predicate 'subsystem == "com.apple.coreservices.useractivityd"'
   ```

---

## Step 5: Handle Universal Links in Your React Native App

### **Listening for Universal Links in React Native**

Add the following code inside your **React Native app**:

```javascript
import { Linking } from "react-native";
import { useEffect } from "react";

const handleDeepLink = (event) => {
  const url = event.url;
  console.log("Received URL: ", url);
  // Parse URL and navigate accordingly
};

useEffect(() => {
  Linking.addEventListener("url", handleDeepLink);
  return () => {
    Linking.removeEventListener("url", handleDeepLink);
  };
}, []);
```

### **Handling Deep Links When App is Closed**

Modify `App.js`:

```javascript
import { useEffect } from "react";
import { Linking } from "react-native";

const App = () => {
  useEffect(() => {
    Linking.getInitialURL().then((url) => {
      if (url) {
        handleDeepLink({ url });
      }
    });
  }, []);

  return <YourAppNavigation />;
};
```

---

## Step 6: Debugging Issues

If Universal Links do not open your app:

- Ensure the AASA file is accessible via `https://yourdomain.com/.well-known/apple-app-site-association`
- Run:
  ```sh
  log stream --predicate 'subsystem == "com.apple.coreservices.useractivityd"'
  ```
  to check validation logs
- Check your Associated Domains setup in **Xcode**
- Reinstall the app to force Apple to refresh Universal Links

---

## **Reference Documentation**

- [Apple Universal Links](https://developer.apple.com/documentation/xcode/supporting-universal-links-in-your-app)
- [Handling Deep Links in React Native](https://reactnative.dev/docs/linking)
- [Apple App Site Association File](https://developer.apple.com/documentation/bundleresources/applesupportfiles/apple-app-site-association)

---

## **🚀 Summary**

✅ Enable **Associated Domains** in Xcode.  
✅ Host a **valid AASA file** on your domain.  
✅ Ensure the file is publicly accessible.  
✅ Handle deep links in your **React Native app**.  
✅ Test with real devices & debug with system logs.

Now your app should properly handle **Universal Links on iOS!** 🚀
