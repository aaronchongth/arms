# arms

Primarily the SO-101 arm.

The `lerobot` directory will be updated to be similar to `~/.cache/huggingface/lerobot`.

## Setup

The leader (teleoperation) arm is on `/dev/ttyACM0`.

The follower arm is on `/dev/ttyACM1`.

Calibrate,

```bash
# leader
lerobot-calibrate \
  --teleop.type=so101_leader \
  --teleop.port=/dev/ttyACM0 \
  --teleop.id=my_awesome_leader_arm

# follower
lerobot-calibrate \
  --robot.type=so101_follower \
  --robot.port=/dev/ttyACM1 \
  --robot.id=my_awesome_follower_arm
```

Teleoperate,

```bash
lerobot-teleoperate \
  --robot.type=so101_follower \
  --robot.port=/dev/ttyACM1 \
  --robot.id=my_awesome_follower_arm \
  --teleop.type=so101_leader \
  --teleop.port=/dev/ttyACM0 \
  --teleop.id=my_awesome_leader_arm
```

Camera (make sure to disconnect webcam),

```bash
--- Detected Cameras ---
Camera #0:
  Name: OpenCV Camera @ /dev/video2
  Type: OpenCV
  Id: /dev/video2
  Backend api: V4L2
  Default stream profile:
    Format: 0.0
    Fourcc: YUYV
    Width: 640
    Height: 480
    Fps: 30.0
--------------------

# To use during dataset recording
--robot.cameras="{ wrist: {type: opencv, index_or_path: 0, width: 640, height: 480, fps: 30}, ...}"
```

Realsense,

```bash
# Make sure to install by running
# pip install lerobot[intelrealsense]

--- Detected Cameras ---
Camera #0:
  Name: Intel RealSense D435I
  Type: RealSense
  Id: 141722074564
  Firmware version: 5.17.0.10
  Usb type descriptor: 2.1
  Physical port: /sys/devices/pci0000:00/0000:00:07.1/0000:2f:00.3/usb7/7-2/7-2:1.0/video4linux/video2
  Product id: 0B3A
  Product line: D400
  Default stream profile:
    Stream_type: Color
    Format: rgb8
    Width: 640
    Height: 480
    Fps: 15
--------------------

# To use during dataset recording
--robot.cameras="{ top: {type: intelrealsense, serial_number_or_name: 141722074564, width: 640, height: 480, fps: 15, use_depth: true}, ...}"
```

Use both cameras,

```bash
--robot.cameras="{ wrist: {type: opencv, index_or_path: 0, width: 640, height: 480, fps: 30}, top: {type: intelrealsense, serial_number_or_name: 141722074564, width: 640, height: 480, fps: 15, use_depth: true}}"
```

## Grab the red cube

Data collection,

```bash
lerobot-record \
    --robot.type=so101_follower \
    --robot.port=/dev/ttyACM1 \
    --robot.id=my_awesome_follower_arm \
    --robot.cameras="{ wrist: {type: opencv, index_or_path: 0, width: 640, height: 480, fps: 30}, top: {type: intelrealsense, serial_number_or_name: 141722074564, width: 640, height: 480, fps: 15, use_depth: true}}" \
    --teleop.type=so101_leader \
    --teleop.port=/dev/ttyACM0 \
    --teleop.id=my_awesome_leader_arm \
    --display_data=true \
    --dataset.repo_id=${HF_USER}/put-red-cube-in-box \
    --dataset.episode_time_s=20 \
    --dataset.reset_time_s=10 \
    --dataset.num_episodes=50 \
    --dataset.single_task="Put red cube in box" \
    --dataset.streaming_encoding=true \
    --dataset.encoder_threads=2 \
    --dataset.push_to_hub=False
```

Training ACT,

```bash
lerobot-train \
  --dataset.repo_id=aaronchongth/put-red-cube-in-box \
  --policy.type=act \
  --output_dir=outputs/train/hf_act_record0 \
  --job_name=hf_act_training_job \
  --policy.device=cuda \
  --wandb.enable=False \
  --policy.repo_id=aaronchongth/hf_act_recordpolicy0
```

Evaluating ACT,

```bash
DISPLAY=:0 lerobot-record \
    --robot.type=so100_follower \
    --robot.port=/dev/ttyACM1 \
    --robot.id=my_awesome_follower_arm \
    --robot.cameras="{ wrist: {type: opencv, index_or_path: 0, width: 640, height: 480, fps: 30}, top: {type: intelrealsense, serial_number_or_name: 141722074564, width: 640, height: 480, fps: 15, use_depth: true}}" \
    --display_data=true \
    --dataset.repo_id=aaronchongth/eval_act_your_dataset \
    --dataset.num_episodes=10 \
    --dataset.episode_time_s=120 \
    --dataset.single_task="Put red cube in box" \
    --dataset.streaming_encoding=true \
    --dataset.encoder_threads=2 \
    --policy.path=aaronchongth/hf_act_recordpolicy0
```

