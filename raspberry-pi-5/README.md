# Human Detection Prototype — Raspberry Pi 5

An educational, prototype-stage **AI-powered health monitoring system** that
simulates human detection and vital-sign estimation on a **Raspberry Pi 5**. It
uses OpenCV + YOLOv8 for person detection, simulates vital signs (heart rate,
respiration, SpO₂, temperature), packs the results into JSON, and transmits
them over a *simulated* LoRa link to a ground station dashboard.

> ⚠️ **Prototype / learning project.** Not a production medical or safety
> system. Detection accuracy, latency, and reliability are not validated for
> real-world use.

---

## Hardware Overview (Target)

| Property | Value |
|---|---|
| Device | Raspberry Pi 5 |
| CPU | Broadcom BCM2712 |
| RAM | 8GB |
| Status | Ready |
| LoRa | Simulation Mode |
| Camera | USB Camera (Connected) |

All device metadata lives in `config.py` (`DEVICE_NAME`, `CPU_MODEL`,
`RAM_SIZE`, `STATUS`, `LORA_MODE`, `CAMERA_NAME`, `DEVICE_VERSION`).

---

## Software Requirements

- Raspberry Pi OS (or any Linux) on a Raspberry Pi 5
- Python 3.9+
- USB camera
- OpenCV, NumPy, Ultralytics (YOLOv8), PyTorch, Flask

Install dependencies:

```bash
pip install -r requirements.txt
```

The first run automatically downloads `yolov8n.pt` (a few MB) if not present.

---

## Installation

```bash
# 1. Create and activate a virtual environment
python -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate

# 2. Install dependencies
pip install -r requirements.txt
```

---

## Running the Project

```bash
python main.py
```

On startup the console prints the Raspberry Pi 5 device information, then
loads the YOLO model, opens the USB camera, and starts the pipeline:

1. USB camera feed captured via OpenCV.
2. YOLOv8 person detection every frame.
3. Simulated vital signs estimated per detection.
4. JSON detection packet built.
5. Packet transmitted over the **simulated** LoRa link (UDP).
6. Ground Station Dashboard served at `http://localhost:5000`.

Press **`q`** in the detection window to quit.

To verify the camera before running:

```bash
python test_cam.py
```

---

## Features (preserved from original)

- OpenCV camera detection
- YOLO object/person detection
- Simulated heart rate generation
- Simulated body temperature generation
- JSON packet creation
- Simulated LoRa transmitter
- Simulated LoRa receiver
- Timestamp
- Confidence score
- Detection status
- Console logging

---

## Future Integration — Real LoRa

The simulated sender/receiver use UDP sockets as a stand-in for the radio link.
To deploy real LoRa on the Raspberry Pi 5:

- Add a LoRa module (e.g. SX1278 / RFM95W) via SPI.
- Replace the socket transport in `sender.py` / `receiver.py` with the
  hardware driver (e.g. `pyserial` + AT commands, or a SPI LoRa library).
- Keep `packet.py` and `DetectionPacket.build` unchanged — only the transport
  layer is swapped.

---

## Future Deployment — Actual Raspberry Pi Hardware

- Run headless with `sudo systemctl` or a startup script.
- Use the Pi's USB port for the camera and GPIO/SPI for the LoRa module.
- The `config.py` values can be updated to reflect the real deployed hardware
  (RAM size, CPU, LoRa mode) without touching pipeline code.

---

## Project Structure

```
raspberry-pi-5/
├── main.py            # Entry point: pipeline, device info, HUD, display
├── config.py          # Raspberry Pi 5 device configuration
├── detector.py        # YOLOv8n person detector
├── human_detection.py # Detection interface wrapper
├── vital_signs.py     # Simulated vital signs
├── packet.py          # JSON packet schema
├── sender.py          # Simulated LoRa transmitter
├── receiver.py        # Simulated LoRa receiver
├── logger.py          # CSV + JSON logging + screenshots
├── dashboard.py       # Flask ground station dashboard
├── utils.py           # Shared constants + timestamp helpers
├── test_cam.py        # Camera sanity check
├── requirements.txt   # Python dependencies
├── yolov8n.pt         # YOLOv8 Nano weights
├── detections/        # Auto-saved detection screenshots
└── README.md          # This file
```
