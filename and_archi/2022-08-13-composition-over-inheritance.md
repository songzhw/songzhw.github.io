## Requirement
We have a requirement to ask users to take some pictures and upload them to our server.
The whole flow is :
```
Take Photo --> review the photo --> (if happy) upload it
                                --> (if unsatisifed) retake the picture
```

So this flow can be divided into 4 activities: `TakePhotoActivity`,  `ReviewActivity`, `RetakePhotoActivity`, and `UploadActivity`.

p.s. The "taking photo" activity are quite different from the "retaking photo" actiivty, that's why we have two separate two activities rather than just one activity. However, this two activities do have something in common : `taking photo`. At the same time, we do need an Activity to host the logic, because we also need to check the camera permission first before we take any pictures. And this checking permission code lies in the Activity. 

In short, it seems we have to add logic in the Activity, but could we reuse the taking photo logic in both activities?


## Inheritance
The impulse is make RetakePhotoActivity extend TakePhotoActivity. So the taking photo logic could exist in the TakePhotoActivity. And this could be reused in its child activity, RetakePhotoActivity.

However, I feel uncomfortable to do so. Actually I've seen this kind of code many many times before. For example, I saw some Activity hierarchy such as : `BaseAcitivity - NetworkActivity - BaseOrderActiivty - PlaceOrderActivity - OrderDetailActivity`. 
The shortcoming of this approach are: 

1). The PlaceOrderActivity has its own layout xml, and the OrderDetailActivity has its own layout xml as well. Which means the OrderDetailActivity#onCreate() will load several super class' xml as well when you call `super.onCreate()`. 

2). Another point is it really is error-prone. Once you change any parent class's method, you may inflect all it's children or grandchildren classes. 
I have come across this kind of bug many times, most of times it just one change of parent class, making the child class behavir different. 

I want to avoid this coupling, which means I can not use inheritance. I would make the Activity independent from each other, they can be siblings, but they are not parent-child.



## Composition
Unlike Ruby and Dart, Kotlin has no `mixin`. Fotunately for us, Kotlin do have the interface which can have some default implementation. 
So All I have to do is just pull all the logic to a new interface.

```kotlin
import androidx.camera.core.ImageCapture

class TakePhotoActivity : AppCompatActivity(R.layout.actv_cfo_take_photo), ITakePhoto {
    override lateinit var capturer: ImageCapture

    override fun onCreate(savedInstanceState: Bundle?) {
      ...
      checkPermissionAndSetupCamera(this, previewView1)
      ivTakePhoto1.setClickListener { capture.takePicture(); ... }
    }
}

class ReTakePhotoActivity : AppCompatActivity(R.layout.actv_cfo_take_photo), ITakePhoto {
    override lateinit var capturer: ImageCapture

      override fun onCreate(savedInstanceState: Bundle?) {
        ...
        checkPermissionAndSetupCamera(this, previewView2)
        ivTakePhoto2.setClickListener { capture.takePicture(); ... }
      }
}

interface ITakePhoto {
    var capturer: ImageCapture //the use case for taking photos

    fun checkPermissionAndSetupCamera(actv: AppCompatActivity, pv: PreviewView) {
        val arLauncher =
            actv.registerForActivityResult(ActivityResultContracts.RequestPermission()) {
                actv.setupCamera(pv) {
                    capturer = ImageCapture.Builder()
                        .setJpegQuality(40) // value is among [1, 100]
                        .build()
                    capturer
                }
            }
        arLauncher.launch(Manifest.permission.CAMERA)
    }

    fun getSaveOption(actv: AppCompatActivity, name: String): ImageCapture.OutputFileOptions {
        val file = actv.filesDir.resolve(name)
        return ImageCapture.OutputFileOptions.Builder(file).build()
        // photo are saved at:  uri = /data/user/0/ca.six.demo.advanced2021/files/hi-11-23-01.jpg
    }
}
```

1). By making an interface, I seperate the logic, and any Activity can add this interface as a component. 

2). the `capturer` is a field in the interface. Note that the filed in the interface has no backing field, and you have to implement it in the implementation class of this interface. 

3). the `capture` is implemented later in a callback function, so I declared `override lateinit var capturer: ImageCapture` in the Activities. This is tricky, but also very elegant.

4). I'm using AndroidX's `Activity Result` API to request CAMERA permssion. 



## Conclusion

Composition, in many cases, are better than inheritance. If you are trying to do so, try the interface with default method, which might help you.