# SamsungCamera-Research

Research for SamsungCamera app and its related static libraries.

## `SamsungCamera.apk`

### App + libraries samples

- [GitHub](https://github.com/illusion0001/SamsungCamera-Research/releases/tag/files)
- [Internet Archive](https://archive.org/details/mod-samsung-camera-files)

Modified versions:
- [S9/Note 9 + S10/Note 10](https://xdaforums.com/t/zero-camera-mod-bitrates-jpg-92-100-pro-video-mode-4k60-external-sd.4055503)
  - File: `ZeroCameraMod_S10_N10_10.0.0.72_v2.zip`
- [S7/Note 7 + S8/Note 8](https://xdaforums.com/t/magisk-exynos-samsung-4k60-camera-mod-uhd-60-fps-hevc-6400-iso-more.3883816)
  - File: `s7cammod-7.6.86-3.0.4.zip`

Thanks to [Cvolton](https://github.com/Cvolton) and zeroprobe.

### Video bitrate

#### Video bitrate 1

Has constants for min/max.
Easily found by searching for specific video profile.

```smali
    # can be found in files:
    # smali/com/sec/android/app/camera/engine/MediaRecorderProfile$1.smali
    # smali/com/sec/android/app/camera/engine/MediaRecorderProfile$2.smali
    # smali/com/sec/android/app/camera/engine/MediaRecorderProfile$3.smali

    invoke-direct {v1, v3, v2, v5, v4}, Lcom/sec/android/app/camera/engine/MediaRecorderProfile$VideoBitrate;-><init>(IIILcom/sec/android/app/camera/engine/MediaRecorderProfile$1;)V
    invoke-virtual {p0, v0, v1}, Lcom/sec/android/app/camera/engine/MediaRecorderProfile$1;->put(ILjava/lang/Object;)V
    sget-object v0, Lcom/sec/android/app/camera/interfaces/Resolution;->RESOLUTION_3840X2160_60FPS:Lcom/sec/android/app/camera/interfaces/Resolution;
    invoke-virtual {v0}, Lcom/sec/android/app/camera/interfaces/Resolution;->getId()I
    move-result v0
    new-instance v1, Lcom/sec/android/app/camera/engine/MediaRecorderProfile$VideoBitrate;
    const v3, 0x44aa200 # max bitrate 72000000
    const v5, 0x280de80 # min bitrate 42000000
```

#### Video bitrate 2

Has constants for min/avg/max.
Easily found by searching for specific video profile.

```smali
    invoke-direct {v1, v3, v2, v3, v4}, Lcom/sec/android/app/camera/engine/MediaRecorderProfile$VideoBitrate;-><init>(IIILcom/sec/android/app/camera/engine/MediaRecorderProfile$1;)V
    invoke-virtual {p0, v0, v1}, Lcom/sec/android/app/camera/engine/MediaRecorderProfile$1;->put(ILjava/lang/Object;)V
    sget-object v0, Lcom/sec/android/app/camera/interfaces/Resolution;->RESOLUTION_3840X2160:Lcom/sec/android/app/camera/interfaces/Resolution;
    invoke-virtual {v0}, Lcom/sec/android/app/camera/interfaces/Resolution;->getId()I
    move-result v0
    new-instance v1, Lcom/sec/android/app/camera/engine/MediaRecorderProfile$VideoBitrate;
    const v2, 0x1ab3f00 # min bitrate 28000000 ?
    const v3, 0x2dc6c00 # avg bitrate 48000000
    const v5, 0x337f980 # max bitrate 54000000
```

### Max long exposure

```smali
    # file:
    # smali/com/sec/android/app/camera/engine/request/TakePictureRequest.smali
    invoke-direct {p0}, Lcom/sec/android/app/camera/engine/request/TakePictureRequest;->isLongExposureShot()Z
    move-result p0
    if-eqz p0, :cond_0
    const p0, 0x88b8 # 3.5 seconds max, 0xf4240 = 10 seconds
```

### Slow motion bitrate

```smali
    # file:
    # smali_classes2/com/sec/android/app/camera/shootingmode/feature/SlowMotionFeature$1.smali
    const p1, 0x895440 # max bitrate 9000000
    invoke-static {p1}, Ljava/lang/Integer;->valueOf(I)Ljava/lang/Integer;
    move-result-object p1
    const-string p2, "recordingBitrate"
    invoke-virtual {p0, p2, p1}, Lcom/sec/android/app/camera/shootingmode/feature/SlowMotionFeature$1;->put(Ljava/lang/Object;Ljava/lang/Object;)Ljava/lang/Object;
```

### Force full 100% JPEG quality

#### Step 1

Add the following lines to:

```
smali/com/sec/android/app/camera/engine/request/TakeVideoSnapshotRequest.smali
smali/com/sec/android/app/camera/engine/request/TakeAgifBurstPictureRequest.smali
smali/com/sec/android/app/camera/engine/request/TakeBurstPictureRequest.smali
smali/com/sec/android/app/camera/engine/request/StartStitchingCaptureRequest.smali
smali/com/sec/android/app/camera/engine/request/StartSingleTakePictureRequest.smali
smali/com/sec/android/app/camera/engine/request/TakePictureRequest.smali
smali/com/sec/android/app/camera/engine/request/TakeProcessingPictureRequest.smali
```

```smali
.method execute()V

# ...

    # registers may differ
    sget-object v3, Lcom/samsung/android/camera/core2/MakerPublicKey;->REQUEST_JPEG_QUALITY:Landroid/hardware/camera2/CaptureRequest$Key;
    const v4, 0x64
    invoke-static {v4}, Ljava/lang/Byte;->valueOf(B)Ljava/lang/Byte;
    move-result-object v4
    # it wants byte, int16 or int32 will crash
    invoke-virtual {v2, v3, v4}, Lcom/sec/android/app/camera/engine/request/MakerPublicSettings;->set(Landroid/hardware/camera2/CaptureRequest$Key;Ljava/lang/Object;)V
    # Usually before it logs the request and gets the location data is where you would want to set these.
    # const-string v2, "TakePictureRequest"
# ...
    return-void
.end method
```

In newer builds, this key is simply named `w`.

```smali
.method execute()V

# ...

    sget-object v3, Lcom/samsung/android/camera/core2/MakerPublicKey;->w:Landroid/hardware/camera2/CaptureRequest$Key;
    const v4, 0x64
    invoke-static {v4}, Ljava/lang/Byte;->valueOf(B)Ljava/lang/Byte;
    move-result-object v4
    invoke-virtual {v2, v3, v4}, Lcom/sec/android/app/camera/engine/request/MakerPublicSettings;->set(Landroid/hardware/camera2/CaptureRequest$Key;Ljava/lang/Object;)V
    # Usually before it logs the request and gets the location data is where you would want to set these.
    # const-string v2, "TakePictureRequest"
# ...
    return-void
.end method
```

#### Step 2

**Note:** Some devices/firmwares may not have this function compiled. if simply setting the `REQUEST_JPEG_QUALITY` key is enough to override quality in Auto and Pro modes, this static library patches won't be needed.

Return 0 on `android::ExynosCameraConfigurations::checkJpegSqueezerEnable` from `libexynoscamera3.so` (Required for modes that isn't auto)

```asm
;; Function `android::ExynosCameraConfigurations::checkJpegSqueezerEnable`
; original
--> 0x0018a562 09b1 cbz r1,0x0018a568
    0x0018a564 0020 movs r0,#0x0
    0x0018a566 7047 bx lr
; patched
--> 0x0018a562 0020 movs r0,#0x0
--> 0x0018a564 7047 bx lr
;;
```

#### Step 3

Force 100% quality factor + Use `QuramJpegWrapper` in `libnode-jni.so` (Required for auto mode)

```asm
; original
-->     0016e098 a6 22 41 29     ldp        w6,w8,[x21, #0x8]
        0016e09c 80 42 40 f9     ldr        x0,[x20, #0x80]
        0016e0a0 c1 06 40 f9     ldr        x1,[x22, #0x8]
        0016e0a4 62 7a 40 b9     ldr        w2,[x19, #0x78]
        0016e0a8 63 82 40 b9     ldr        w3,[x19, #0x80]
        0016e0ac a4 16 42 29     ldp        w4,w5,[x21, #0x10]
        0016e0b0 e8 0b 00 b9     str        w8,[sp, #0x8]=>local_138
        0016e0b4 e8 63 01 91     add        x8,sp,#0x58
        0016e0b8 e7 63 00 91     add        x7,sp,#0x18
        0016e0bc e8 03 00 f9     str        x8,[sp]=>local_140
        0016e0c0 64 69 ff 97     bl         JpegSqueezerWrapper::encodeJPEGToBuffer

; patched
-->     0016e098 88 0c 80 52     mov        w8,#0x64
        0016e09c 80 42 40 f9     ldr        x0,[x20, #0x80]
        0016e0a0 c1 06 40 f9     ldr        x1,[x22, #0x8]
        0016e0a4 62 7a 40 b9     ldr        w2,[x19, #0x78]
        0016e0a8 63 82 40 b9     ldr        w3,[x19, #0x80]
        0016e0ac a4 16 42 29     ldp        w4,w5,[x21, #0x10]
        0016e0b0 e8 0b 00 b9     str        w8,[sp, #0x8]=>local_138
        0016e0b4 e8 63 01 91     add        x8,sp,#0x58
        0016e0b8 e7 63 00 91     add        x7,sp,#0x18
        0016e0bc e8 03 00 f9     str        x8,[sp]=>local_140
        0016e0c0 64 69 ff 97     bl         JpegSqueezerWrapper::encodeJPEGToBuffer
```

### Idle Inactivity Timeout

```smali
.field private static final INACTIVITY_TIMEOUT:I = 0x1d4c0
```

### Allow all recordings to external storage

You must have a fast enough SD Card to save 4K 60FPS videos.

```smali
.method public static getCamcorderExternalStorageAvailableFeature(ILcom/sec/android/app/camera/interfaces/Resolution;)Z
    .registers 3

    const/4 p0, 0x1

    return p0
.end method
```

### Allow all possible ISO range in all cameras

This also fixes an index mysteriously disappearing when it is too high (3200 -> 6400. this 3200 entry will not be usable if not patched)

```smali
.method public getIsoScrollerRange()Landroid/util/Range;
    .registers 4
    .annotation system Ldalvik/annotation/Signature;
        value = {
            "()",
            "Landroid/util/Range<",
            "Ljava/lang/Integer;",
            ">;"
        }
    .end annotation

    new-instance v1, Landroid/util/Range;

    const v0, 0x0

    invoke-static {v0}, Ljava/lang/Integer;->valueOf(I)Ljava/lang/Integer;

    move-result-object v0

    # array length - 1
    const p0, 0xf

    invoke-static {p0}, Ljava/lang/Integer;->valueOf(I)Ljava/lang/Integer;

    move-result-object p0

    invoke-direct {v1, v0, p0}, Landroid/util/Range;-><init>(Ljava/lang/Comparable;Ljava/lang/Comparable;)V

    return-object v1
.end method
```

### Allow all possible shutter speed range during Pro Video Mode

```smali
.method public static getMaxVideoShutterSpeed(I)I
    .registers 1

    const/16 p0, 0x25

    return p0
.end method
```

### Allow saving Raw + JPEG to SD card

#### V13 and older

```smali
.method getPictureSavingStorage(Lcom/sec/android/app/camera/interfaces/InternalEngine$PictureSavingType;)I
    .registers 4

    .line 1
    iget-object v0, p0, Lcom/sec/android/app/camera/engine/PictureProcessor;->mCameraContext:Lcom/sec/android/app/camera/interfaces/CameraContext;

    invoke-interface {v0}, Lcom/sec/android/app/camera/interfaces/CameraContext;->isRecording()Z

    move-result v0

    if-eqz v0, :cond_13

    .line 2
    iget-object p0, p0, Lcom/sec/android/app/camera/engine/PictureProcessor;->mEngine:Lcom/sec/android/app/camera/engine/CommonEngine;

    invoke-virtual {p0}, Lcom/sec/android/app/camera/engine/CommonEngine;->getRecordingManager()Lcom/sec/android/app/camera/interfaces/RecordingManager;

    move-result-object p0

    invoke-interface {p0}, Lcom/sec/android/app/camera/interfaces/RecordingManager;->getRecordingStorage()I

    move-result p0

    return p0

    .line 3
    :cond_13
    sget-object v0, Lcom/sec/android/app/camera/engine/PictureProcessor$1;->$SwitchMap$com$sec$android$app$camera$interfaces$InternalEngine$PictureSavingType:[I

    invoke-virtual {p1}, Ljava/lang/Enum;->ordinal()I

    move-result v1

    aget v0, v0, v1

    const/4 v1, 0x0

    packed-switch v0, :pswitch_data_56

    .line 4
    new-instance p0, Ljava/lang/IllegalArgumentException;

    new-instance v0, Ljava/lang/StringBuilder;

    invoke-direct {v0}, Ljava/lang/StringBuilder;-><init>()V

    const-string v1, "not supported picture saving type : "

    invoke-virtual {v0, v1}, Ljava/lang/StringBuilder;->append(Ljava/lang/String;)Ljava/lang/StringBuilder;

    invoke-virtual {p1}, Ljava/lang/Enum;->name()Ljava/lang/String;

    move-result-object p1

    invoke-virtual {v0, p1}, Ljava/lang/StringBuilder;->append(Ljava/lang/String;)Ljava/lang/StringBuilder;

    invoke-virtual {v0}, Ljava/lang/StringBuilder;->toString()Ljava/lang/String;

    move-result-object p1

    invoke-direct {p0, p1}, Ljava/lang/IllegalArgumentException;-><init>(Ljava/lang/String;)V

    throw p0

    .line 5
    :pswitch_3a  #0x4
    iget-object p0, p0, Lcom/sec/android/app/camera/engine/PictureProcessor;->mCameraSettings:Lcom/sec/android/app/camera/interfaces/CameraSettings;

    sget-object p1, Lcom/sec/android/app/camera/interfaces/CameraSettings$Key;->STORAGE:Lcom/sec/android/app/camera/interfaces/CameraSettings$Key;

    invoke-interface {p0, p1}, Lcom/sec/android/app/camera/interfaces/CameraSettings;->get(Lcom/sec/android/app/camera/interfaces/CameraSettings$Key;)I

    move-result p0

    return p0
    :pswitch_43  #0x2, 0x3, 0x5, 0x6, 0x7
    # fallthrough
    # return v1

    .line 6
    :pswitch_44  #0x1
    iget-object p1, p0, Lcom/sec/android/app/camera/engine/PictureProcessor;->mEngine:Lcom/sec/android/app/camera/engine/CommonEngine;

    invoke-virtual {p1}, Lcom/sec/android/app/camera/engine/CommonEngine;->isRawSingleCaptureEnabled()Z

    move-result p1

    # fallthrough
    # if-eqz p1, :cond_4d

    # goto :goto_55

    :cond_4d
    iget-object p0, p0, Lcom/sec/android/app/camera/engine/PictureProcessor;->mCameraSettings:Lcom/sec/android/app/camera/interfaces/CameraSettings;

    sget-object p1, Lcom/sec/android/app/camera/interfaces/CameraSettings$Key;->STORAGE:Lcom/sec/android/app/camera/interfaces/CameraSettings$Key;

    invoke-interface {p0, p1}, Lcom/sec/android/app/camera/interfaces/CameraSettings;->get(Lcom/sec/android/app/camera/interfaces/CameraSettings$Key;)I

    move-result v1

    :goto_55
    return v1

    :pswitch_data_56
    .packed-switch 0x1
        :pswitch_44  #00000001
        :pswitch_43  #00000002
        :pswitch_43  #00000003
        :pswitch_3a  #00000004
        :pswitch_43  #00000005
        :pswitch_43  #00000006
        :pswitch_43  #00000007
    .end packed-switch
.end method
```

#### V14

```smali
.method public getPictureSavingStorage(Lcom/sec/android/app/camera/interfaces/InternalEngine$PictureSavingType;)I
    .registers 3

    iget-object v0, p0, Lcom/sec/android/app/camera/engine/PictureProcessor;->mCameraContext:Lcom/sec/android/app/camera/interfaces/CameraContext;

    invoke-interface {v0}, Lcom/sec/android/app/camera/interfaces/CameraContext;->isRecording()Z

    move-result v0

    if-eqz v0, :cond_13

    iget-object p0, p0, Lcom/sec/android/app/camera/engine/PictureProcessor;->mEngine:Lcom/sec/android/app/camera/engine/CommonEngine;

    invoke-virtual {p0}, Lcom/sec/android/app/camera/engine/CommonEngine;->getRecordingManager()Lcom/sec/android/app/camera/interfaces/RecordingManager;

    move-result-object p0

    invoke-interface {p0}, Lcom/sec/android/app/camera/interfaces/RecordingManager;->getRecordingStorage()I

    move-result p0

    return p0

    :cond_13
    sget-object v0, Lcom/sec/android/app/camera/engine/PictureProcessor$1;->$SwitchMap$com$sec$android$app$camera$interfaces$InternalEngine$PictureSavingType:[I

    invoke-virtual {p1}, Ljava/lang/Enum;->ordinal()I

    move-result p1

    aget p1, v0, p1

    const/4 v0, 0x1

    if-eq p1, v0, :cond_2b

    const/4 v0, 0x4

    if-eq p1, v0, :cond_22

    goto :goto_3c

    :cond_22
    iget-object p0, p0, Lcom/sec/android/app/camera/engine/PictureProcessor;->mCameraSettings:Lcom/sec/android/app/camera/interfaces/CameraSettings;

    sget-object p1, Lcom/sec/android/app/camera/interfaces/CameraSettings$Key;->STORAGE:Lcom/sec/android/app/camera/interfaces/CameraSettings$Key;

    invoke-interface {p0, p1}, Lcom/sec/android/app/camera/interfaces/CameraSettings;->get(Lcom/sec/android/app/camera/interfaces/CameraSettings$Key;)I

    move-result p0

    return p0

    :cond_2b
    iget-object p1, p0, Lcom/sec/android/app/camera/engine/PictureProcessor;->mEngine:Lcom/sec/android/app/camera/engine/CommonEngine;

    invoke-virtual {p1}, Lcom/sec/android/app/camera/engine/CommonEngine;->isRawSingleCaptureEnabled()Z

    move-result p1

    # dont return if != 0
    # if-nez p1, :cond_3c

    iget-object p0, p0, Lcom/sec/android/app/camera/engine/PictureProcessor;->mCameraSettings:Lcom/sec/android/app/camera/interfaces/CameraSettings;

    sget-object p1, Lcom/sec/android/app/camera/interfaces/CameraSettings$Key;->STORAGE:Lcom/sec/android/app/camera/interfaces/CameraSettings$Key;

    invoke-interface {p0, p1}, Lcom/sec/android/app/camera/interfaces/CameraSettings;->get(Lcom/sec/android/app/camera/interfaces/CameraSettings$Key;)I

    move-result p0

    return p0

    :cond_3c
    :goto_3c
    const/4 p0, 0x0

    return p0
.end method
```

### No arbitrary time limit in recording modes

```smali
.class Lcom/sec/android/app/camera/engine/recordingsession/FileInfo;

.method setMaxDurationInMs(I)V
    .registers 3

    const v0, 0x493e0 # 300 seconds # 2160p60 time limit on `starlte`+`crownlte`

    if-eq p1, v0, :cond_b

    const v0, 0x927c0 # 600 seconds # 1080p60 time limit on `starlte`+`crownlte`

    if-eq p1, v0, :cond_b

    goto :goto_e

    :cond_b
    const p1, -0x1

    :goto_e
    iput p1, p0, Lcom/sec/android/app/camera/engine/recordingsession/FileInfo;->mMaxDuration:I

    return-void
.end method
```

## `libexynoscamera3.so`

### Edge Denoise and Sharpness

Change `0x5` in `movs` to desired value, uint8 only. Found in `/vendor/lib/libexynoscamera3.so`.
if no symbols, search for string:
- `android::ExynosCameraMetadataConverter::translateEdgeControlData`
  - ```
    [CAM_ID(%d)][%s]-DEBUG(%s[%d]):ANDROID_EDGE_STRENGTH(%d)
    [CAM_ID(%d)][%s]-DEBUG(%s[%d]):ANDROID_EDGE_MODE(%d)
    ```
  - ```asm
    0x000f56e6 0520 movs r0,#0x5 ; change this
    0x000f56e8 84f8c008 strb.w r0,[r4,#0x8c0]
    ```

- `android::ExynosCameraMetadataConverter::translateNoiseControlData`
  - ```
    [CAM_ID(%d)][%s]-DEBUG(%s[%d]):ANDROID_NOISE_REDUCTION_STRENGTH(%d)
    [CAM_ID(%d)][%s]-DEBUG(%s[%d]):ANDROID_NOISE_REDUCTION_MODE(%d)
    ```
  - ```asm
    0x000f60e4 0520 movs r0,#0x5 ; change this
    0x000f60e6 84f86009 strb.w r0,[r4,#0x960]
    ```
