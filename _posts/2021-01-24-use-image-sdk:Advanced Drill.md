---
title: "Use Image SDK:Advanced Drill"
description: 20
---

##### **Step 1**: Import the Image Vision service package

Import needed packages:

<pre><div id="copy-button12" class="copy-btn" title="Copy" onclick="copyCode(this.id)"></div><code>import com.huawei.hms.image.vision.*;
import com.huawei.hms.image.vision.bean.ImageVisionResult;<span class="pln">
</span></code></pre>

##### **Step 2: Construct an ImageVision instance**

The instance will be used to perform related operations such as obtaining the filter effect.

<pre><div id="copy-button13" class="copy-btn" title="Copy" onclick="copyCode(this.id)"></div><code>  // Obtain an ImageVisionImpl object.
  ImageVisionImpl imageVisionAPI = ImageVision.getInstance(this);<span class="pln">
</span></code></pre>

**Step 3: Initialize the service**

To call setVisionCallBack during service initialization, your app must implement the VisionCallBack API and override its onSuccess(int successCode) and onFailure(int errorCode) methods. If the ImageVision instance is successfully obtained, the onSuccess method is called and then the Image Vision service is initialized. If the ImageVision instance fails to be obtained, the onFailure method is called and an error code is returned. Your app is allowed to use the Image Vision service only after the successful verification. That is, the value of initCode must be 0, indicating that the initialization is successful.

<pre><div id="copy-button14" class="copy-btn" title="Copy" onclick="copyCode(this.id)"></div><code>private fun initFilter() {
        // Create an ImageVision instance and initialize the service.
        imageVisionAPI = ImageVision.getInstance(baseContext)
        imageVisionAPI.setVisionCallBack(object : VisionCallBack {
            override fun onSuccess(successCode: Int) {
                initCode = imageVisionAPI.init(baseContext, authJson)
                // initCode must be 0 if the initialization is successful.
                if (initCode == 0)
                    Log.d(TAG, "getImageVisionAPI rendered image successfully"
            }override fun onFailure(errorCode: Int) {
                Log.e(TAG, "getImageVisionAPI failure, errorCode = $errorCode")
                Toast.makeText(this@MainActivity, "initFailed", Toast.LENGTH_SHORT).show()
                initCode = errorCode
            }
        })
}<span class="pln">
</span></code></pre>

Description of **authJson** parameters:

| Parameter    | Type   | Mandatory or Not | Description                                                  |
| ------------ | ------ | ---------------- | ------------------------------------------------------------ |
| projectId    | String | M                | Project ID you  obtained when registering with HUAWEI Developers. |
| appId        | String | M                | Project ID you  obtained during registration.                |
| authApiKey   | String | M                | API key used  for authentication, which is provided by HUAWEI Developers. |
| clientSecret | String | M                | Client secret  used for authentication.                      |
| clientId     | String | M                | Client ID.                                                   |
| token        | String | O                | Session token,  which is used to verify an app. It is recommended that the app server obtain  the session token from the AppGallery using clientId and ClientSecret. |

<aside class="special">
	<p><strong>Note:</strong>
        <p>You can find the preceding parameters in the agconnect-services.json file. For details, please refer to the Development Guide.
For security purposes, the authentication information used in the sample code is fake. You need to enter the actual authentication information during app development.     
    </p></p>
</aside>

**Step 4: Construct a jsonObject object**

Description of **requestJson** and **imageBitmap** parameters:

| Parameter | Type       | Mandatory  or Not | Description                                                  |
| --------- | ---------- | ----------------- | ------------------------------------------------------------ |
| requestId | String     | Optional          | Request  ID provided by the Image Vision service.            |
| taskJson  | JSONObject | Mandatory         | JSON  string containing detailed service request information. |
| authJson  | JSONObject | Mandatory         | JSON  string containing authentication parameters.           |

Description of **responseJson** parameters:

| Parameter    | Type  | Mandatory  or Not | Description                                                  |
| ------------ | ----- | ----------------- | ------------------------------------------------------------ |
| Index        | int   | Optional          | Image  index for color mapping. The value ranges from 0 to 24. Value 0 indicates the  original image. |
| intensity    | float | Optional          | Filter  strength. Generally, set this parameter to 1.        |
| compressRate | Float | Optional          | Compression  rate. Compression is not performed by default. The value range is (0-1]. |

 **authJson** mapping:
 For details, see the description of **authJson** parameters in step 3.
 **filterType** mapping:

<div style="padding: 5px">
        <img style="width: 100.00px ; padding: 5px" src="https://raw.githubusercontent.com/ZehraYilmaz/gh-pages-imagekitcodelab/gh-pages/assets/image-kit-codelab-doc-ss-2.png">
</div>

**Step 5: Obtain the rendered image**

Make sure that you can the getColorFilter() API in the subthread rather than the main thread. If the image cannot be displayed during the test, you are advised to turn off hardware acceleration.

<pre><div id="copy-button15" class="copy-btn" title="Copy" onclick="copyCode(this.id)"></div><code>  //Obtain the rendering result from visionResult.
  ImageVisionResult visionResult =     
  imageVisionAPI.getColorFilter(requestJson,imageBitmap);<span class="pln">
</span></code></pre>

Description of **visionResult** parameters:

| Parameter    | Type       | Mandatory  or Not | Description                                                  |
| ------------ | ---------- | ----------------- | ------------------------------------------------------------ |
| resultCode   | int        | Mandatory         | Result  code.                                                |
| responseJson | JsonObject | Optional          | JSON  string containing the returned result.                 |
| image        | Bitmap     | Optional          | Processed  image. For an API that directly returns images, this parameter is used by the  API to transfer the processed image. |

Description of **responseJson** parameters:

| Parameter | Type   | Mandatory  or Not | Description                                                  |
| --------- | ------ | ----------------- | ------------------------------------------------------------ |
| requestId | String | Optional          | Request  ID provided by the Image Vision service. This parameter is available only if  it is carried in the service request. |
| serviceId | String | Mandatory         | ID of the  service being called.                             |

We will perform asynchronous operations with Kotlin coroutines to protect battery life and prevent memory leak by making a performance-oriented image filtering application.

In our scenario, a user selects an image from the Gallery. Call Init filter method and then start filtering images one by one which are located in recyclerView. (24 different items included in recyclerview to show all the filter types.)

All the sub-images were filtered by different filter types. 

<pre><div id="copy-button16" class="copy-btn" title="Copy" onclick="copyCode(this.id)"></div><code>private fun startFilterForSubImages() {
    subBitmapList.clear()
    viewModel.getAllFilterItems(null)
    recylerFilter.scrollToPosition(0)
    channelIsFetching = true
    coroutineScope.launch {
        if (filterList.isNotEmpty()) {
            initFilter() // initialize the vision service
            for (x in 1..filterList.size) {
                startFilter(x.toString(), "1", "1") // start filtering
                var bitmap = channel.receive()
                subBitmapList.put(x, bitmap)
                viewModel.getAllFilterItems(subBitmapList)
            }
            stopFilter() // stop the vision service
            channelIsFetching = false
        }
    }
  }<span class="pln">
</span></code></pre>

<pre><div id="copy-button17" class="copy-btn" title="Copy" onclick="copyCode(this.id)"></div><code>private fun startFilter(filterType: String, intensity: String, compress: String) {
    if (initCode == 0) {
        coroutineScope.launch { // CoroutineScope is used for the async     calls
            val jsonObject = JSONObject()
            val taskJson = JSONObject()
            try {
                taskJson.put("intensity", intensity) //Filter strength. Generally, set this parameter to 1.
                taskJson.put("filterType", filterType) // 24 different filterType code
                taskJson.put("compressRate", compress) // Compression ratio.
                jsonObject.put("requestId", "1") // Optinal
                jsonObject.put("taskJson", taskJson)
                jsonObject.put("authJson", authJson) // App can use the service only after it is successfully authenticated.
                coroutineScope.launch {
                    visionResult = withContext(Dispatchers.IO) {
                        imageVisionAPI?.getColorFilter(
                            jsonObject,
                            bitmapFromGallery
                        )
                    }
                    // Obtain the rendering result from visionResult
                    val image = visionResult?.image
                    if (image == null)
                        Log.e(TAG, "FilterException: Couldn't render the image. Check the restrictions while rendering an image by Image Vision Service")
                    channel.send(image)
                    // Sending image bitmap with an async channel to make it receive with another channel
                }
            } catch (e: JSONException) {
                Log.e(TAG, "JSONException: " + e.message)
            }
        }
    }
    else {
        Toast.makeText(this@MainActivity, "initFailed", Toast.LENGTH_SHORT).show()
    }
 }<span class="pln">
</span></code></pre>

Then user clicks a filter to render the selected image we provide and the Image Vision Result object returns the filtered bitmap. So we need to implement onSelected method of the interface to our activity which gets the FilterItem object of the clicked item from the adapter.

We will use same startFilter method for the sub-images and the selected image.

<pre><div id="copy-button18" class="copy-btn" title="Copy" onclick="copyCode(this.id)"></div><code> // Initialize and start a filter operation when a filter item is selected
  override fun onSelected(item: BaseInterface) {
    if (!channelIsFetching)
    {
    if (bitmapFromGallery == null)
    Toast.makeText(baseContext, getString(R.string.select_photo_from_gallery), Toast.LENGTH_SHORT).show()
    else
    {
        var filterItem = item as FilterItem
        coroutineScope.launch {
        initFilter() // initialize the vision service
        startFilter(filterItem.filterId, "1", "1") // intensity and compress are 1
           var receivedBitmap = withContext(Dispatchers.IO){channel.receive()} // receive the filtered bitmap result from another channel
           imgView.setImageBitmap(receivedBitmap)
           stopFilter() // stop the vision service
        }
    }
  }
    else
        Toast.makeText(baseContext, getString(R.string.wait_to_complete_filtering), Toast.LENGTH_SHORT).show()
 }<span class="pln">
</span></code></pre>

**Step 6: Stop the Image Vision service**

If you do not need to use filters any longer, call the **imageVisionAPI.stop()** API.

<pre><div id="copy-button19" class="copy-btn" title="Copy" onclick="copyCode(this.id)"></div><code> private fun stopFilter() {
    if(imageVisionAPI != null)
    imageVisionAPI.stop() // Stop the service if you don't need anymore 
 }<span class="pln">
</span></code></pre>

To turn off hardware acceleration, configure the **AndroidManifest.xml** file as follows:

<pre><div id="copy-button20" class="copy-btn" title="Copy" onclick="copyCode(this.id)"></div><code><span class="tag">&lt;application></span>
        ...
        android:hardwareAccelerated="false" 
        ...
 <span class="tag">&lt;/application></span><span class="pln">
</span></code></pre>

