# VR Hand Steering

Drive racing games with just your webcam and your hands. The app tracks both hands with [MediaPipe](https://developers.google.com/mediapipe), turns your gestures into steering, throttle, brake and boost, and sends them to Windows as a **virtual Xbox 360 controller**, so any game with controller support works with no game-side setup.

## How it works

```
Webcam → MediaPipe Hand Landmarker (2 hands, 21 landmarks each)
       → Gesture logic (steering / pedals)
       → Virtual Xbox 360 controller (vgamepad + ViGEmBus)
       → Game
```

| Gesture | Controller output |
|---|---|
| Tilt the imaginary wheel (angle of the line between your two hands) | Left stick X (-1 to +1) |
| Close your four fingers (more fingers closed = more throttle) | Right trigger (RT), analog 0-100% |
| Fold your **right** thumb | Left trigger (LT) = brake |
| Fold your **left** thumb | **A** button = boost |

Details:
- **Steering** is calibrated automatically the first time both hands are visible; that pose becomes "straight". Full lock is reached at 45° of tilt, with smoothing to avoid jitter.
- **Throttle** is read from the left hand, or the right hand if the left isn't visible. It is smoothed, so it ramps rather than jumps.
- **Brake/boost** use a short debounce (a gesture must hold for a few frames) to avoid flicker.
- A live on-screen dashboard shows steering angle, throttle bar, and brake/boost state.

## Requirements

- **Windows 10/11** (the virtual controller driver and camera backends are Windows-specific)
- **Python 3.9 - 3.12**
- A webcam
- **[ViGEmBus driver](https://github.com/nefarius/ViGEmBus/releases)**, required by `vgamepad` to create the virtual controller

## Setup

```bash
git clone https://github.com/<your-username>/vr-hand-steering.git
cd vr-hand-steering

python -m venv .venv
.venv\Scripts\activate
pip install -r requirements.txt
```

Download the MediaPipe hand model into `models/mediapipe/`:

```powershell
curl -L -o models/mediapipe/hand_landmarker.task https://storage.googleapis.com/mediapipe-models/hand_landmarker/hand_landmarker/float16/latest/hand_landmarker.task
```

## Run

```bash
python tracking/hand_landmarker.py
```

Hold both hands up in front of the camera, as if gripping a steering wheel. Press **Q** in the video window to quit.

Options (environment variables):

| Variable | Default | Purpose |
|---|---|---|
| `CAMERA_INDEX` | `0` | Which camera to use (`set CAMERA_INDEX=1` in cmd) |
| `DEBUG` | `0` | Set to `1` to print landmark and thumb-angle debug output |

Tip: open your game's controller settings and test the inputs. Windows' "Set up USB game controllers" (`joy.cpl`) also shows the virtual controller live.

## Project structure

```
vr-hand-steering/
├── tracking/
│   └── hand_landmarker.py   # entry point: camera loop, MediaPipe, wiring
├── control/
│   ├── steering.py          # hand angle → steering value
│   ├── pedals.py            # finger/thumb gestures → throttle, brake, boost
│   └── controllers.py       # virtual Xbox 360 controller wrapper
├── visualization/
│   └── dashboard.py         # on-screen HUD
├── models/mediapipe/        # place hand_landmarker.task here (git-ignored)
└── requirements.txt
```

## Troubleshooting

- **"Could not open camera"**: close other apps using the webcam, check Windows camera permissions, or try `CAMERA_INDEX=1`.
- **Controller not appearing**: make sure the ViGEmBus driver is installed (reboot after installing).
- **Model not found**: confirm `models/mediapipe/hand_landmarker.task` exists.
- **Gestures flicker or misfire**: use even lighting, keep both hands fully in frame, and adjust thresholds in `control/pedals.py` and `control/steering.py`.
- Hand labels are intentionally swapped in code because the camera image is mirrored.

## Roadmap

- [ ] Keyboard shortcut to recalibrate steering
- [ ] Config file for thresholds and button mapping
- [ ] Demo GIF
- [ ] Unit tests for the gesture logic

## License

MIT, see [LICENSE](LICENSE).
