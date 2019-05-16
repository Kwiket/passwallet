# PassWallet Android mobile app
Collect all your Passbook / Apple Wallet (.pkpass files) tickets & coupons, from flight boarding passes to coffee shop loyalty cards in one easy to use application.

# Sharing a pass into PassWallet
## Open PassWallet and auto import a pass by link
Just create a link
```
https://passwallet.page.link/?apn=com.attidomobile.passwallet&link=$HTTP_LINK_TO_PASS_HERE
```
and send it via email or other messengers, for example:
```
https://passwallet.page.link/?apn=com.attidomobile.passwallet&link=https://quicket.io/demopass/AttidoMobile.pkpass
```
The following link will launch PassWallet Android app if installed, or launch Google Play if not.
The pass will be imported automatically into the app after launch.

## Launching PassWallet from an Android app
The following code will launch PassWallet from an Android app if installed, or launch Google Play if not.
The “referrer” will be the URL of the pass you’re trying to install and will be automatically imported on the device when PassWallet is first run.

### Java sample:
```java
private static boolean launchPassWallet(Context applicationContext, Uri uri, boolean launchGooglePlay) {
    if (null != applicationContext) {
        PackageManager packageManager = applicationContext.getPackageManager();
        if (null != packageManager) {
            final String strPackageName = "com.attidomobile.passwallet";
            Intent startIntent = new Intent();
            startIntent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
            startIntent.setAction(Intent.ACTION_VIEW);
            Intent passWalletLaunchIntent = packageManager.getLaunchIntentForPackage(strPackageName);
            if (null == passWalletLaunchIntent) {
                // PassWallet isn't installed, open Google Play:
                if (launchGooglePlay) {
                    String strReferrer = "";
                    try {
                        final String strEncodedURL = URLEncoder.encode(uri.toString(), "UTF-8");
                        strReferrer = "&referrer=" + strEncodedURL;
                    } catch (UnsupportedEncodingException e) {
                        e.printStackTrace();
                        strReferrer = "";
                    }
                    try {
                        startIntent.setData(Uri.parse("market://details?id=" + strPackageName + strReferrer));
                        applicationContext.startActivity(startIntent);
                    } catch (android.content.ActivityNotFoundException anfe) {
                        // Google Play not installed, open via website
                        startIntent.setData(Uri.parse("http://play.google.com/store/apps/details?id=" + strPackageName + strReferrer));
                        applicationContext.startActivity(startIntent);
                    }
                }
            } else {
                final String strClassName = "com.attidomobile.passwallet.ui.main.MainActivity";
                startIntent.setClassName(strPackageName, strClassName);
                startIntent.addCategory(Intent.CATEGORY_BROWSABLE);
                startIntent.setDataAndType(uri, "application/vnd.apple.pkpass");
                applicationContext.startActivity(startIntent);
                return true;
            }
        }
    }
    return false;
}
```
And an example call is:
```java
launchPassWallet(getApplicationContext(), Uri.parse("https://quicket.io/demopass/AttidoMobile.pkpass"), true);
```
### Kotlin sample:
```kotlin
private fun launchPassWallet(applicationContext: Context?, uri: Uri, launchGooglePlay: Boolean): Boolean {
    if (null != applicationContext) {
        val packageManager = applicationContext.packageManager
        if (null != packageManager) {
            val strPackageName = "com.attidomobile.passwallet"
            val startIntent = Intent()
            startIntent.flags = Intent.FLAG_ACTIVITY_NEW_TASK
            startIntent.action = Intent.ACTION_VIEW
            val passWalletLaunchIntent = packageManager.getLaunchIntentForPackage(strPackageName)
            if (null == passWalletLaunchIntent) {
                // PassWallet isn't installed, open Google Play:
                if (launchGooglePlay) {
                    val strReferrer = try {
                        val strEncodedURL = URLEncoder.encode(uri.toString(), "UTF-8")
                        "&referrer=$strEncodedURL"
                    } catch (e: UnsupportedEncodingException) {
                        e.printStackTrace()
                        ""
                    }

                    try {
                        startIntent.data = Uri.parse("market://details?id=$strPackageName$strReferrer")
                        applicationContext.startActivity(startIntent)
                    } catch (anfe: android.content.ActivityNotFoundException) {
                        // Google Play not installed, open via website
                        startIntent.data = Uri.parse("http://play.google.com/store/apps/details?id=$strPackageName$strReferrer")
                        applicationContext.startActivity(startIntent)
                    }
                }
            } else {
                val strClassName = "com.attidomobile.passwallet.ui.main.MainActivity"
                startIntent.setClassName(strPackageName, strClassName)
                startIntent.addCategory(Intent.CATEGORY_BROWSABLE)
                startIntent.setDataAndType(uri, "application/vnd.apple.pkpass")
                applicationContext.startActivity(startIntent)
                return true
            }
        }
    }
    return false
}
```

# Push Notifications
## Registering a Device to Receive Push Notifications for a Pass
In addition to the web service detailed in the Apple document [Passbook Web Service Reference](https://developer.apple.com/library/archive/documentation/PassKit/Reference/PassKit_WebService/WebService.html), we require an additional endpoint for registering PassWallet devices.
This has been kept similar to the standard Apple registration endpoint in order to reduce the work required, but is created as a new endpoint so as to not cause potential issues with services that do not support PassWallet’s update method.
Registration is performed via a POST request to:

`webServiceURL/version/devices/deviceLibraryIdentifier/registrations_attido/passTyp
eIdentifier/serialNumber`

### Parameters
* **webServiceURL** - The URL to your web service, as specified in the pass.
* **version** - The protocol version. Currently, v1.
* **deviceLibraryIdentifier** - A unique identifier that is used to identify and
authenticate this device in future requests.
* **passTypeIdentifier** - The pass’s type, as specified in the pass.
* **serialNumber** - The pass’s serial number, as specified in the pass.

### Header
The Authorization header is supplied; its value is the word “AttidoPass”, followed by a space, followed by the pass’s authorization token as specified in the pass.

### Payload
The POST payload is a JSON dictionary, containing two key and value pairs:
* **pushToken** - The pushtoken that the server can use to send push
notifications to this device.
* **pushServiceUrl** - The URL of PassWallet’s web service for posting update
notifications to devices.

### Response
* If the serial number is already registered for this device, return HTTP status 200.
* If registration succeeds, return HTTP status 201.
* If the request is not authorized, return HTTP status 401.
* Otherwise, return the appropriate standard HTTP status.

### Discussion
* The URL of PassWallet’s server should be stored on a per registration basis, and used when an update to the pass occurs.


# FAQ
Can be found here: https://www.facebook.com/notes/passwallet/passwallet-faq/546965562018601
