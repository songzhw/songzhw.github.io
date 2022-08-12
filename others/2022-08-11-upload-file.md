
## uploading files in .... 2013
The uploading request has a strict format that we need to follow, something like this: 

```
Content-type:multipart/formdata, boundary=boundary

--boundary
Content-disposition: form-data; name="name"

mike

--boundary
Content-disposition: form-data; name: "pic", filename: "myPic.jpg"
Content-type: image/jpg

<myPic.jpg>

--boundary--
```

I still remember when I wrote some code to upload a file in 2013, I need to assemble the request one line by one line, and it's kind of painful. Yes, that's the dawn before the OkHttp. 

And when I try to find out how iOS upload a file, I found `URLSession` is still using this clumsy approach: 

```objective-c
   NSString *TWITTERFON_FORM_BOUNDARY = @"12344321";
    NSMutableURLRequest *mulrequest = [NSMutableURLRequest requestWithURL:[NSURL URLWithString:@"http"] cachePolicy:(NSURLRequestReloadIgnoringLocalCacheData) timeoutInterval:10];
    NSString *boundary = [NSString stringWithFormat:@"--%@",TWITTERFON_FORM_BOUNDARY];
    NSString *endBoundary = [NSString stringWithFormat:@"--%@--",TWITTERFON_FORM_BOUNDARY];
    UIImage *image = [UIImage imageNamed:@"pic"];
    NSData *imageData = UIImageJPEGRepresentation(image, 0.3);
    NSMutableString *body = [NSMutableString string];
    NSArray *keys = [param allKeys];
    for (NSString *key in keys) {
        [body appendFormat:@"%@\r\n",boundary];
        [body appendFormat:@"Content-Disposition: form-data; name=\"%@\"\r\n\r\n",key];
        [body appendFormat:@"%@\r\n",[dic objectForKey:key]];
    }
    [body appendFormat:@"%@\r\n",boundary];
    [body appendFormat:@"Content-Disposition:form-data; name=\"pic\"; filename=\"myPic.jpg\"\r\n"];
    [body appendFormat:@"Cotent-Type: image/png"];
    NSMutableData *requestData = [NSMutableData data];
    [requestData appendData:[body dataUsingEncoding:NSUTF8StringEncoding]];
    [requestData appendData:imageData];
    [requestData appendData:[[NSString stringWithFormat:@"\r\n%@",endBoundary] dataUsingEncoding:NSUTF8StringEncoding]];
    
```

In this article, I will try to build a bakcend to receive uploaded files, and also build the mobile side to upload files in a mordern way.

## Backend
Mobile upload files manually, which means there is a Html form in the mobile side. I have to receive files without form, without form field or form field name. 

I will use a thried party library, called "multer". 
multer will encapsulate the upload process into some easy process and simple configuration. 

```yaml
yarn add multer  # help us simplify the uploading receiving process
yarn add app-root-path  # tell me where is the app root path
```

And the multer would declare the storage destination by `multer.diskStorage(config)`, also it will use `upload.any()` to accept any uploaded files. So the backend code is:

```js
var express = require("express");
var app = express(); // the app is using Express to provide the basic router

var bodyParser = require("body-parser");
var appRoot = require("app-root-path");
var multer = require("multer");

const storage = multer.diskStorage({
    destination: appRoot + "/public/images/",
    filename: (req, file, callback) => {
        // file = {"fieldname":"de84lvxrgy59-1","originalname":"de84lvxrgy59-1","encoding":"7bit","mimetype":"application/octet-stream"}
        callback(null, file.originalname); // the file name will be the second arg: `file.originalname`
    }
});
const upload = multer({ storage: storage });


app.post("/wallpapers", upload.any(), function(req, resp) {
    resp.end(`upload ${req.files.length} files done`);
});


const { getIPAddress } = require("../utils/utils");
app.listen(8899, function() {
    console.log(`server started at ${getIPAddress()}:8899...`);
});

```

The multer will receive the uploaded files for us and put the files into the `destination` through the `disStorage(config)` configuration. 


## Android
First we need a Retrofit interface to setup the endpoint config. Note that the argument is a post body, and its type is OkHttp's `MultiplartBody` class.

```kotlin
interface UploadPhotoApi {
    @POST("wallpapers")
    fun uploadWallpapers(@Body body: MultipartBody): Observable<ResponseBody>
}
```




## iOS


## Flutter