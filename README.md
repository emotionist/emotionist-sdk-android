# Emotionist SDK for Android

[![Platform](https://img.shields.io/badge/platform-android-orange.svg)](https://github.com/emotionist/emotionist-sdk-android)
[![Languages](https://img.shields.io/badge/language-java-orange.svg)](https://github.com/emotionist/emotionist-sdk-android)
[![Commercial License](https://img.shields.io/badge/license-Commercial-brightgreen.svg)](https://github.com/emotionist/emotionist-sdk-android/blob/master/LICENSE.md)

## Table of contents

  1. [About Emotionist SDK](#about-emotionist-sdk)
  1. [Install Emotionist SDK](#install-emotionist-sdk)
  1. [Making your first recognition](#making-your-first-recognition)

<br />

## About Emotionist SDK

Through our **Emotionist SDK** for Android, you can efficiently integrate real-time recognition of facial expression, heart response and emotions into your mobile app. This and other pages in the Getting Started provide the Emotionist SDK’s structure and installation steps, then goes through the preliminary steps of implementing the Emotionist SDK in your own project.

### Requirements

The minimum requirements for Emotionist SDK for Android are:

- Android 4.2.0 (API level 17) or later
- Java 8 or later
- Gradle 4.0.0 or later

```groovy
// build.gradle(app)
android {
    defaultConfig {
        minSdkVersion 17
    }

    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }
}
```

### Key functions

|Solutions|Description|
|---|---|
|FaceRppg| We measures heart response non-contactly using a camera. The facial blood volume pulse is estimated from facial color variations and head movements caused by heartbeat using remote Photoplethysmography and Ballistocardiography. Our solution provide heart rate, heart rate variaility reflecting autonomic nervous system activity and respiration. |

### Try the sample app

Our sample app has the core features of the Emotionist SDK. Download the app from our [GitHub repository](https://github.com/emotionist/emotionist-sample-android) to get an idea of what you can build with the actual SDK and start building in your project.

> Note: The fastest way to see our Emotionist SDK in action is to build your app on top of our sample app. Make sure to change the application ID of the sample app to your own.

<br />

## Install Emotionist SDK

This page provides a step-by-step guide that demonstrates how to build and configure an in-app face and bio-analysis using the Emotionist SDK. 

### Step 1: Install the Emotionist SDK

First, select a solution you want in the Emotionist SDK. Then, copy the Emotionist SDK `.aar` file to the `app/libs` folder in your app. Then, add the dependency to your module `build.gradle` file:

```groovy
dependencies {
    implementation fileTree(dir: 'libs', include: ['*.jar', '*.aar'])

    // Mediapipe deps
    implementation 'com.google.flogger:flogger:latest.release'
    implementation 'com.google.flogger:flogger-system-backend:latest.release'
    implementation 'com.google.code.findbugs:jsr305:latest.release'
    implementation 'com.google.guava:guava:27.0.1-android'
    implementation 'com.google.protobuf:protobuf-javalite:3.19.1'
    
    // CameraX core library
    def camerax_version = "1.0.0-beta10"
    implementation "androidx.camera:camera-core:$camerax_version"
    implementation "androidx.camera:camera-camera2:$camerax_version"
    implementation "androidx.camera:camera-lifecycle:$camerax_version"
    
    // AutoValue
    def auto_value_version = "1.8.1"
    implementation "com.google.auto.value:auto-value-annotations:$auto_value_version"
    annotationProcessor "com.google.auto.value:auto-value:$auto_value_version"
}
```

> Note: Make sure the above code block isn't added to your module `bundle.gradle` file.


### Step 2: Grant system permissions to the Emotionist SDK

The Emotionist SDK requires system permissions. These permissions allow the Emotionist SDK to use a camera and read from and write on a user device’s storage. To grant system permissions, add the following lines to your `AndroidManifest.xml` file.

```manifest
<!-- For using the camera -->
<uses-permission android:name="android.permission.CAMERA" />
<uses-feature android:name="android.hardware.camera" />
<uses-feature android:name="android.hardware.camera.autofocus" />
<!-- For logging solution events -->
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
<!-- For Emotionist -->
<uses-feature android:glEsVersion="0x00020000" android:required="true" />
```

The `CAMERA` permission is classified as `dangerous` and require users to grant them explicitly when an app is run for the first time on devices running Android 6.0 or higher.

For more information about requesting app permissions, see  Android’s Request App Permissions [guide](https://developer.android.com/training/permissions/requesting.html).

<br />

## Making your first recognition

The Emotionist SDK simplifies vision features into an effortless and straightforward process. To recognize your facial expression, heart response and emotion, do the following steps:

This page provides a step-by-step guide that demonstrates how to build and configure an in-app face and bio-analysis using Emotionist SDK. License key can be received by requesting by the email: **support@emotionist.ai**.

### Step 1: Initialize the Emotionist SDK

Initialization binds the Emotionist SDK to Android’s context, thereby allowing it to use a camera in your mobile. To the `init()` method, pass the **App ID** of your application to initialize the SDK and the **InitResultHandler** to received callback for validation of the App ID.

```java
FaceRppg faceRppg = new FaceRppg(this,
                        FaceRppgOptions.builder()
                            .setStaticImageMode(false)
                            .setRunOnGpu(true)
                            .setMinDetectionConfidence(0.5f)
                            .build());
// Connects Emotionist FaceRppg solution to the user-defined FaceRppgResultImageView.
faceRppg.setResultListener(
        faceRppgResult -> {
          // TODO
        });
faceRppg.setErrorListener(
        (message, e) -> Log.e(TAG, "Emotionist FaceRppg error:" + message));

// Initializes Emotionist FaceRPPG solution instance in the streaming mode.
faceRppg.init(APP_ID, new FaceRppg.InitResultHandler() {
    @Override
    public void onInitSucceed() {
        // TODO
    }
    
    @Override
    public void onInitFailed() {
        // TODO
    }
}
```

> Note: The `init()` method must be called once across your Android app. It is recommended to initialize the Emotionist SDK in the `onCreate()` method of the Application instance.

### Step 2: Initialize camera nad surfaceveiw

If you want to access the camera and draw the measurements on the view easily, you can use the Emotionist solutioncore. Include the **container** to bind the Emotionist Fragment in your layout `.xml` file.

```xml
<FrameLayout
        android:id="@+id/preview_display_layout"
        android:layout_width="fill_parent"
        android:layout_height="fill_parent" />
```

> Note: FrameLayout is just one of examples. You can change to other layout type to purpose your app.

Bind the emotionist to display the image taken with the camera on the screen. FaceRppgResultGlRenderer automatically display the image to fit the size of your custom layout.

```java
// Initializes camera.
CameraInput cameraInput = new CameraInput(activity);
cameraInput.setNewFrameListener(textureFrame -> faceRppg.send(textureFrame));

// Initializes a new Gl surface view with a user-defined
// FaceRppgResultGlRenderer.
SolutionGlSurfaceView<FaceRppgResult> glSurfaceView = 
    new SolutionGlSurfaceView<>(
        getApplicationContext(), faceRppg.getGlContext(), faceRppg.getGlMajorVersion());
glSurfaceView.setSolutionResultRenderer(new FaceRppgResultGlRenderer());
glSurfaceView.setRenderInputImage(true);
faceRppg.setResultListener(
        faceRppgResult -> {
          // TODO
          
          // Render view
          glSurfaceView.setRenderData(faceRppgResult);
          glSurfaceView.requestRender();
        });
```

> Note: The code for initlaization of camera and view must be called once across your Android app. It is recommended to call the 'onInitSucceed()' method of the callback of 'init()' method.

### Step 3: Start the Emotionist SDK

Start the Emotionist SDK to recognize your facial expression, heart response and emotion. Please refer to **[sample app](https://github.com/emotionist/emotionist-sample-android)**.

```java
// The runnable to start camera after the gl surface view is attached.
glSurfaceView.post(this::startCamera);

// Updates the preview layout.
FrameLayout frameLayout = findViewById(R.id.preview_display_layout);
frameLayout.removeAllViewsInLayout();
frameLayout.addView(glSurfaceView);
glSurfaceView.setVisibility(View.VISIBLE);
frameLayout.requestLayout();

// Define a function to start camera.
private void startCamera() {
    cameraInput.start(activity,
        faceRppg.getGlContext(),
        CameraInput.CameraFacing.FRONT,
        glSurfaceView.getWidth(),
        glSurfaceView.getHeight());
}
```

> Note: The code to start Emotionist SDK must be called once across your Android app. It is recommended to call the 'onInitSucceed()' method of the callback of 'init()' method.

### Step 4: Stop the Emotionist SDK

When your app is not use the camera or destroyed, stop the Emotionist SDK.

```java
if (cameraInput != null) {
    cameraInput.setNewFrameListener(null);
    cameraInput.close();
}
if (glSurfaceView != null) {
    glSurfaceView.setVisibility(View.GONE);
}
if (faceRppg != null) {
    faceRppg.close();
}
```
