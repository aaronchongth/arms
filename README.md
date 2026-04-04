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
