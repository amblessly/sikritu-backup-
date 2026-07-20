# Human Detection Drone-Vision Prototype

An educational, prototype-stage human detection pipeline that simulates the
vision system of an early-stage surveillance / search-and-rescue drone.

It streams live video from an Android phone (no webcam required), runs a
real CNN object detector (**YOLOv8n**) on every frame, draws bounding boxes
and confidence scores around detected people, and logs every detection event
to CSV/JSON for future dashboard use.

> ⚠️ **This is a prototype for learning purposes**, not a production
> surveillance or rescue system. Detection accuracy, latency, and reliability
> have not been validated for any real safety-critical use.

---

## 1. Project Structure

```
project/
│
├── main.py              # Entry point: video loop, FPS, logging, display
├── detector.py           # YOLOv8n CNN wrapper (person-only detection)
├── logger.py              # CSV + JSON detection logging, screenshots
├── detections.csv         # Detection log (timestamp, confidence, frame_number)
├── detections.json        # Same data in JSON, for future dashboard use
├── detections/            # Auto-saved screenshots of each detection
├── requirements.txt       # Python dependencies
└── README.md              # This file
```

---

## 2. Installation

### 2.1 Requirements
- Python 3.11+ installed on your laptop/PC
- An Android phone with a camera
- Both devices connected to the **same Wi-Fi network**

### 2.2 Install Python dependencies

From inside the `project/` folder, run:

```bash
pip install -r requirements.txt
```

This installs:
- `opencv-python` — video capture & image processing
- `numpy` — array operations
- `ultralytics` — YOLOv8n model loading and inference
- `torch` / `torchvision` — deep learning backend used by YOLOv8
- `requests` — general HTTP utilities (useful for future dashboard work)
- `pandas` — convenient CSV/data handling (useful for future dashboard work)

The first time you run `main.py`, Ultralytics will **automatically download**
the `yolov8n.pt` weights file (a few MB) into the project folder — you need
an internet connection for this one-time download only.

---

## 3. Phone Camera Setup (IP Webcam App)

Since there's no physical webcam, this prototype uses the free Android app
**"IP Webcam"** to turn your phone into a network camera.

### 3.1 Install IP Webcam
1. Open the **Google Play Store** on your Android phone.
2. Search for **"IP Webcam"** (developer: Pavel Khlebovich).
3. Install the free app.

### 3.2 Start the server on your phone
1. Open the IP Webcam app.
2. Scroll down and tap **"Start server"**.
3. The app will show a live preview and, at the bottom of the screen, an
   address like:
   ```
   http://192.168.1.42:8080
   ```

### 3.3 Get your phone's IP address
- This address is displayed directly in the IP Webcam app once the server
  starts (e.g. `192.168.1.42`).
- Make sure your phone is connected to **Wi-Fi**, not mobile data — IP
  Webcam needs your phone and laptop on the same local network.

### 3.4 Connect Python to the stream
1. Open `main.py` in a text editor.
2. Find this line near the top:
   ```python
   PHONE_IP = "192.168.1.42"
   ```
3. Replace `"192.168.1.42"` with your phone's actual IP address from step 3.3.
4. The full stream URL the code uses is:
   ```
   http://PHONE_IP:8080/video
   ```
5. **Tip:** before running the Python code, test the URL in a normal web
   browser on your laptop (e.g. `http://192.168.1.42:8080/video`). You
   should see the live MJPEG video stream. If that works in the browser,
   it will work in OpenCV too.

---

## 4. Running the System

From inside the `project/` folder:

```bash
python main.py
```

What happens:
1. The script connects to your phone's video stream.
2. The YOLOv8n model loads (downloads automatically on first run).
3. A window titled **"Human Detection - Drone Vision Prototype"** opens,
   showing the live feed with:
   - Green bounding boxes around detected people
   - A label like `Person (92%)` above each box
   - An FPS counter (top-left)
   - A running detection count (top-left, below FPS)
4. Every time a person is detected (after a short cooldown to avoid
   duplicate spam), the event is:
   - Appended to `detections.csv`
   - Appended to `detections.json`
   - Saved as a screenshot in `detections/`, named like
     `person_20260624_182215.jpg`
5. Press **`q`** while the video window is focused to stop the program.

---

## 5. Expected Output

**Console output** while running:
```
============================================================
Human Detection Drone-Vision Prototype
============================================================
Connecting to phone camera stream at: http://192.168.1.42:8080/video
[OK] Connected to phone camera stream.

[HumanDetector] Loading YOLOv8n model from 'yolov8n.pt'...
[HumanDetector] Model loaded successfully.
Press 'q' in the video window to quit.

[DETECTION] Frame 125 | Confidence: 93% | Logged to CSV/JSON + screenshot saved.
[DETECTION] Frame 140 | Confidence: 88% | Logged to CSV/JSON + screenshot saved.

[INFO] 'q' pressed. Shutting down...

Session summary: 612 frames processed, 14 total person detections.
Detection logs saved to detections.csv and detections.json
Screenshots saved to the detections/ folder.
```

**detections.csv** example:
```
timestamp,confidence,frame_number
2026-06-24 18:22:15,0.93,125
2026-06-24 18:22:21,0.88,140
```

**detections.json** example:
```json
[
  {
    "timestamp": "2026-06-24 18:22:15",
    "confidence": 0.93,
    "frame_number": 125
  },
  {
    "timestamp": "2026-06-24 18:22:21",
    "confidence": 0.88,
    "frame_number": 140
  }
]
```

**detections/ folder**: contains `.jpg` screenshots named like
`person_20260624_182215.jpg`, one for each logged detection.

---

## 6. Troubleshooting Guide

| Problem | Likely Cause | Fix |
|---|---|---|
| `[ERROR] Could not open the video stream.` | Wrong IP, phone not on same Wi-Fi, or server not started | Re-check `PHONE_IP`, confirm both devices on same network, tap "Start server" again in IP Webcam |
| Video window opens but is black/frozen | Phone screen turned off / app backgrounded | Keep the IP Webcam app open and the phone screen on while streaming |
| Very low FPS / laggy video | CPU-only inference, high phone camera resolution | In the IP Webcam app, lower the video resolution/quality in its settings |
| `ModuleNotFoundError: No module named 'ultralytics'` (or similar) | Dependencies not installed | Run `pip install -r requirements.txt` again, ensure you're using the right Python environment |
| Model download hangs on first run | No internet connection | Connect to the internet once so `yolov8n.pt` can download, then it's cached locally |
| No detections ever appear | Confidence threshold too high, poor lighting, person not fully in frame | Lower `CONFIDENCE_THRESHOLD` in `main.py` (e.g. to `0.35`), improve lighting |
| `detections.csv` / `detections.json` not updating | Detection cooldown (`LOG_COOLDOWN_FRAMES`) still counting down | Wait a few more frames, or lower `LOG_COOLDOWN_FRAMES` in `main.py` |
| OpenCV window doesn't respond to `q` | Window not focused | Click on the video window first, then press `q` |

---

## 7. Notes on the CNN Model Choice

This prototype uses **YOLOv8n ("Nano")** from Ultralytics because:
- It is a genuine CNN-based object detector (not a Haar Cascade).
- The "Nano" variant is the smallest/fastest YOLOv8 model, making it
  realistic to run in real time on a typical laptop CPU.
- It comes pretrained on the COCO dataset, where `person` is class `0`,
  so no custom training or dataset collection is needed for this
  prototype — `detector.py` simply filters predictions down to that
  one class.

---

## 8. Possible Next Steps (Beyond This Prototype)

- Build a web dashboard (e.g. Flask + Chart.js) that reads `detections.json`
  and plots detection events over time.
- Add multi-person tracking (e.g. with a tracker like ByteTrack) instead of
  per-frame detection only.
- Add GPS/telemetry data alongside detections for real drone integration.
- Switch to a larger YOLOv8 variant (`yolov8s.pt`, `yolov8m.pt`, etc.) if
  running on a GPU, for higher accuracy at the cost of speed.
