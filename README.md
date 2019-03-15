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
https://passwallet.page.link/?apn=com.attidomobile.passwallet&link=http://test.attidomobile.com/PassWallet/Passes/AttidoMobile.pkpass
```
The following link will launch PassWallet from an Android app if installed, or launch Google Play if not.
The pass will be imported into the app after launch.

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
                final String strClassName = "com.attidomobile.passwallet.activity.TicketDetailActivity";
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
launchPassWallet(getApplicationContext(), Uri.parse("http://test.attidomobile.com/PassWallet/Passes/AttidoMobile.pkpass"), true);
```
### Kotlin samle:

# FAQ
Can be found here: https://www.facebook.com/notes/passwallet/passwallet-faq/546965562018601
