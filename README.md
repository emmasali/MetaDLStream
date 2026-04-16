# 🕶️ MetaDLStream — Live Object Detection with Meta Ray-Ban Glasses

An Android app that combines **live camera streaming** from Meta Ray-Ban Gen 1 smart glasses with **real-time object detection** using TensorFlow Lite (EfficientDet-Lite0).

> Built on top of the official [Meta Wearables DAT Android sample](https://github.com/facebook/meta-wearables-dat-android).

---

## 📱 What it does

- Connects to Meta Ray-Ban glasses via Bluetooth
- Streams live camera feed from the glasses to your Android screen
- Runs **EfficientDet-Lite0** on every 10th frame to detect objects
- Displays detection results (label + confidence) overlaid on the live stream
- Captures and shares photos from the glasses

---

## 🧠 How it works

```
Ray-Ban glasses → BLE → Meta DAT SDK → VideoFrame (I420) → YUV→Bitmap → TFLite → Overlay
```

- Every 10th frame is analyzed by EfficientDet-Lite0 (configurable via `DETECTION_INTERVAL_FRAMES`)
- Results are displayed as a transparent overlay at the top of the stream
- Score threshold: 30% confidence minimum

---

## ⚙️ Requirements

| Component | Version |
|---|---|
| Android Studio | Arctic Fox (2021.3.1) or newer |
| JDK | 11 or newer |
| Android SDK | 31+ (Android 12.0+) |
| Target device | Android 10+ with Bluetooth |
| Glasses | Meta Ray-Ban Gen 1 or Gen 2 |
| Meta View app | Installed and glasses paired |

---

## 🗺️ Full Setup Guide

### Step 1 — Meta Developer Center

1. Go to [wearables.developer.meta.com](https://wearables.developer.meta.com)
2. Log in with your Meta account and create an **Organization**
3. Create a new **Project** (e.g. `MetaDLStream`)
4. Go to **App configuration** → **Android** → **+ Add app details**:
   - **Package name**: `com.meta.wearable.dat.externalsampleapps.cameraaccess`
   - **App signature**: see Step 3 below
5. Go to **Permissions** → **+ Request permission** → add **Camera** with a rationale
6. Go to **Distribute** → **+ New version** → **Create version** (1.0.0)
7. Go to **Release channels** → **Edit** Alpha → **+ Invite users** → add your Meta email
8. Accept the invite at [wearables.meta.com/invites](https://wearables.meta.com/invites)

---

### Step 2 — GitHub Personal Access Token

The Meta DAT SDK is distributed via GitHub Packages.

1. Go to [github.com](https://github.com) → **Settings** → **Developer settings**
2. Click **Personal access tokens** → **Tokens (classic)**
3. Click **Generate new token (classic)**
4. Name it `MetaDAT`, expiration 90 days, check only ✅ **read:packages**
5. Click **Generate token** and **copy it immediately**

---

### Step 3 — Get the App Signature

This project uses a custom keystore (`sample.keystore`). Run this in PowerShell:

```powershell
# Add keytool to PATH
$env:PATH += ";C:\Program Files\Android\Android Studio\jbr\bin"

# Get SHA256 fingerprint
keytool -list -v -keystore app\sample.keystore -alias sample -storepass sample -keypass sample
```

Convert the SHA256 to base64url format:

```powershell
[Convert]::ToBase64String([byte[]](0x56,0x7C,0x39,0xB4,...)).Replace('+','-').Replace('/','_').TrimEnd('=')
```

> For this repo, the pre-computed signature is:
> ```
> Vnw5tMbqPlKg1kL4_po7CC05UjBt82T8JqLZYk5V924
> ```

Paste this in **App signature** in the Meta Developer Center.

---

### Step 4 — Clone and Configure

```bash
git clone https://github.com/emmasali/MetaDLStream.git
cd MetaDLStream
```

Create `local.properties` in the project root:

```
sdk.dir=C\:\\Users\\YourName\\AppData\\Local\\Android\\Sdk
github_token=YOUR_GITHUB_TOKEN_HERE
```

---

### Step 5 — Add Meta Credentials

Open `app/src/main/AndroidManifest.xml` and add your credentials from the Meta Developer Center under **App configuration → Android integration**:

```xml
<meta-data
    android:name="com.meta.wearable.mwdat.APPLICATION_ID"
    android:value="YOUR_APPLICATION_ID" />
<meta-data
    android:name="com.meta.wearable.mwdat.CLIENT_TOKEN"
    android:value="YOUR_CLIENT_TOKEN" />
```

---

### Step 6 — Add the TFLite Model

Download **EfficientDet-Lite0** from [TensorFlow Hub](https://tfhub.dev/tensorflow/lite-model/efficientdet/lite0/detection/metadata/1) and place it in:

```
app/src/main/assets/efficientdet_lite0.tflite
```

---

### Step 7 — Build and Run

1. Open the project in **Android Studio**
2. Click **File → Sync Project with Gradle Files**
3. Connect your Android device via USB with **Developer Mode** and **USB Debugging** enabled
4. Click **▶ Run** → select your device

---

### Step 8 — Pair the Glasses

1. Make sure **Meta View** is installed and glasses are paired
2. Launch the app and accept camera permission
3. The live stream starts automatically with detection overlay

---

## 🔧 Troubleshooting

### "Application package signature is not matching"
- Re-extract SHA256 from `app/sample.keystore` (see Step 3)
- Update in Meta Developer Center
- **Clear Meta View cache**: Settings → Apps → Meta View → Clear Cache → Force Stop
- Create a new version in Distribute, assign to Alpha, re-run

### "Token not configured" / Registration fails
- Verify `APPLICATION_ID` and `CLIENT_TOKEN` in `AndroidManifest.xml`
- Make sure your email is **Active** in the Alpha release channel

### Gradle sync fails with 401 Unauthorized
- Check `local.properties` — only ONE `github_token=` line
- Generate a new token with `read:packages` scope

### No objects detected
- Lower `setScoreThreshold` in `StreamViewModel.kt` (e.g. from `0.3f` to `0.1f`)
- Lower `DETECTION_INTERVAL_FRAMES` to analyze more frames

---

## 📁 Project Structure

```
MetaDLStream/
├── app/
│   ├── src/main/
│   │   ├── assets/
│   │   │   └── efficientdet_lite0.tflite   ← TFLite model
│   │   ├── java/.../cameraaccess/
│   │   │   ├── stream/
│   │   │   │   ├── StreamViewModel.kt      ← Stream + TFLite detection
│   │   │   │   └── YuvToBitmapConverter.kt ← YUV→Bitmap conversion
│   │   │   └── ui/
│   │   │       └── StreamScreen.kt         ← UI with detection overlay
│   │   └── AndroidManifest.xml
│   ├── sample.keystore
│   └── build.gradle.kts
├── local.properties                        ← NOT committed
└── README.md
```

---

## 🔑 Credentials Summary

| Item | Where to find |
|---|---|
| APPLICATION_ID | Meta Developer Center → App configuration → Android integration |
| CLIENT_TOKEN | Meta Developer Center → App configuration → Android integration |
| App Signature | Extracted from `sample.keystore` (base64url SHA256) |
| GitHub Token | github.com → Settings → Developer settings → Personal access tokens |

---

## 🗺️ Future Work

- [ ] Bounding boxes overlay on detected objects
- [ ] Text-to-speech readout of detected objects
- [ ] Custom TFLite model fine-tuned for specific use cases
- [ ] GPU delegate for faster inference
- [ ] Lower detection interval for real-time performance

---

## 👩‍💻 Author

**Imane** — Built with ❤️ in Montréal

---

## 📄 License

Based on the [Meta Wearables DAT Android sample](https://github.com/facebook/meta-wearables-dat-android).
See [LICENSE](https://github.com/facebook/meta-wearables-dat-android/blob/main/LICENSE) for details.
