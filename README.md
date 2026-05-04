<div align="center">

<img src="app/src/main/res/mipmap-xxxhdpi/ic_launcher_round.png" width="100" alt="VehicleInfoCheck Logo"/>

# VehicleInfoCheck

**An Android application that reads Indian vehicle license plates using computer vision and retrieves real-time vehicle details from the national VAHAN registry.**

[![Platform](https://img.shields.io/badge/Platform-Android-3DDC84?style=flat-square&logo=android&logoColor=white)](https://developer.android.com)
[![Language](https://img.shields.io/badge/Language-Java-ED8B00?style=flat-square&logo=openjdk&logoColor=white)](https://www.java.com)
[![OpenCV](https://img.shields.io/badge/OpenCV-3.4.12-5C3EE8?style=flat-square&logo=opencv&logoColor=white)](https://opencv.org)
[![TensorFlow Lite](https://img.shields.io/badge/TensorFlow_Lite-2.x-FF6F00?style=flat-square&logo=tensorflow&logoColor=white)](https://www.tensorflow.org/lite)
[![Min SDK](https://img.shields.io/badge/Min_SDK-21_(Lollipop)-informational?style=flat-square)](https://developer.android.com/about/versions/lollipop)
[![License](https://img.shields.io/badge/License-GPLv3-blue?style=flat-square)](LICENSE)

<br/>

<img src="https://user-images.githubusercontent.com/81788169/124381452-51e9e480-dce0-11eb-8a7e-be483d1d0fe1.gif" height="480" alt="App Demo"/>

</div>

---

## Overview

VehicleInfoCheck automates the tedious process of manually entering license plate numbers to look up vehicle information. Point your camera at any Indian license plate, and the app does the rest — extracting the plate, segmenting each character, classifying them with a TensorFlow Lite neural network, and launching a pre-filled lookup on the [VAHAN national vehicle registry](https://vahan.nic.in/nrservices/faces/user/login.xhtml).

---

## Features

- **Camera & Gallery Support** — Capture live photos or import from the device gallery
- **Interactive Cropping** — Manually crop the image to isolate the license plate region
- **OpenCV Image Pipeline** — Grayscale conversion, binary thresholding, morphological ops, and contour detection
- **TensorFlow Lite OCR** — On-device 36-class character recognition (A–Z, 0–9) with no internet dependency
- **VAHAN Integration** — Embedded WebView auto-navigates to the vehicle lookup page with the detected plate pre-filled
- **Copy to Clipboard** — One-tap copy of the recognized plate number
- **Offline Inference** — Character recognition runs entirely on-device; no data sent to external servers

---

## System Architecture

```
┌─────────────────────────────────────────────────────────────────────────┐
│                          VehicleInfoCheck App                           │
│                                                                         │
│  ┌─────────────┐                                                        │
│  │ MainActivity│  Splash Screen (2.5s) ──────────────────────────────► │
│  └─────────────┘                                                        │
│                      ┌──────────────────────────────────────────────┐  │
│                       │             ImageActivity                    │  │
│                       │                                              │  │
│   ┌──────────┐        │  ┌─────────────────────────────────────┐    │  │
│   │  Camera  │──────► │  │        Image Input Layer            │    │  │
│   └──────────┘        │  │  Camera API  │  MediaStore (Gallery)│    │  │
│   ┌──────────┐        │  └──────────────┬──────────────────────┘    │  │
│   │  Gallery │──────► │                 │                            │  │
│   └──────────┘        │                 ▼                            │  │
│                       │  ┌─────────────────────────────────────┐    │  │
│                       │  │      Image Cropper (ArthurHub)       │    │  │
│                       │  └──────────────┬──────────────────────┘    │  │
│                       │                 │                            │  │
│                       │                 ▼                            │  │
│                       │  ┌─────────────────────────────────────┐    │  │
│                       │  │       OpenCV Processing Pipeline     │    │  │
│                       │  │                                      │    │  │
│                       │  │  1. extractPlate()                   │    │  │
│                       │  │     └─ Cascade Classifier            │    │  │
│                       │  │        (indian_license_plate.xml)    │    │  │
│                       │  │                                      │    │  │
│                       │  │  2. preProcessing()                  │    │  │
│                       │  │     ├─ Resize  → 333 × 75 px         │    │  │
│                       │  │     ├─ Grayscale conversion          │    │  │
│                       │  │     ├─ Otsu Binary Thresholding      │    │  │
│                       │  │     └─ Erosion + Dilation (morph)    │    │  │
│                       │  │                                      │    │  │
│                       │  │  3. findContour()                    │    │  │
│                       │  │     ├─ Image inversion               │    │  │
│                       │  │     ├─ Contour detection             │    │  │
│                       │  │     └─ Segment chars → 28 × 28 px    │    │  │
│                       │  └──────────────┬──────────────────────┘    │  │
│                       │                 │                            │  │
│                       │                 ▼                            │  │
│                       │  ┌─────────────────────────────────────┐    │  │
│                       │  │     TensorFlow Lite Inference        │    │  │
│                       │  │                                      │    │  │
│                       │  │  Model: BmpCharacterRecognition      │    │  │
│                       │  │  Input:  28 × 28 × 3 (FLOAT32)      │    │  │
│                       │  │  Output: 36 classes (A–Z, 0–9)      │    │  │
│                       │  │  Runs: On-device, no internet        │    │  │
│                       │  │                                      │    │  │
│                       │  │  Per-character argmax → plate string │    │  │
│                       │  └──────────────┬──────────────────────┘    │  │
│                       └─────────────────┼────────────────────────────┘  │
│                                         │                               │
│                                         ▼                               │
│                       ┌──────────────────────────────────────────────┐  │
│                       │              WebActivity                     │  │
│                       │                                              │  │
│                       │  ┌──────────────────────────────────────┐   │  │
│                       │  │          WebScraper (WebView)         │   │  │
│                       │  │  • JS + DOM Storage enabled           │   │  │
│                       │  │  • Pre-fills plate number             │   │  │
│                       │  │  • Copy-to-clipboard button           │   │  │
│                       │  └──────────────┬───────────────────────┘   │  │
│                       └─────────────────┼────────────────────────────┘  │
└─────────────────────────────────────────┼─────────────────────────────┘
                                          │  HTTPS
                                          ▼
                          ┌───────────────────────────┐
                          │   VAHAN National Registry  │
                          │   vahan.nic.in             │
                          │                            │
                          │  Vehicle Model, Fuel Type, │
                          │  Owner, RTO, Insurance...  │
                          └───────────────────────────┘
```

---

## Tech Stack

| Layer | Technology |
|---|---|
| Platform | Android (min SDK 21 / API 30 target) |
| Language | Java 1.8 |
| Image Processing | OpenCV 3.4.12 |
| Machine Learning | TensorFlow Lite (ML Model Binding) |
| Image Cropping | Android-Image-Cropper v2.8 (ArthurHub) |
| Vehicle Lookup | VAHAN Registry via Android WebView |
| Build System | Gradle 4.1.3 |
| UI Framework | AndroidX (AppCompat, ConstraintLayout, Navigation) |
| Fonts | Tenor Sans, Raleway |

---

## Project Structure

```
VehicleInfoCheck/
├── app/
│   └── src/main/
│       ├── java/com/example/vehicleinfocheck/
│       │   ├── MainActivity.java       # Splash screen with 2.5s delay
│       │   ├── ImageActivity.java      # Image capture + full OCR pipeline
│       │   ├── WebActivity.java        # VAHAN website display + clipboard
│       │   ├── WebScraper.java         # Configured WebView wrapper
│       │   └── Element.java            # Website element interaction
│       ├── ml/
│       │   ├── bmp_character_recognition_model.tflite
│       │   └── character_recognition_model.tflite
│       ├── jniLibs/                    # OpenCV native libs (arm, x86, mips)
│       └── res/
│           ├── layout/                 # Activity XML layouts
│           ├── raw/                    # Cascade classifier XML
│           ├── font/                   # Tenor Sans, Raleway
│           └── values/                 # Colors, strings, themes
├── openCVLibrary3412/                  # OpenCV Android library module
├── build.gradle
└── settings.gradle
```

---

## Getting Started

### System Requirements

| Component | Requirement |
|---|---|
| IDE | Android Studio 4.1.0+ |
| JDK | Java 1.8+ |
| RAM | 8 GB recommended (4 GB causes emulator issues) |
| Disk Space | 8 GB+ (IDE + SDK + Emulator) |
| OS | Windows 8/10 (64-bit), macOS 10.14+, or 64-bit Linux |
| Android Device | API Level 21 (Android 5.0 Lollipop) or higher |

### Installation

**1. Clone the repository**

```bash
git clone https://github.com/tanishaapriya/VehicleInfoCheck.git
cd VehicleInfoCheck
```

**2. Open in Android Studio**

- Launch Android Studio
- Go to **File → New → Project from Version Control**
- Select **Git** from the dropdown
- Paste the repository URL and click **Clone**

**3. Configure the OpenCV Library**

- In the Project panel, open `openCVLibrary3412/build.gradle`
- Ensure `compileSdkVersion` and `targetSdkVersion` match your installed Android SDK version
- Update if needed, then go to **File → Sync Project with Gradle Files**

**4. Build & Run**

- Connect an Android device or start an emulator (API 21+)
- Press **Shift + F10** or click the **Run** button
- Grant Camera, Storage, and Internet permissions when prompted

---

## Usage

1. **Launch the app** — The splash screen appears briefly before the main screen loads
2. **Select an image source** — Tap **Camera** to capture a photo, or **Gallery** to pick an existing image
3. **Crop the image** — Drag the crop handles to tightly frame the license plate, then confirm
4. **Wait for processing** — OpenCV segments the plate; TensorFlow Lite classifies each character
5. **Review the result** — The detected plate number is displayed; copy it with the clipboard button
6. **Look up on VAHAN** — The embedded browser opens the VAHAN registry with the plate pre-filled; log in to view vehicle details

---

## How It Works

### Image Processing Pipeline (OpenCV)

| Step | Operation | Detail |
|---|---|---|
| 1 | Plate Extraction | Haar Cascade Classifier (`indian_license_plate.xml`) detects the plate region |
| 2 | Resize | Normalized to **333 × 75 px** |
| 3 | Grayscale | Color image → single-channel intensity |
| 4 | Binarization | Otsu's adaptive thresholding |
| 5 | Morphological Ops | Erosion + dilation to remove noise and separate characters |
| 6 | Contour Detection | Image inversion → contour finding → bounding box per character |
| 7 | Character Crop | Each character cropped to **28 × 28 px**, ordered left-to-right by X position |

### TensorFlow Lite Inference

| Property | Value |
|---|---|
| Model File | `bmp_character_recognition_model.tflite` |
| Input Tensor | `28 × 28 × 3` — `FLOAT32` |
| Output Tensor | `36` classes (A–Z mapped to 0–25, 0–9 mapped to 26–35) |
| Inference | On-device; synchronous; no network required |

---

## Permissions Required

```xml
<uses-permission android:name="android.permission.CAMERA" />
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
<uses-permission android:name="android.permission.INTERNET" />
```

---

## Known Limitations

- The current TensorFlow Lite model produces inaccurate predictions for some license plate characters — improving model accuracy is an active area of development
- The VAHAN website requires manual login; the app cannot automate authentication
- Plate detection accuracy depends on image quality, lighting, and camera angle

---

## Credits

| Resource | Source |
|---|---|
| Deep Learning Model | [SarthakV7 / AI-based Indian License Plate Detection](https://github.com/SarthakV7/AI-based-indian-license-plate-detection) |
| Image Cropping Library | [ArthurHub / Android-Image-Cropper](https://github.com/ArthurHub/Android-Image-Cropper) |
| WebScraper Reference | [Udayraj123 / VehicleInfoOCR](https://github.com/Udayraj123/VehicleInfoOCR) |

---

## License

This project is licensed under the **GNU General Public License v3.0**.
See the [LICENSE](LICENSE) file for the full license text.

---

<div align="center">

Made with care by **[tanishaapriya](https://github.com/tanishaapriya)**

</div>
