# Quadruped I

A budget-friendly, fully 3D-printable Spot-style robot dog. Ten servos, a
Raspberry Pi 4 brain, an Arduino Nano for sensing, and a tele-op
controller. This repo holds the Python code that runs on the Pi: the
gait engine, the keyboard / network controllers, and the optional video
streamer.

For the full hardware build (CAD, BOM, print settings, step-by-step
assembly), see the project listing on Printables. This README only
covers the **code**.

> Looking for the bigger sibling with bearings, 12 servos, and a ROS
> Noetic stack? See [Quadruped v1.0](https://www.printables.com/model/1451339-quadruped-v10-full-project).

---

## Repo layout

All code currently lives under `quadruped-robot-main/`:

```
quadruped-robot-main/
├── control_quadruped.py           # top-level launcher
├── gait_logic/
│   ├── quadruped.py               # core gait + servo driving
│   ├── stair.py / stairs.py       # stair-climbing gait (see note below)
├── controllers/
│   ├── local_keyboard_controller.py     # drive from a keyboard on the Pi
│   ├── network_sender_keyboard.py       # send keypresses from a laptop
│   ├── network_receiver.py              # receive keypresses on the Pi
│   ├── object_tracker_controller.py     # vision-based tracking
│   └── utils/ip_helper.py
├── image-sender/                  # Pi-side video streaming
│   ├── rpi_send_video.py
│   └── imagezmq.py                # vendored copy of the imagezmq library
└── image-receiver/
    └── receive_and_store_video.py # laptop-side video receiver
```

> **Note:** `gait_logic/stair.py` and `gait_logic/stairs.py` are nearly
> identical in size and look like duplicates. One of them is the
> canonical version — TODO: confirm which and delete the other.

---

## Hardware

- Raspberry Pi 4 (running the Python code in this repo)
- Arduino Nano (sensing — firmware not currently in this repo, TODO)
- 10 × hobby servos (see the BOM in the Printables listing for exact model)
- Servo driver board (PCA9685 or equivalent — TODO: confirm)
- A controller for tele-op (keyboard supported today; PS4 controller
  is mentioned in the project description but no joystick code is
  present in this repo yet — TODO)

---

## Pin / servo-channel assignments

> **TODO — maker to fill in.** The servo-channel-to-joint mapping is
> defined inside `gait_logic/quadruped.py`. Hoist it into the table
> below so a builder doesn't have to read source to wire the legs.

| Channel | Joint           | Notes |
|--------:|-----------------|-------|
|       0 | FL hip pivot    | TODO  |
|       1 | FL upper leg    | TODO  |
|       2 | FL lower leg    | TODO  |
|       3 | FR hip pivot    | TODO  |
|       4 | FR upper leg    | TODO  |
|       5 | FR lower leg    | TODO  |
|       6 | RL upper leg    | TODO  |
|       7 | RL lower leg    | TODO  |
|       8 | RR upper leg    | TODO  |
|       9 | RR lower leg    | TODO  |

(Quadruped I has 10 servos — only the **front** hips pivot, hence 3
joints per front leg and 2 per rear leg.)

---

## Install (Raspberry Pi)

Tested on a Raspberry Pi 4 running Raspberry Pi OS (Bookworm or Bullseye).

```bash
sudo apt update
sudo apt install -y python3-pip python3-venv git i2c-tools

git clone https://github.com/ToolKnox/Quadruped-I.git
cd Quadruped-I/quadruped-robot-main

python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt   # TODO: requirements.txt is not yet committed
```

> **TODO — maker:** run `pip freeze > requirements.txt` on the working
> Pi and commit it. Until then, the dependency list below is a best
> guess; install whatever your imports complain about:
>
> ```bash
> pip install imagezmq opencv-python numpy pyzmq adafruit-circuitpython-servokit
> ```

Enable I²C on the Pi (for the servo driver):

```bash
sudo raspi-config         # Interface Options → I2C → Enable
sudo i2cdetect -y 1       # confirm the servo driver shows up
```

---

## Run

### 1. Drive locally with a keyboard (Pi-only)

Plug a keyboard into the Pi, then:

```bash
cd quadruped-robot-main
python control_quadruped.py
```

Controls live in `controllers/local_keyboard_controller.py` — TODO:
list the actual key bindings here once verified against source.

### 2. Drive from a laptop over the network

On the **Pi** (receiver):

```bash
python controllers/network_receiver.py
```

On your **laptop** (sender):

```bash
python controllers/network_sender_keyboard.py
```

The sender needs to know the Pi's IP — see `controllers/utils/ip_helper.py`.

### 3. (Optional) Stream video from the Pi camera to the laptop

On the **Pi**:

```bash
python image-sender/rpi_send_video.py
```

On your **laptop**:

```bash
python image-receiver/receive_and_store_video.py
```

### 4. Object-tracking controller

`controllers/object_tracker_controller.py` runs a vision-based
controller that drives the robot toward a tracked object. TODO: document
which object/colour it tracks and what model (if any) it loads.

---

## Troubleshooting

- **`i2cdetect` shows nothing** — I²C isn't enabled, or the servo driver
  isn't wired to SDA/SCL/3V3/GND. Re-run `raspi-config` and re-check.
- **Servos twitch on boot** — the driver is getting power from the Pi's
  3V3 rail instead of a dedicated battery / BEC. Servos need their own
  5–6 V supply with the grounds tied together.
- **`ModuleNotFoundError: imagezmq`** — either `pip install imagezmq`,
  or run from the `image-sender/` directory so the vendored copy is on
  the path.
- **Network controller doesn't connect** — check that the laptop and Pi
  are on the same subnet and that the Pi's firewall (if any) allows the
  port used by `network_receiver.py`.

---

## License

> **TODO — maker to confirm.** No `LICENSE` file is currently committed.
> Defaulting to the license on the Printables listing. Remixes welcome;
> contact the maker for commercial use.

---

## Credits

Built by [ToolKnox](https://github.com/ToolKnox). Full hardware project,
print files, and assembly walkthrough on Printables.