# SamsungCamera-Research

Research for SamsungCamera app and its related static libraries.

**TODO: Make revanced patch for this.**

## `SamsungCamera.apk`

Found by diff'ing this [apk](https://forum.xda-developers.com/t/zero-camera-mod-bitrates-jpg-92-100-pro-video-mode-4k60-external-sd.4055515/) against stock.

Thanks to [Cvolton](https://github.com/Cvolton) and zeroprobe.

The apk itself contains the following:
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

## Max long exposure

```smali
    # file:
    # smali/com/sec/android/app/camera/engine/request/TakePictureRequest.smali
    invoke-direct {p0}, Lcom/sec/android/app/camera/engine/request/TakePictureRequest;->isLongExposureShot()Z
    move-result p0
    if-eqz p0, :cond_0
    const p0, 0x88b8 # 3.5 seconds max, 0xf4240 = 10 seconds
```

## Slow motion bitrate

```smali
    # file:
    # smali_classes2/com/sec/android/app/camera/shootingmode/feature/SlowMotionFeature$1.smali
    const p1, 0x895440 # max bitrate 9000000
    invoke-static {p1}, Ljava/lang/Integer;->valueOf(I)Ljava/lang/Integer;
    move-result-object p1
    const-string p2, "recordingBitrate"
    invoke-virtual {p0, p2, p1}, Lcom/sec/android/app/camera/shootingmode/feature/SlowMotionFeature$1;->put(Ljava/lang/Object;Ljava/lang/Object;)Ljava/lang/Object;
```

## JPEG quality

### Step 1

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
    # registers may differ
    sget-object v4, Lcom/samsung/android/camera/core2/MakerPublicKey;->REQUEST_JPEG_QUALITY:Landroid/hardware/camera2/CaptureRequest$Key;
    const v4, 0x64
    invoke-static {v4}, Ljava/lang/Byte;->valueOf(B)Ljava/lang/Byte;
    move-result-object v4
    # it wants byte, int16 or int32 will crash
    invoke-virtual {v2, v3, v4}, Lcom/sec/android/app/camera/engine/request/MakerPublicSettings;->set(Landroid/hardware/camera2/CaptureRequest$Key;Ljava/lang/Object;)V
```

### Step 2

Return 0 on `android::ExynosCameraConfigurations::checkJpegSqueezerEnable` from `libexynoscamera3.so` (Required for modes that isn't auto)

```asm
        ; android::ExynosCameraConfigurations::checkJpegSqueezerEnable configureStreams:00167476(c)
        ; to
        00189906 09 b1           cbz        r1,LAB_0018990c
        00189908 00 20           movs       r0,#0x0
        0018990a 70 47           bx         lr
        ; to
        00189906 00 20           movs       r0,#0x0
        00189908 70 47           bx         lr
```

### Step 3

Force 100% quality factor + Use `QuramJpegWrapper` in `libnode-jni.so` (Required for auto mode)

```asm
; never use JpegSqueezerWrapper::encodeJPEGToBuffer
; original
        0016a5c4 98 42 40 f9     ldr        x24,[x20, #0x80]
; to
        0016a5c4 18 00 80 d2     mov        x24,#0x0

; always use QuramJpegWrapper::encodeJPEGToBuffer
; original
        0016a6f8 29 01 00 34     cbz        w9,LAB_0016a71c
; to
        0016a6f8 1f 20 03 d5     nop

; force 100% quality factor
; original
        0016a718 d6 62 ff 97     bl         <EXTERNAL>::__android_log_print
        0016a72c a4 1e 41 29     ldp        w4,w7,[x21, #0x8]
; to
        0016a718 a4 1e 41 29     ldp        w4,w7,[x21, #0x8]
        0016a72c 87 0c 80 d2     mov        x7,#0x64
```

## Idle Inactivity Timeout

```smali
.field private static final INACTIVITY_TIMEOUT:I = 0x1d4c0
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
