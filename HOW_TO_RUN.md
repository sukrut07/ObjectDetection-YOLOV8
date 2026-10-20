# How to Run YOLOv8 Object Detection (Terminal Guide)

This guide provides step-by-step instructions with copyable terminal commands to set up the environment, prepare the ONNX model, and run image, webcam, and video object detection.

---

## Table of Contents
1. [Prerequisites](#1-prerequisites)
2. [Clone the Repository](#2-clone-the-repository)
3. [Set Up Virtual Environment](#3-set-up-virtual-environment)
4. [Install Dependencies](#4-install-dependencies)
5. [Prepare the YOLOv8 ONNX Model](#5-prepare-the-yolov8-onnx-model)
6. [Run Object Detection](#6-run-object-detection)
   - [A. Image Inference](#a-image-inference)
   - [B. Webcam Real-Time Inference](#b-webcam-real-time-inference)
   - [C. Video Inference (YouTube or Local File)](#c-video-inference-youtube-or-local-file)
7. [Troubleshooting & Tips](#7-troubleshooting--tips)

---

## 1. Prerequisites

Make sure you have:
- **Python 3.8 – 3.11** installed (`python3 --version` or `python --version`)
- **Git** installed (`git --version`)
- A working webcam (optional, required only for `webcam_object_detection.py`)

---

## 2. Clone the Repository

Open your terminal and clone the repository:

```bash
git clone https://github.com/sukrut07/ObjectDetection-YOLOV8.git
```

Navigate into the project directory:

```bash
cd ObjectDetection-YOLOV8
```

---

## 3. Set Up Virtual Environment

It is recommended to use an isolated Python virtual environment.

### On macOS / Linux:

```bash
python3 -m venv .venv
```

Activate the environment:

```bash
source .venv/bin/activate
```

### On Windows (PowerShell):

```powershell
python -m venv .venv
.venv\Scripts\Activate.ps1
```

*(If execution scripts are disabled on PowerShell, run `Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser` first)*

### On Windows (Command Prompt - CMD):

```cmd
python -m venv .venv
.venv\Scripts\activate.bat
```

---

## 4. Install Dependencies

Upgrade `pip` and install the required packages:

```bash
pip install --upgrade pip
```

Install from `requirements.txt`:

```bash
pip install -r requirements.txt
```

> **Note on ONNX Runtime:**
> - On **macOS (Apple Silicon / Intel)** and standard CPU systems, `onnxruntime` is installed automatically.
> - On **Linux / Windows with an NVIDIA GPU and CUDA configured**, `onnxruntime-gpu` will be used automatically.

---

## 5. Prepare the YOLOv8 ONNX Model

The detection scripts look for the model file at:
```text
models/yolov8m.onnx
```

If you do not already have `models/yolov8m.onnx`, export it from Ultralytics using the copyable commands below:

### Option 1: Export using the YOLO CLI

1. Install Ultralytics:
```bash
pip install ultralytics
```

2. Export the `yolov8m.pt` model to ONNX format (input size: 480x640):
```bash
yolo export model=yolov8m.pt format=onnx imgsz=480,640
```

3. Ensure the `models/` directory exists:
```bash
mkdir -p models
```

4. Move the exported model into the `models/` folder:
```bash
mv yolov8m.onnx models/
```
*(On Windows CMD/PowerShell, run `move yolov8m.onnx models\`)*

### Option 2: Export using a Python One-Liner

```bash
python -c "from ultralytics import YOLO; YOLO('yolov8m.pt').export(format='onnx', imgsz=[480, 640])" && mkdir -p models && mv yolov8m.onnx models/
```

### Option 3: Use Google Colab
Alternatively, you can convert and download the model using the Colab notebook:
[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/drive/1-yZg6hFg27uCPSycRCRtyezHhq_VAHxQ?usp=sharing)

---

## 6. Run Object Detection

Ensure your virtual environment is active (`source .venv/bin/activate`). Always run scripts from the project root directory.

### A. Image Inference

Performs object detection on a sample image loaded from a URL, displays the result in a window, and saves the annotated image to `doc/img/detected_objects.jpg`.

Run command:
```bash
python image_object_detection.py
```

- **How to Exit:** Press **any key** while the image window is active to close it.
- **Custom Image:** You can edit `img_url` in `image_object_detection.py` to point to any image URL or replace it with `cv2.imread("your_image.jpg")` to test a local image.

---

### B. Webcam Real-Time Inference

Streams live video from your primary webcam (`camera 0`), runs YOLOv8 detection on each frame in real-time, and renders bounding boxes with class labels and confidence scores.

Run command:
```bash
python webcam_object_detection.py
```

- **How to Exit:** Press **`q`** on your keyboard while focused on the video window.
- **Switch Camera Index:** If you have multiple webcams (or an external USB webcam), change `cap = cv2.VideoCapture(0)` to `cv2.VideoCapture(1)` inside `webcam_object_detection.py`.
- **macOS Permissions:** If running on macOS, ensure your terminal application (e.g., Terminal, iTerm2, VS Code) has camera permission granted under **System Settings > Privacy & Security > Camera**.

---

### C. Video Inference (YouTube or Local File)

Streams a sample video from YouTube (at 720p) and runs real-time object detection.

Run command:
```bash
python video_object_detection.py
```

- **How to Exit:** Press **`q`** on your keyboard while focused on the video window.
- **Using a Local Video File:**
  Open `video_object_detection.py`, comment out the YouTube stream lines, and use OpenCV's native video reader:
  ```python
  cap = cv2.VideoCapture("path/to/your/video.mp4")
  ```

---

## 7. Troubleshooting & Tips

### 1. `ModuleNotFoundError: No module named 'yolov8'`
Make sure your terminal's current working directory is the repository root:
```bash
pwd
```
The output should end with `ObjectDetection-YOLOV8` (or the folder where you cloned the repo). Run the scripts from this root directory.

### 2. `FileNotFoundError: models/yolov8m.onnx`
Verify that `yolov8m.onnx` exists in the `models/` directory:
```bash
ls -lh models/yolov8m.onnx
```
If missing, refer back to [Step 5](#5-prepare-the-yolov8-onnx-model) to export or place it.

### 3. OpenCV Window Does Not Open / SSH Headless Sessions
OpenCV `imshow()` requires a graphical desktop display environment. If you are connected over an SSH session without X11 forwarding or running on a headless server, `cv2.imshow` will fail. You can save the output directly using `cv2.imwrite("output.jpg", combined_img)`.

### 4. Adjusting Confidence & IoU Thresholds
You can fine-tune sensitivity directly when initializing the detector:
```python
# Lower conf_thres detects more objects (more false positives)
# Higher conf_thres detects only high-certainty objects
yolov8_detector = YOLOv8(model_path, conf_thres=0.3, iou_thres=0.45)
```
