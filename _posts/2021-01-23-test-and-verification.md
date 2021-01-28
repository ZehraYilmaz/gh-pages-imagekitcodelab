---
title: "Test and Verification"
description: 5
---

After the project is imported, connect your mobile device to the PC and enable the USB debugging mode. In the Android Studio window, click 

<div style="padding: 5px">
        <img style="width: 50.00px ; padding: 5px" src="https://raw.githubusercontent.com/ZehraYilmaz/gh-pages-imagekitcodelab/gh-pages/assets/image_kit_codelab_doc_ss_1.png">
</div>

to run the project you have created in Android Studio to generate an APK. Then install the APK on the mobile device.

<div style="padding: 5px">
        <img style="width: 300.00px ; padding: 5px" src="https://raw.githubusercontent.com/ZehraYilmaz/gh-pages-imagekitcodelab/gh-pages/assets/image_kit_codelab_app_ss_1.png">
</div>

1. Click INIT_SERVICE. If the result code is 0, the initialization is successful.
2. Click IMAGE and select an image from the album. Currently, the following image formats are     supported for test: JPG, JPEG, PNG, and WEBP.
3. After the image is automatically loaded, set the following parameters and click GET_RESULT:

- **filterType**: The value ranges from 0 to 24. The default value is **0**.
- **intensity**: The value is greater than or equal to 0 but less than or equal to 1. The default value is **1**.
- **compressRate**: The value is greater than 0 but less than or equal to 1. The default value is **1**.

1. Check whether the image applied with the filter is displayed. If the image is not displayed, turn off hardware acceleration. For details, see step 5 in **Advanced Drill**.
2. After the test is complete, click **STOP_SERVICE** to destroy the image and stop the Image Vision service.
