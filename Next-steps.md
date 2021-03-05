# Next steps for getting your PWA into HUAWEI AppGallery app store
You've successfully generated a HUAWEI AppGallery app package for your PWA. 😎

Your next steps:
1. **Test your app**: install and launch the `.apk` file on a HUAWEI Android device or Cloud Debugging.
2. **Create an app on AppGallery**: follow the instruction below.
3. **Upload your app to AppGallery**: upload the `.apk` file to the AppGallery.

Each step is explained below.

## 1. Test your app

Your zip file contains an `.apk` (Android Package) file, which can be loaded on your personal HUAWEI Android device.

**To test your app, install open the `.apk` file on your Android device or upload to Cloud Debugging remote device.**

If you don't have a physical HUAWEI Android device, you can use a remote physical HUAWEI device from [Cloud Debugging](https://developer.huawei.com/consumer/en/agconnect/cloud-adjust/). Upload the `.apk` file to install and test your app.

> 💁‍♂️ *Heads up*: 
> 
> To use Cloud Debugging, you need to have a HUAWEI developer account. If you don't have one, please follow this [instruction](https://developer.huawei.com/consumer/en/doc/start/registration-and-verification-0000001053628148) to create it.

## 2. Create an app on AppGallery

If you are a HUAWEI developer, please follow the instruction to [create an AppGallery app](https://developer.huawei.com/consumer/en/doc/distribution/app/agc-create_app).

> 💁‍♂️ *Heads up*: 
> 
> If you wish to integrate **[HMS capabilities](https://developer.huawei.com/consumer/en/hms/)** (Huawei Mobile Services) such as [Ads Kit](https://developer.huawei.com/consumer/en/hms/huawei-adskit/), [Push Kit](https://developer.huawei.com/consumer/en/hms/huawei-pushkit/), [Analytics Kit](https://developer.huawei.com/consumer/en/hms/huawei-analyticskit/), please follow the [official doc here](https://developer.huawei.com/consumer/en/doc/development/AppGallery-connect-Guides/agc-get-started) to download the `agconnect-services.json`. This JSON is required to connect your SDKs with HMS backend api. 

## 3. Upload your app to AppGallery

Your zip file contains an `.apk` file which can be submitted directly to HUAWEI AppGallery through the [HUAWEI Developer Console](https://developer.huawei.com/consumer/en/service/josp/agc/index.html#/).

Please follow the instruction to [upload and publish your app](https://developer.huawei.com/consumer/en/doc/distribution/app/agc-release_app). Make sure that you review [Self Check Before Review](https://developer.huawei.com/consumer/en/doc/distribution/app/agc-auto_check) first.

Once you submit your app, it will be reviewed. Once approved, your PWA will be available in the HUAWEI AppGallery. 😎

## Save your signing key

Your zip file contains `signing.keystore` and `signing-key-info.txt`:

- `signing.keystore` is the Android key store file containing the signing key.
- `signing-key-info.txt` is a text file containing your signing key information, such as the key password, store password, and key alias.

Keep both of these files in a safe place. **You'll need them to deploy future versions of your app.** See [Uploading a new version](#uploading-a-new-version) for more info.

## Need more help?

If you're otherwise stuck, we're here to help. You can email us - [dtse02@futurewei.com](mailto:dtse02@futurewei.com) and we'll help walk you through it.