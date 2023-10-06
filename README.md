# Pixelbin kotlin library

Pixelbin kotlin library helps you integrate Pixelbin with your Android Application.

## Usage

### Setup
Add it in your root build.gradle at the end of repositories:
```
//jitpack
allprojects {
	repositories {
		...
		maven { url 'https://jitpack.io' }
	}
}
```
Add the dependency
```
dependencies {
//jitpack
    implementation 'com.github.pixelbin-dev:pixelbin-kotlin-sdk:version'
//maven central   
    implementation 'io.github.pixelbin-dev:pixelbin-kotlin-sdk:version'
}
```
For maven
```
//jitpack
//add jitpack repository to your build file
<repositories>
	<repository>
	    <id>jitpack.io</id>
	    <url>https://jitpack.io</url>
	</repository>
</repositories>

//add the dependency
<dependency>
	<groupId>com.github.pixelbin-dev</groupId>
    <artifactId>pixelbin-kotlin-sdk</artifactId>	   
    <version>v0.0.3</version>
</dependency>
	
//maven central
<dependency>
    <groupId>io.github.pixelbin-dev</groupId>
    <artifactId>pixelbin-kotlin-sdk</artifactId>
    <version>v0.0.3</version>
</dependency>
```
Import the Pixelbin class

```
import com.pixelbin.PixelBin
```

Create your instance

```
val pixelbin = PixelBin.getInstance()
val image = pixelbin.url("https://cdn.pixelbin.io/v2/cloudName/z-slug/transformation/path/to/image.jpeg")
    
val image = pixelbin.url(
    UrlObj(
        baseUrl = baseUrl,
        version = version,
        cloudName = cloudName,
        zone = zone,
        transformation = transformationList,
        filePath = imagePath
    )
)
```

---

### Transform and Optimize Images

Import transformations

```
import com.pixelbin.transformation.Transformation

val eraseTransformation = Transformation.erasebg();

// Create a new instance. If you have't (see above for the details)
val pixelbin = PixelBin.getInstance()
val image = pixelbin.url(imageUrl)

// Create EraseBg.bg transformation
// Kotlin
val t1 = Transformation.erasebg();

//Java
TransformationObj t1 = Transformation.INSTANCE.erasebg();

// Create resize transformation
// Kotlin
val t2 = Transformation.resize(height = 100,width = 100)

//Java
TransformationObj t2 = Transformation.INSTANCE.resize(height = 100,width = 100);

// Add the transformations to the image object
//add single transformation
image.addTransformation(t1);

//or add multiple transformation
//Kotlin
image.addTransformation(arrayListOf(t1,t2));

//Java
ArrayList<TransformationObj> list = new ArrayList<>();
list.add(t1);
list.add(t2);
image.add(list);

// Get the image url
image.getUrl()
// output
// https://cdn.pixelbin.io/v2/cloudName/z-slug/erase.bg()~t.resize(h:100,w:100)/path/to/image.jpeg
```

### upload(file, signedDetails):

| parameter                                                            | type                                                                                                                                 |
| -------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| file ([File](https://developer.mozilla.org/en-US/docs/Web/API/File)) | File to upload to Pixelbin                                                                                                           |
| signedDetails (Object)                                               | `signedDetails` can be generated with the Pixelbin Backend SDK [@pixelbin/admin](https://github.com/pixelbin-dev/pixelbin-js-admin). |

-   Resolves with no response on success.
-   Rejects with error on failure.

Example :

1. Define a file element

```
val file =  File(pathname);
```

2. Use the presignedUrl generated with the backend sdk. [click here](https://github.com/pixelbin-dev/pixelbin-js-admin/blob/main/documentation/platform/ASSETS.md#createsignedurl).

```
val signedDetails = SignedDetails(url = "url",fields = fieldsToHashMap(fields))

//fields refer to hashmap of fields object which we got from signed url api 
//Example
data class Fields(
    @SerializedName("key") var key: String? = null,
    @SerializedName("x-amz-meta-assetData") var xAmzMetaAssetData: String? = null,
    @SerializedName("x-amz-meta-token") var xAmzMetaToken: String? = null,
    @SerializedName("bucket") var bucket: String? = null,
    @SerializedName("X-Amz-Algorithm") var xAmzAlgorithm: String? = null,
    @SerializedName("X-Amz-Credential") var xAmzCredential: String? = null,
    @SerializedName("X-Amz-Date") var xAmzDate: String? = null,
    @SerializedName("Policy") var policy: String? = null,
    @SerializedName("X-Amz-Signature") var xAmzSignature: String? = null
)

fun fieldsToHashMap(fields: Fields): HashMap<String, String> {
        val hashMap = HashMap<String, String>()
        hashMap["key"] = fields.key?:""
        hashMap["x-amz-meta-assetData"] = fields.xAmzMetaAssetData?:""
        hashMap["x-amz-meta-token"] = fields.xAmzMetaToken?:""
        hashMap["bucket"] = fields.bucket?:""
        hashMap["X-Amz-Algorithm"] = fields.xAmzAlgorithm?:""
        hashMap["X-Amz-Credential"] = fields.xAmzCredential?:""
        hashMap["X-Amz-Date"] = fields.xAmzDate?:""
        hashMap["Policy"] = fields.policy?:""
        hashMap["X-Amz-Signature"] = fields.xAmzSignature?:""
        return hashMap
}

//for kotlin
CoroutineScope(Dispatchers.IO).launch {
    PixelBin.getInstance().upload(file,signedDetails){
    //here Result class is from com.pixelbin.upload.Result
            when(it){
                is Result.Success ->{
                    val response = it.data
                }
                is Result.Failure ->{
                    val response = it.response
                }
                is Result.Error ->{
                    val exception = it.exception
                }
                else -> {}
            }
        }
}
//for java
  pixelbin.upload(file,signedDetails, result->{
  //here Result class is from com.pixelbin.upload.Result
      if (result instanceof Result.Success) {
                System.out.println("Upload successful");
          } else if (result instanceof Result.Failure) {
                Result.Failure failure = (Result.Failure) result;
                System.out.println("Upload failed: " + failure.getResponse());
            } else if (result instanceof Result.Error) {
                Result.Error error = (Result.Error) result;
                System.out.println("Error during upload: " + error.getException().getMessage());
            }
            return null;
        });
```

## Utilities

Import the utils class

```
import com.pixelbin.Utils
```

Pixelbin provides url utilities to construct and deconstruct Pixelbin urls.

### urlToObj

Deconstruct a pixelbin url

| parameter            | description          | example                                                                                               |
| -------------------- | -------------------- | ----------------------------------------------------------------------------------------------------- |
| pixelbinUrl (string) | A valid pixelbin url | `https://cdn.pixelbin.io/v2/your-cloud-name/z-slug/t.resize(h:100,w:200)~t.flip()/path/to/image.jpeg` |

**Returns**:

| property                | description                            | example                    |
| ----------------------- | -------------------------------------- | -------------------------- |
| cloudName (string)      | The cloudname extracted from the url   | `your-cloud-name`          |
| zone (string)           | 6 character zone slug                  | `z-slug`                   |
| version (string)        | cdn api version                        | `v2`                       |
| transformations (array) | Extracted transformations from the url |                            |
| filePath                | Path to the file on Pixelbin storage   | `/path/to/image.jpeg`      |
| baseUrl (string)        | Base url                               | `https://cdn.pixelbin.io/` |

Example:

```
val pixelbinUrl =
    "https://cdn.pixelbin.io/v2/your-cloud-name/z-slug/t.resize(h:100,w:200)~t.flip()/path/to/image.jpeg";

//string representation of url object
//name of transformation = plugin+"."+name

val obj = Utils.urlToUrlObj(pixelbinUrl)
//        UrlObj(
//            baseUrl = "https://cdn.pixelbin.io/",
//            version = "v2",
//            cloudName = "your-cloud-name",
//            transformation= arrayListOf(
//                TransformationObj(
//                    plugin = "t",
//                    name = "resize",
//                    values= hashMapOf(
//                      "w" to "200",
//                      "h" to "100",
//                   )
//                ),
//                TransformationObj(
//                    plugin = "t",
//                    name = "flip",
//                )
//                            ),
//                zone =" z-slug",
//            filePath = "path/to/image.jpeg"
//        )

val pixelbinUrl =
    "https://cdn.pixelbin.io/v2/your-cloud-name/z-slug/t.resize(h:100,w:200)~t.flip()/path/to/image.jpeg?dpr=2.0&f_auto=True";
```

```
//string representation of url object
//name of transformation = plugin+"."+name
val obj = Utils.urlToUrlObj(pixelbinUrl)
//        UrlObj(
//            baseUrl = "https://cdn.pixelbin.io/",
//            version = "v2",
//            cloudName = "your-cloud-name",
//            transformation = arrayListOf(
//                TransformationObj(
//                    plugin = "t",
//                    name = "resize",
//                    values= hashMapOf(
//                      "w" to "200",
//                      "h" to "100",
//                      )
//                ),
//                TransformationObj(
//                    plugin="t",
//                    name="flip",
//                )
//                            ),
//            zone = "z-slug",
//            options = hashMapOf(
//                  "dpr" to "2.0",
//                  "f_auto" to "true"
//              ),
//            filePath = "path/to/image.jpeg"
//        )
```

### objToUrl

Converts the extracted url obj to a Pixelbin url.

| property                | description                            | example                    |
| ----------------------- | -------------------------------------- | -------------------------- |
| cloudName (string)      | The cloudname extracted from the url   | `your-cloud-name`          |
| zone (string)           | 6 character zone slug                  | `z-slug`                   |
| version (string)        | cdn api version                        | `v2`                       |
| transformations (array) | Extracted transformations from the url |                            |
| filePath                | Path to the file on Pixelbin storage   | `/path/to/image.jpeg`      |
| baseUrl (string)        | Base url                               | `https://cdn.pixelbin.io/` |

Example

```
val obj = UrlObj(
    cloudName = "your-cloud-name",
    zone = "z-slug",
    version = "v2",
    transformation = arrayListOf(
               TransformationObj(
                    plugin = "t",
                    name = "resize",
                    values= hashMapOf(
                       "w" to "200",
                       "h" to "100",
                    )
                ),
                TransformationObj(
                    plugin = "t",
                    name = "flip",
                )
    ),
    filePath = "path/to/image.jpeg",
    baseUrl = "https://cdn.pixelbin.io"
)


val url = Utils.objToUrl(pixelbinUrl) // obj is as shown above
// url
// https://cdn.pixelbin.io/v2/your-cloud-name/z-slug/t.resize(h:100,w:200)~t.flip()/path/to/image.jpeg
```

```
val obj = UrlObj(
    cloudName = "your-cloud-name",
    zone = "z-slug",
    version = "v2",
    transformation = arrayListOf(
               TransformationObj(
                    plugin = "t",
                    name = "resize",
                    values= hashMapOf(
                       "w" to "200",
                       "h" to "100",
                    )
                ),
                TransformationObj(
                    plugin = "t",
                    name = "flip()",
                )
    ),
    filePath = "path/to/image.jpeg",
    baseUrl = "https://cdn.pixelbin.io",
    options = hashMapOf(
        "dpr" to "2.0",
        "f_auto" to "true"
    )
)

val url = Utils.objToUrl(pixelbinUrl)  // obj is as shown above
// url
// https://cdn.pixelbin.io/v2/your-cloud-name/z-slug/t.resize(h:100,w:200)~t.flip()/path/to/image.jpeg?dpr=2.0&f_auto=True
```

## Transformation

A transformation is an operation or a list of operations that can be performed on an image. Please refer [list of supported transformations](#list-of-supported-transformations) for details.

Example

```
// Create resize transformation
val t2 = Transformation.resize(height = 100,width = 100)

// Transformations can be set on an image by using `addTransformation` on the imageUrl object.
//add single transformation
image.addTransformation(t1);

//or add multiple transformation
image.addTransformation(arrayListOf(t1,t2));

To get the url of the image with the applied transformations, use the `getUrl` on the image object.
image.getUrl()
```

For a working example, refer [here](#transform-and-optimize-images)

## List of supported transformations

### 1. DetectBackgroundType

<details>
<summary>1. detect</summary>

#### Usage Example

```
val t = Transformation.detectbackgroundtype();
```

</details>

### 2. Artifact

<details>
<summary>1. remove</summary>

#### Usage Example

```
val t = Transformation.artifact();
```

</details>

### 3. AWSRekognitionPlugin

<details>
<summary>1. detectLabels</summary>

#### Supported Configuration

| Parameter         | Type    | Default |
| ----------------- | ------- | ------- |
| maximumLabels     | integer | 5       |
| minimumConfidence | integer | 55      |

#### Usage Example

```
val t = Transformation.detectlabels(maximumlabels = 5, minimumconfidence = 55);
```

</details>

<details>
<summary>2. moderation</summary>

#### Supported Configuration

| Parameter         | Type    | Default |
| ----------------- | ------- | ------- |
| minimumConfidence | integer | 55      |

#### Usage Example

```
val t = Transformation.moderation(minimumconfidence = 55);
```

</details>

### 4. BackgroundGenerator

<details>
<summary>1. bg</summary>

#### Supported Configuration

| Parameter        | Type                          | Default                                                                                          |
| ---------------- | ----------------------------- | ------------------------------------------------------------------------------------------------ |
| backgroundPrompt | custom                        | `cmVhbGlzdGljIGdyZWVuIGdyYXNzLCBsYXduIGZpZWxkIG9mIGdyYXNzLCBibHVlIHNreSB3aXRoIHdoaXRlIGNsb3Vkcw` |
| focus            | enum: `Product`, `Background` | `Product`                                                                                        |
| negativePrompt   | custom                        | ``                                                                                               |
| seed             | integer                       | 123                                                                                              |

#### Usage Example

```
val t = Transformation.backgroundgenerator(backgroundprompt = "cmVhbGlzdGljIGdyZWVuIGdyYXNzLCBsYXduIGZpZWxkIG9mIGdyYXNzLCBibHVlIHNreSB3aXRoIHdoaXRlIGNsb3Vkcw", focus = BackgroundGenerator.Focus.PRODUCT, negativeprompt = "", seed = 123);
```

</details>

### 5. EraseBG

<details>
<summary>1. bg</summary>

#### Supported Configuration

| Parameter    | Type                                | Default   |
| ------------ | ----------------------------------- | --------- |
| industryType | enum: `general`, `ecommerce`, `car` | `general` |
| addShadow    | boolean                             | false     |

#### Usage Example

```
val t = Transformation.erasebg(industrytype = EraseBG.Industrytype.GENERAL, addshadow = false);
```

</details>

### 6. GoogleVisionPlugin

<details>
<summary>1. detectLabels</summary>

#### Supported Configuration

| Parameter     | Type    | Default |
| ------------- | ------- | ------- |
| maximumLabels | integer | 5       |

#### Usage Example

```
val t = Transformation.googlevisionplugin(maximumlabels = 5);
```

</details>

### 7. ImageCentering

<details>
<summary>1. detect</summary>

#### Supported Configuration

| Parameter          | Type    | Default |
| ------------------ | ------- | ------- |
| distancePercentage | integer | 10      |

#### Usage Example

```
val t = Transformation.imagecentering(distancepercentage = 10);
```

</details>

### 8. IntelligentCrop

<details>
<summary>1. crop</summary>

#### Supported Configuration

| Parameter              | Type                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              | Default  |
| ---------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- |
| requiredWidth          | integer                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           | 0        |
| requiredHeight         | integer                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           | 0        |
| paddingPercentage      | integer                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           | 0        |
| maintainOriginalAspect | boolean                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           | false    |
| aspectRatio            | string                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            | ``       |
| gravityTowards         | enum: `object`, `foreground`, `face`, `none`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      | `none`   |
| preferredDirection     | enum: `north_west`, `north`, `north_east`, `west`, `center`, `east`, `south_west`, `south`, `south_east`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          | `center` |
| objectType             | enum: `airplane`, `apple`, `backpack`, `banana`, `baseball_bat`, `baseball_glove`, `bear`, `bed`, `bench`, `bicycle`, `bird`, `boat`, `book`, `bottle`, `bowl`, `broccoli`, `bus`, `cake`, `car`, `carrot`, `cat`, `cell_phone`, `chair`, `clock`, `couch`, `cow`, `cup`, `dining_table`, `dog`, `donut`, `elephant`, `fire_hydrant`, `fork`, `frisbee`, `giraffe`, `hair_drier`, `handbag`, `horse`, `hot_dog`, `keyboard`, `kite`, `knife`, `laptop`, `microwave`, `motorcycle`, `mouse`, `orange`, `oven`, `parking_meter`, `person`, `pizza`, `potted_plant`, `refrigerator`, `remote`, `sandwich`, `scissors`, `sheep`, `sink`, `skateboard`, `skis`, `snowboard`, `spoon`, `sports_ball`, `stop_sign`, `suitcase`, `surfboard`, `teddy_bear`, `tennis_racket`, `tie`, `toaster`, `toilet`, `toothbrush`, `traffic_light`, `train`, `truck`, `tv`, `umbrella`, `vase`, `wine_glass`, `zebra` | `person` |

#### Usage Example

```
val t = Transformation.intelligentcrop(requiredwidth = 0, requiredheight = 0, paddingpercentage = 0, maintainoriginalaspect = false, aspectratio = "", gravitytowards = IntelligentCrop.Gravitytowards.NONE, preferreddirection = IntelligentCrop.Preferreddirection.CENTER, objecttype = IntelligentCrop.Objecttype.PERSON);
```

</details>

### 9. ObjectCounter

<details>
<summary>1. detect</summary>

#### Usage Example

```
val t = Transformation.objectcounter();
```

</details>

### 10. NumberPlateDetection

<details>
<summary>1. detect</summary>

#### Usage Example

```
val t = Transformation.numberplatedetection();
```

</details>

### 11. CheckObjectSize

<details>
<summary>1. detect</summary>

#### Supported Configuration

| Parameter              | Type    | Default |
| ---------------------- | ------- | ------- |
| objectThresholdPercent | integer | 50      |

#### Usage Example

```
val t = Transformation.checkobjectsize(objectthresholdpercent = 50);
```

</details>

### 12. TextDetectionandRecognition

<details>
<summary>1. extract</summary>

#### Supported Configuration

| Parameter  | Type    | Default |
| ---------- | ------- | ------- |
| detectOnly | boolean | false   |

#### Usage Example

```
val t = Transformation.textdetectionandrecognition(detectonly = false);
```

</details>

### 13. PdfWatermarkRemoval

<details>
<summary>1. remove</summary>

#### Usage Example

```
val t = Transformation.pdfwatermarkremoval();
```

</details>

### 14. ProductTagging

<details>
<summary>1. tag</summary>

#### Usage Example

```
val t = Transformation.producttagging();
```

</details>

### 15. CheckProductVisibility

<details>
<summary>1. detect</summary>

#### Usage Example

```
val t = Transformation.checkproductvisibility();
```

</details>

### 16. RemoveBG

<details>
<summary>1. bg</summary>

#### Usage Example

```
val t = Transformation.removebg();
```

</details>

### 17. Basic

<details>
<summary>1. resize</summary>

#### Supported Configuration

| Parameter  | Type                                                                                                     | Default    |
| ---------- | -------------------------------------------------------------------------------------------------------- | ---------- |
| height     | integer                                                                                                  | 0          |
| width      | integer                                                                                                  | 0          |
| fit        | enum: `cover`, `contain`, `fill`, `inside`, `outside`                                                    | `cover`    |
| background | color                                                                                                    | `000000`   |
| position   | enum: `top`, `bottom`, `left`, `right`, `right_top`, `right_bottom`, `left_top`, `left_bottom`, `center` | `center`   |
| algorithm  | enum: `nearest`, `cubic`, `mitchell`, `lanczos2`, `lanczos3`                                             | `lanczos3` |
| dpr        | float                                                                                                    | 1          |

#### Usage Example

```
val t = Transformation.resize(height = 0, width = 0, fit = Resize.Fit.COVER, background = "000000", position = Resize.Position.CENTER, algorithm = Resize.Algorithm.LANCZOS3, dpr = 1);
```

</details>

<details>
<summary>2. compress</summary>

#### Supported Configuration

| Parameter | Type    | Default |
| --------- | ------- | ------- |
| quality   | integer | 80      |

#### Usage Example

```
val t = Transformation.compress(quality = 80);
```

</details>

<details>
<summary>3. extend</summary>

#### Supported Configuration

| Parameter  | Type                                             | Default    |
| ---------- | ------------------------------------------------ | ---------- |
| top        | integer                                          | 10         |
| left       | integer                                          | 10         |
| bottom     | integer                                          | 10         |
| right      | integer                                          | 10         |
| background | color                                            | `000000`   |
| borderType | enum: `constant`, `replicate`, `reflect`, `wrap` | `constant` |
| dpr        | float                                            | 1          |

#### Usage Example

```
val t = Transformation.extend(top = 10, left = 10, bottom = 10, right = 10, background = "000000", bordertype = Extend.Bordertype.CONSTANT, dpr = 1);
```

</details>

<details>
<summary>4. extract</summary>

#### Supported Configuration

| Parameter | Type    | Default |
| --------- | ------- | ------- |
| top       | integer | 10      |
| left      | integer | 10      |
| height    | integer | 50      |
| width     | integer | 20      |

#### Usage Example

```
val t = Transformation.extract(top = 10, left = 10, height = 50, width = 20);
```

</details>

<details>
<summary>5. trim</summary>

#### Supported Configuration

| Parameter | Type    | Default |
| --------- | ------- | ------- |
| threshold | integer | 10      |

#### Usage Example

```
val t = Transformation.trim(threshold = 10);
```

</details>

<details>
<summary>6. rotate</summary>

#### Supported Configuration

| Parameter  | Type    | Default  |
| ---------- | ------- | -------- |
| angle      | integer | 0        |
| background | color   | `000000` |

#### Usage Example

```
val t = Transformation.rotate(angle = 0, background = "000000");
```

</details>

<details>
<summary>7. flip</summary>

#### Usage Example

```
val t = Transformation.flip();
```

</details>

<details>
<summary>8. flop</summary>

#### Usage Example

```
val t = Transformation.flop();
```

</details>

<details>
<summary>9. sharpen</summary>

#### Supported Configuration

| Parameter | Type  | Default |
| --------- | ----- | ------- |
| sigma     | float | 1.5     |

#### Usage Example

```
val t = Transformation.sharpen(sigma = 1.5);
```

</details>

<details>
<summary>10. median</summary>

#### Supported Configuration

| Parameter | Type    | Default |
| --------- | ------- | ------- |
| size      | integer | 3       |

#### Usage Example

```
val t = Transformation.median(size = 3);
```

</details>

<details>
<summary>11. blur</summary>

#### Supported Configuration

| Parameter | Type  | Default |
| --------- | ----- | ------- |
| sigma     | float | 0.3     |
| dpr       | float | 1       |

#### Usage Example

```
val t = Transformation.blur(sigma = 0.3, dpr = 1);
```

</details>

<details>
<summary>12. flatten</summary>

#### Supported Configuration

| Parameter  | Type  | Default  |
| ---------- | ----- | -------- |
| background | color | `000000` |

#### Usage Example

```
val t = Transformation.flatten(background = "000000");
```

</details>

<details>
<summary>13. negate</summary>

#### Usage Example

```
val t = Transformation.negate();
```

</details>

<details>
<summary>14. normalise</summary>

#### Usage Example

```
val t = Transformation.normalise();
```

</details>

<details>
<summary>15. linear</summary>

#### Supported Configuration

| Parameter | Type    | Default |
| --------- | ------- | ------- |
| a         | integer | 1       |
| b         | integer | 0       |

#### Usage Example

```
val t = Transformation.linear(a = 1, b = 0);
```

</details>

<details>
<summary>16. modulate</summary>

#### Supported Configuration

| Parameter  | Type    | Default |
| ---------- | ------- | ------- |
| brightness | float   | 1       |
| saturation | float   | 1       |
| hue        | integer | 90      |

#### Usage Example

```
val t = Transformation.modulate(brightness = 1, saturation = 1, hue = 90);
```

</details>

<details>
<summary>17. grey</summary>

#### Usage Example

```
val t = Transformation.grey();
```

</details>

<details>
<summary>18. tint</summary>

#### Supported Configuration

| Parameter | Type  | Default  |
| --------- | ----- | -------- |
| color     | color | `000000` |

#### Usage Example

```
val t = Transformation.tint(color = "000000");
```

</details>

<details>
<summary>19. toFormat</summary>

#### Supported Configuration

| Parameter | Type                                        | Default |
| --------- | ------------------------------------------- | ------- |
| format    | enum: `jpeg`, `png`, `webp`, `tiff`, `avif` | `jpeg`  |

#### Usage Example

```
val t = Transformation.toformat(format = Toformat.Format.JPEG);
```

</details>

<details>
<summary>20. density</summary>

#### Supported Configuration

| Parameter | Type    | Default |
| --------- | ------- | ------- |
| density   | integer | 300     |

#### Usage Example

```
val t = Transformation.density(density = 300);
```

</details>

<details>
<summary>21. merge</summary>

#### Supported Configuration

| Parameter      | Type                                                                                                                                                                                                                                                                                          | Default    |
| -------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------- |
| mode           | enum: `overlay`, `underlay`, `wrap`                                                                                                                                                                                                                                                           | `overlay`  |
| image          | file                                                                                                                                                                                                                                                                                          | ``         |
| transformation | custom                                                                                                                                                                                                                                                                                        | ``         |
| background     | color                                                                                                                                                                                                                                                                                         | `00000000` |
| height         | integer                                                                                                                                                                                                                                                                                       | 0          |
| width          | integer                                                                                                                                                                                                                                                                                       | 0          |
| top            | integer                                                                                                                                                                                                                                                                                       | 0          |
| left           | integer                                                                                                                                                                                                                                                                                       | 0          |
| gravity        | enum: `northwest`, `north`, `northeast`, `east`, `center`, `west`, `southwest`, `south`, `southeast`, `custom`                                                                                                                                                                                | `center`   |
| blend          | enum: `over`, `in`, `out`, `atop`, `dest`, `dest-over`, `dest-in`, `dest-out`, `dest-atop`, `xor`, `add`, `saturate`, `multiply`, `screen`, `overlay`, `darken`, `lighten`, `colour-dodge`, `color-dodge`, `colour-burn`, `color-burn`, `hard-light`, `soft-light`, `difference`, `exclusion` | `over`     |
| tile           | boolean                                                                                                                                                                                                                                                                                       | false      |
| listOfBboxes   | bboxList                                                                                                                                                                                                                                                                                      |            |
| listOfPolygons | polygonList                                                                                                                                                                                                                                                                                   |            |

#### Usage Example

```
val t = Transformation.merge(mode = Merge.Mode.OVERLAY, image = "", transformation = "", background = "00000000", height = 0, width = 0, top = 0, left = 0, gravity = Merge.Gravity.CENTER, blend = Merge.Blend.OVER, tile = false, listofbboxes = "", listofpolygons = "");
```

</details>

### 18. SuperResolution

<details>
<summary>1. upscale</summary>

#### Supported Configuration

| Parameter   | Type             | Default |
| ----------- | ---------------- | ------- |
| type        | enum: `2x`, `4x` | `2x`    |
| enhanceFace | boolean          | false   |

#### Usage Example

```
val t = Transformation.superresolution(type = SuperResolution.Type._2X, enhanceface = false);
```

</details>

### 19. WatermarkRemoval

<details>
<summary>1. remove</summary>

#### Supported Configuration

| Parameter  | Type    | Default       |
| ---------- | ------- | ------------- |
| removeText | boolean | false         |
| removeLogo | boolean | false         |
| box1       | string  | `0_0_100_100` |
| box2       | string  | `0_0_0_0`     |
| box3       | string  | `0_0_0_0`     |
| box4       | string  | `0_0_0_0`     |
| box5       | string  | `0_0_0_0`     |

#### Usage Example

```
val t = Transformation.watermarkremoval(removetext = false, removelogo = false, box1 = "0_0_100_100", box2 = "0_0_0_0", box3 = "0_0_0_0", box4 = "0_0_0_0", box5 = "0_0_0_0");
```

</details>

### 20. WatermarkDetection

<details>
<summary>1. detect</summary>

#### Supported Configuration

| Parameter  | Type    | Default |
| ---------- | ------- | ------- |
| detectText | boolean | false   |

#### Usage Example

```
val t = Transformation.watermarkdetection(detecttext = false);
```

</details>
