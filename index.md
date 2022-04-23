## CameraX Project Tutorial
*It should be noted that features not pertaining to the camera will not be covered. This mainly pertains to any xml elements.* <br>
*Additionally all imports that were used in my project are included in the code blocks, you may end up not needing some depending on how much is implemented.*
### Initial Setup

Create a new android project using an empty activity. Make sure "Minimum SDK" is set to 21 or higher. 
* CameraX is not supported below API Level 21 <br>

In the newly created build.gradle file for the Module add the following inside of dependencies{} block:
```
  def camerax_version = "1.1.0-beta01"
  implementation "androidx.camera:camera-core:${camerax_version}"
  implementation "androidx.camera:camera-camera2:${camerax_version}"
  implementation "androidx.camera:camera-lifecycle:${camerax_version}"
  implementation "androidx.camera:camera-video:${camerax_version}"

  implementation "androidx.camera:camera-view:${camerax_version}"
  implementation "androidx.camera:camera-extensions:${camerax_version}"
  ```
  then add the following at end of android{} block which essentially allows findByViewId to be replaced with viewbinding:
```
buildFeatures {
   viewBinding true
}
  ```
  also in settings.gradle make the following are inside both repositories{} block:
```
   google()
   mavenCentral()
  ```
Permissions must also be established and granted in order for the app to access the camera, microphone, and ability to save captures to the gallery. Therefore the following lines are added to AndroidManifest.xml before Application tag:
  ```
 <uses-feature android:name="android.hardware.camera.any" />

    <uses-permission android:name="android.permission.CAMERA" />
    <uses-permission android:name="android.permission.RECORD_AUDIO" />
    <uses-permission
        android:name="android.permission.WRITE_EXTERNAL_STORAGE"
        android:maxSdkVersion="28" />
  ```
### Establishing the activity_main.xml and MainActivity.kt
If you want to have the same layout, I recommend that you download [my res folder](https://github.com/romanbrancato/CameraXProject/tree/master/app/src/main/res).
Otherwise, the following will need to be added to your activity_main.xml in addition to adding listeners for any of the buttons:

* Note that androidx.camera.view.PreviewView is the view to which the camera preview will be streamed to
```
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <androidx.camera.view.PreviewView
        android:id="@+id/viewFinder"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        tools:layout_editor_absoluteX="0dp"
        tools:layout_editor_absoluteY="0dp" />
     
     //Capture and Record Buttons go here. 
     //It is suggested to additionally add a seekBar for future zooming purposes, a button to toggle flash, and another for toggling cameras.
</androidx.constraintlayout.widget.ConstraintLayout>
```
In order to set up MainAcitivity, the following code has been provided by the [Official CameraX CodeLabs](https://developer.android.com/codelabs/camerax-getting-started#0).<br> 
* This will serve as the foundation for the most basic of camera functionalities. 

Tweak the package name to fit your project name in addition to the button listeners in the onCreate{} block
```
package com.android.example.PROJECTNAMEGOESHERE
import com.example.PROJECTNAMEGOESHERE.databinding.ActivityMainBinding

import android.Manifest
import android.annotation.SuppressLint
import android.content.ContentValues
import android.content.pm.PackageManager
import android.media.MediaPlayer
import android.os.Build
import android.os.Bundle
import android.os.SystemClock
import android.provider.MediaStore
import android.util.Log
import android.view.MotionEvent
import android.view.View
import android.widget.RadioButton
import android.widget.SeekBar
import android.widget.Toast
import androidx.appcompat.app.AppCompatActivity
import androidx.camera.core.*
import androidx.camera.lifecycle.ProcessCameraProvider
import androidx.camera.video.*
import androidx.camera.video.VideoCapture
import androidx.core.app.ActivityCompat
import androidx.core.content.ContextCompat
import androidx.core.content.PermissionChecker
import androidx.lifecycle.LifecycleOwner
import java.text.DecimalFormat
import java.text.SimpleDateFormat
import java.util.*
import java.util.concurrent.ExecutorService
import java.util.concurrent.Executors


class MainActivity : AppCompatActivity() {
   private lateinit var viewBinding: ActivityMainBinding

   private var imageCapture: ImageCapture? = null

   private var videoCapture: VideoCapture<Recorder>? = null
   private var recording: Recording? = null

   private lateinit var cameraExecutor: ExecutorService
  
   private var lensFacing: Int = CameraSelector.LENS_FACING_BACK
  
   private var camera: Camera? = null

   override fun onCreate(savedInstanceState: Bundle?) {
       super.onCreate(savedInstanceState)
       viewBinding = ActivityMainBinding.inflate(layoutInflater)
       setContentView(viewBinding.root)

       // Request camera permissions
       if (allPermissionsGranted()) {
           startCamera()
       } else {
           ActivityCompat.requestPermissions(
               this, REQUIRED_PERMISSIONS, REQUEST_CODE_PERMISSIONS)
       }

       // Set up the listeners for take photo and video capture buttons
       viewBinding.IMAGEBUTTONIDGOESHERE.setOnClickListener { takePhoto() }
       viewBinding.VIDEOBUTTONIDGOESHERE.setOnClickListener { captureVideo() }

       cameraExecutor = Executors.newSingleThreadExecutor()
   }
  
   override fun onRequestPermissionsResult(
        requestCode: Int,
        permissions: Array<String>,
        grantResults: IntArray
    ) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults)
        if (requestCode == REQUEST_CODE_PERMISSIONS) {
            if (allPermissionsGranted()) {
                startCamera()
            } else {
                Toast.makeText(
                    this,
                    "Permissions Denied",
                    Toast.LENGTH_SHORT
                ).show()
                finish()
            }
        }
    }

   private fun takePhoto() {}

   private fun captureVideo() {}

   private fun startCamera() {}

   private fun allPermissionsGranted() = REQUIRED_PERMISSIONS.all {
       ContextCompat.checkSelfPermission(
           baseContext, it) == PackageManager.PERMISSION_GRANTED
   }

   override fun onDestroy() {
       super.onDestroy()
       cameraExecutor.shutdown()
   }

   companion object {
       private const val TAG = "CameraXApp"
       private const val FILENAME_FORMAT = "yyyy-MM-dd-HH-mm-ss-SSS"
       private const val REQUEST_CODE_PERMISSIONS = 10
       private val REQUIRED_PERMISSIONS =
           mutableListOf (
               Manifest.permission.CAMERA,
               Manifest.permission.RECORD_AUDIO
           ).apply {
               if (Build.VERSION.SDK_INT <= Build.VERSION_CODES.P) {
                   add(Manifest.permission.WRITE_EXTERNAL_STORAGE)
               }
           }.toTypedArray()
   }
}
```
 
### Implementing the Basic Functionality
The following code contains the implementation of the Preview, ImageCapture, and VideoCapture use cases.

The Preview class allows you to see what you will capture, imagine when you first open the default android or ios camera

All use cases are built then binded to the lifecycle of the cameras

* Its important to note that this is where the camera object is intialized. The camera object is how you will be able to control the current camera

The following code must be inserted inside the startCamera(){} block
 ```
        val cameraProviderFuture = ProcessCameraProvider.getInstance(this)

        cameraProviderFuture.addListener({
            // Used to bind the lifecycle of cameras to the lifecycle owner
            val cameraProvider: ProcessCameraProvider = cameraProviderFuture.get()

            val cameraSelector = CameraSelector.Builder()
                .requireLensFacing(lensFacing)
                .build()

            // Preview Use Case
            val preview = Preview.Builder()
                .build()
                .also {
                    it.setSurfaceProvider(viewBinding.viewFinder.surfaceProvider)
                }

            //Image Capture Use Case
            imageCapture = ImageCapture.Builder()
                .build()

            //Video Capture Use Case
            val recorder = Recorder.Builder()
                .setQualitySelector(QualitySelector.from(Quality.HIGHEST))
                .build()
            videoCapture = VideoCapture.withOutput(recorder)

            // Unbind use cases before rebinding
            cameraProvider.unbindAll()

            try {

                // Bind use cases to camera
                camera = cameraProvider.bindToLifecycle(
                    this as LifecycleOwner,
                    cameraSelector,
                    preview,
                    imageCapture,
                    videoCapture
                )

            } catch (exc: Exception) {
                Log.e(TAG, "Use case binding failed", exc)
            }
        }, ContextCompat.getMainExecutor(this))
 ```
 *Next Code blocks are referenced from the [Official CameraX CodeLabs](https://developer.android.com/codelabs/camerax-getting-started#0)*
 
 Functionality for ImageCapture and VideoCapture must still be implemented by doing the following
 
 When the imageCapture button is pressed, an image is saved to the defined location, name, and timestamp in the created MediaStore entry
 
 This is to be added into the takePhoto(){} block:
```
// Get a stable reference of the modifiable image capture use case
   val imageCapture = imageCapture ?: return

   // Create time stamped name and MediaStore entry.
   val name = SimpleDateFormat(FILENAME_FORMAT, Locale.US)
              .format(System.currentTimeMillis())
   val contentValues = ContentValues().apply {
       put(MediaStore.MediaColumns.DISPLAY_NAME, name)
       put(MediaStore.MediaColumns.MIME_TYPE, "image/jpeg")
       if(Build.VERSION.SDK_INT > Build.VERSION_CODES.P) {
           put(MediaStore.Images.Media.RELATIVE_PATH, "Pictures/CameraX-Image")
       }
   }

   // Create output options object which contains file + metadata
   val outputOptions = ImageCapture.OutputFileOptions
           .Builder(contentResolver,
                    MediaStore.Images.Media.EXTERNAL_CONTENT_URI,
                    contentValues)
           .build()

   // Set up image capture listener, which is triggered after photo has
   // been taken
   imageCapture.takePicture(
       outputOptions,
       ContextCompat.getMainExecutor(this),
       object : ImageCapture.OnImageSavedCallback {
           override fun onError(exc: ImageCaptureException) {
               Log.e(TAG, "Photo capture failed: ${exc.message}", exc)
           }

           override fun
               onImageSaved(output: ImageCapture.OutputFileResults){
               val msg = "Photo capture succeeded: ${output.savedUri}"
               Toast.makeText(baseContext, msg, Toast.LENGTH_SHORT).show()
               Log.d(TAG, msg)
           }
       }
   )
 // Get a stable reference of the modifiable image capture use case
        val imageCapture = imageCapture ?: return

        // Create time stamped name and MediaStore entry.
        val name = SimpleDateFormat(FILENAME_FORMAT, Locale.US)
            .format(System.currentTimeMillis())
        val contentValues = ContentValues().apply {
            put(MediaStore.MediaColumns.DISPLAY_NAME, name)
            put(MediaStore.MediaColumns.MIME_TYPE, "image/jpeg")
            if (Build.VERSION.SDK_INT > Build.VERSION_CODES.P) {
                put(MediaStore.Images.Media.RELATIVE_PATH, "Pictures/CameraXProject-Image")
            }
        }

        // Create output options object which contains file + metadata
        val outputOptions = ImageCapture.OutputFileOptions
            .Builder(
                contentResolver,
                MediaStore.Images.Media.EXTERNAL_CONTENT_URI,
                contentValues
            )
            .build()

        // Set up image capture listener, which is triggered after photo has been taken
        imageCapture.takePicture(outputOptions,
            ContextCompat.getMainExecutor(this),
            object : ImageCapture.OnImageSavedCallback {
                override fun onError(exc: ImageCaptureException) {
                    Log.e(TAG, "Photo capture failed: ${exc.message}", exc)
                }

                override fun onImageSaved(output: ImageCapture.OutputFileResults) {
                    val msg = "Photo capture succeeded"
                    Toast.makeText(baseContext, msg, Toast.LENGTH_SHORT).show()
                    Log.d(TAG, msg)
                }
            }
        )
 ```
 This defines what happens when the videoCapture button is pressed.
 
 This is to be added into the captureVideo(){} block:
 ```
        val videoCapture = this.videoCapture ?: return


        viewBinding.captureButton.isEnabled = false

        val curRecording = recording
        if (curRecording != null) {
            // Stop the current recording session.
            curRecording.stop()
            recording = null
            return
        }

        // create and start a new recording session
        val name = SimpleDateFormat(FILENAME_FORMAT, Locale.US)
            .format(System.currentTimeMillis())
        val contentValues = ContentValues().apply {
            put(MediaStore.MediaColumns.DISPLAY_NAME, name)
            put(MediaStore.MediaColumns.MIME_TYPE, "video/mp4")
            if (Build.VERSION.SDK_INT > Build.VERSION_CODES.P) {
                put(MediaStore.Video.Media.RELATIVE_PATH, "Movies/CameraXProject-Video")
            }
        }

        val mediaStoreOutputOptions = MediaStoreOutputOptions
            .Builder(contentResolver, MediaStore.Video.Media.EXTERNAL_CONTENT_URI)
            .setContentValues(contentValues)
            .build()
        recording = videoCapture.output
            .prepareRecording(this, mediaStoreOutputOptions)
            .apply {
                if (PermissionChecker.checkSelfPermission(
                        this@MainActivity,
                        Manifest.permission.RECORD_AUDIO
                    ) ==
                    PermissionChecker.PERMISSION_GRANTED
                ) {
                    withAudioEnabled()
                }
            }
            .start(ContextCompat.getMainExecutor(this)) { recordEvent ->
                when (recordEvent) {
                    is VideoRecordEvent.Start -> {
                        viewBinding.captureButton.apply {
                            isEnabled = true
                        }
                    }
                    is VideoRecordEvent.Finalize -> {

                        if (!recordEvent.hasError()) {
                            val msg = "Video capture succeeded"
                            Toast.makeText(baseContext, msg, Toast.LENGTH_SHORT).show()
                            Log.d(TAG, msg)
                        } else {
                            recording?.close()
                            recording = null
                            Log.e(
                                TAG, "Video capture ends with error: " +
                                        "${recordEvent.error}"
                            )
                        }
                        viewBinding.captureButton.apply {
                            isEnabled = true
                        }
                    }
                }
            }
  ``` 
### Adding Control Over The Camera
As of now, you should have an app that can only take photos and videos and saves them to the gallery. In order to get features such as zooming, tap to focus, and     flash toggling cameraControl must be obtained from the camera object initialized in startCamera(){} which can be done by:
 ```
  camera!!.cameraControl. ...
 ``` 
 Getting various information about the camera can be obtained through:
 ```
  camera!!.cameraInfo. ...
 ``` 
  
 **Implementing Zoom through seekBar**<br>
 * zoomBar is the id of the seekBar used. Displays the zoom factor through a toast
 Intially Referenced from [this article](https://proandroiddev.com/android-camerax-tap-to-focus-pinch-to-zoom-zoom-slider-eb88f3aa6fc6):
 ```
 val df = DecimalFormat("#.##")
        //Zoom using slider
        viewBinding.zoomBar.setOnSeekBarChangeListener(object : SeekBar.OnSeekBarChangeListener {
            override fun onProgressChanged(seekBar: SeekBar?, progress: Int, fromUser: Boolean) {
                camera!!.cameraControl.setLinearZoom(progress / 100.toFloat())
            }

            override fun onStartTrackingTouch(seekBar: SeekBar?) {}

            override fun onStopTrackingTouch(seekBar: SeekBar?) {
                val msg =
                    df.format(camera!!.cameraInfo.zoomState.value!!.zoomRatio).toString() + "x"
                Toast.makeText(baseContext, msg, Toast.LENGTH_SHORT).show()
            }
        })
  ``` 
 
 **Implementing Tap to Focus** <br>
 *Intially Referenced from [this article](https://proandroiddev.com/android-camerax-tap-to-focus-pinch-to-zoom-zoom-slider-eb88f3aa6fc6)*
 This implementation uses the MeteringPointFactory to apply focusing to specific coordinates on the Preview View: 
 ```
  viewBinding.viewFinder.setOnTouchListener(View.OnTouchListener { _: View, motionEvent: MotionEvent ->
            when (motionEvent.action) {
                MotionEvent.ACTION_DOWN -> return@OnTouchListener true
                MotionEvent.ACTION_UP -> {
                    // Get the MeteringPointFactory from PreviewView
                    val factory = viewBinding.viewFinder.meteringPointFactory

                    // Create a MeteringPoint from the tap coordinates
                    val point = factory.createPoint(motionEvent.x, motionEvent.y)

                    // Create a MeteringAction from the MeteringPoint, you can configure it to specify the metering mode
                    val action = FocusMeteringAction.Builder(point).build()

                    // Trigger the focus and metering
                    camera!!.cameraControl.cancelFocusAndMetering()
                    camera!!.cameraControl.startFocusAndMetering(action)

                    return@OnTouchListener true
                }
                else -> return@OnTouchListener false
            }
        })
 ``` 
 **Toggling Flash** <br>
 The flash is enabled and disabled by calling enableTorch(boolean) on the cameraControl instance:
 ```
    //Flash Toggle Button Listener
        viewBinding.flashButton.setOnCheckedChangeListener { _, isChecked ->
            if (camera!!.cameraInfo.hasFlashUnit()) {
                if (isChecked) {
                    camera!!.cameraControl.enableTorch(true)
                } else {
                    camera!!.cameraControl.enableTorch(false)
                }
            } else {
                if (isChecked) {
                    val msg = "Device Does Not Have Flash Unit"
                    Toast.makeText(baseContext, msg, Toast.LENGTH_SHORT).show()
                }
            }
        }
 ``` 
 **Flipping Between Cameras** <br>
 Can simply be done by altering the CameraSelector then calling the startCamera() function again:
 ```
   //Flip Between Back and Front Camera
    private fun flipCamera() {
        if (lensFacing == CameraSelector.LENS_FACING_FRONT) {
            lensFacing = CameraSelector.LENS_FACING_BACK
        } else if (lensFacing == CameraSelector.LENS_FACING_BACK) {
            lensFacing = CameraSelector.LENS_FACING_FRONT
        }
        startCamera()
    }
 ```
