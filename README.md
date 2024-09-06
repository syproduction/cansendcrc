# cansendcrc
Script for automatic CRC calculation for cansend.
It will calculate proper CRC and execute cansend command with CRC applied.

Tested with Raspberry Pi 5, Waveshare RS485 CAN HAT, MKS SERVO42D CAN MB.

## Installation
Download `cansendsrc` file or git clone this repository

`sudo chmod +x cansendsrc`

## Usage
```sh
./cansendcrc <CAN_ID>#<data_bytes>
```
Where 

`<CAN_ID>` is slave device ID

`<data_bytes>` are the actual bytes without CRC


## Example
Command:
```sh
./cansendcrc 002#f300
```

Answer:
`Calculated CRC: F5
cansend can0 002#f300F5`

## Installation of Waveshare RS485 CAN HAT on Raspberry Pi 5
```sh
sudo apt update && sudo apt upgrade -y
sudo apt-get install can-utils -y
echo -e "dtparam=spi=on\ndtoverlay=mcp2515-can0,oscillator=12000000,interrupt=25,spimaxfrequency=2000000" | sudo tee -a /boot/firmware/config.txt
# check oscillator on  RS485 CAN HAT is 12.000
sudo reboot now
sudo ip link set can0 down
sudo ip link set can0 up type can bitrate 500000 #you need to set CanRate on the motor to 500k manually
wget https://github.com/syproduction/cansendcrc/raw/main/cansendcrc
sudo chmod +x cansendcrc
```

## Commands for MKS SERVO42D CAN MB

Commands are without CRC since we dont need it with `cansendcrc`

`CanID` for the motor is `2`, thats why there are `002` before `#`

Answers were received with `candump can0`

# Table of Contents

- [`30` Read the Encoder Value (Carry)](#30-read-the-encoder-value-carry)
- [`31` Read the Encoder Value (Addition)](#31-read-the-encoder-value-addition)
- [`30 32` Read the Real-Time Speed of the Motor (RPM)](#30-32-read-the-real-time-speed-of-the-motor-rpm)
- [`33` Read the Number of Pulses Received](#33-read-the-number-of-pulses-received)
- [`39` Read the Error of the Motor Shaft Angle](#39-read-the-error-of-the-motor-shaft-angle)
- [`3A` Read the EN Pins Status](#3a-read-the-en-pins-status)
- [`3B` Read the Go Back to Zero Status When Power On](#3b-read-the-go-back-to-zero-status-when-power-on)
- [`3D` Release the Motor Shaft Locked-Rotor Protection State](#3d-release-the-motor-shaft-locked-rotor-protection-state)
- [`3E` Read the Motor Shaft Protection State](#3e-read-the-motor-shaft-protection-state)
- [`80 00` Calibrate the Encoder (Same as the "Cal" option on screen)](#80-00-calibrate-the-encoder-same-as-the-cal-option-on-screen)
- [`82` Set the Work Mode (Same as the "Mode" option on screen)](#82-set-the-work-mode-same-as-the-mode-option-on-screen)
- [`83` Set the Current (Same as the "Ma" option on screen)](#83-set-the-current-same-as-the-ma-option-on-screen)
- [`84` Set Subdivision (Same as the "MStep" option on screen)](#84-set-subdivision-same-as-the-mstep-option-on-screen)
- [`85` Set the Active of the EN Pin](#85-set-the-active-of-the-en-pin)
- [`86` Set the Direction of Motor Rotation](#86-set-the-direction-of-motor-rotation)
- [`87` Set Auto Turn Off the Screen Function](#87-set-auto-turn-off-the-screen-function)
- [`88` Set the Motor Shaft Locked-Rotor Protection Function](#88-set-the-motor-shaft-locked-rotor-protection-function)
- [`89` Set the Subdivision Interpolation Function](#89-set-the-subdivision-interpolation-function)
- [`8A` Set the CAN Bit Rate](#8a-set-the-can-bit-rate)
- [`8B` Set the CAN ID](#8b-set-the-can-id)
- [`8C` Set the Slave Respond](#8c-set-the-slave-respond)
- [`8D` Set the Group ID](#8d-set-the-group-id)
- [`90` Set the Parameter of Home](#90-set-the-parameter-of-home)
- [`91` Go Home](#91-go-home)
- [`92` Set Current Axis to Zero](#92-set-current-axis-to-zero)
- [`3F` Restore the Default Parameter](#3f-restore-the-default-parameter)
- [`F1` Query the Motor Status](#f1-query-the-motor-status)
- [`F3` Enable/Disable the Motor](#f3-enable-disable-the-motor)
- [`F6` Run the Motor in Speed Mode](#f6-run-the-motor-in-speed-mode)
- [`F6` Stop the Motor in Speed Mode](#f6-stop-the-motor-in-speed-mode)
- [`FF` Save/Clear the Parameter in Speed Mode](#ff-save-clear-the-parameter-in-speed-mode)
- [`FD` Run the Motor in Position Mode1](#fd-run-the-motor-in-position-mode1)
- [`FD` Stop the Motor in Position Mode1](#fd-stop-the-motor-in-position-mode1)
- [`F4` Run the Motor in Position Mode2](#f4-run-the-motor-in-position-mode2)
- [`FD` Stop the Motor in Position Mode2](#fd-stop-the-motor-in-position-mode2)
- [`F5` Run the Motor in Position Mode3](#f5-run-the-motor-in-position-mode3)
- [`F5` Stop the Motor in Position Mode3](#f5-stop-the-motor-in-position-mode3)

---
 
### `30` Read the Encoder Value (Carry)

| Command    | Answer                    | Comment |
|------------|---------------------------|---------|
| `002#30`   | `can0  002   [8]  30 00 00 00 00 00 0B 3D` |         |
| **Where**  |                           |         |
|            | CAN ID: `002` (Identifier for the response) |         |
|            | DLC: `8` (Number of data bytes in the response) |         |
|            | Byte1: `30` (Indicates the response is for the 30 command) |         |
|            | Bytes 2-5: `00 00 00 00` (Carry, 32-bit integer showing the number of full rotations) |         |
|            | Bytes 6-7: `0B` (Value, 16-bit integer showing the position within one rotation) | 1° ≈ 11.38 Value |
|            | Byte8: `3D` (CRC, checksum for the response data) |         |

### `31` Read the Encoder Value (Addition)

| Command    | Answer                    | Comment |
|------------|---------------------------|---------|
| `002#31`   | `can0  002   [8]  31 00 00 00 00 00 0C 3F` |         |
| **Where**  |                           |         |
|            | CAN ID: `002` (Identifier for the response) |         |
|            | DLC: `8` (Number of data bytes in the response) |         |
|            | Byte1: `31` (Indicates the response is for the 31 command) |         |
|            | Bytes 2-6: `00 00 00 00` (Reserved or padding bytes) |         |
|            | Byte7: `0C` (Value, 16-bit integer, little-endian format) |         |
|            | Byte8: `3F` (CRC, checksum for the response data) |         |


### `32` Read the Real-Time Speed of the Motor (RPM)

| Command    | Answer                    | Comment |
|------------|---------------------------|---------|
| `002#32` | `can0  002   [4]  32 00 00 34` |         |
| **Where**  |                           |         |
|            | CAN ID: `002` (Identifier for the response) |         |
|            | DLC: `4` (Number of data bytes in the response) |         |
|            | Byte1: `32` (Indicates the response is for the 3032 command) |         |
|            | Bytes2-3: `00 0C` (Value, 16-bit integer in little-endian format) | Real-time speed of the motor in RPM. |
|            | Byte4: `34` (CRC, checksum for the response data) |         |


### `33` Read the Number of Pulses Received

| Command    | Answer                    | Comment |
|------------|---------------------------|---------|
| `002#33`   | `can0  002   [6]  33 00 00 00 00 35` |         |
| **Where**  |                           |         |
|            | CAN ID: `002` (Identifier for the response) |         |
|            | DLC: `6` (Number of data bytes in the response) |         |
|            | Byte1: `33` (Indicates the response is for the 33 command) |         |
|            | Bytes2-5: `00 00 00 00` (Value, 32-bit integer in little-endian format) | The number of pulses received by the motor. |
|            | Byte6: `35` (CRC, checksum for the response data) |         |


### `39` Read the Error of the Motor Shaft Angle
 The error is the difference between the angle you want to control minus the real-time angle of the motor, 0~FFFF corresponds to 0~360°. 
| Command    | Answer                    | Comment |
|------------|---------------------------|---------|
| `002#39`   | `can0  002   [6]  39 00 00 00 1A 55` |         |
| **Where**  |                           |         |
|            | CAN ID: `002` (Identifier for the response) |         |
|            | DLC: `6` (Number of data bytes in the response) |         |
|            | Byte1: `39` (Indicates the response is for the 39 command) |         |
|            | Bytes2-5: `00 00 00 1A` (Value, 32-bit integer in little-endian format) |
|            | Byte6: `55` (CRC, checksum for the response data) |         |

**Note:**
- The error is calculated as the difference between the desired angle and the real-time angle of the motor.
- `0xFFFF` (65536 in decimal) corresponds to 360°.
- For example, if the angle error is `1°`, the returned error value is approximately `182` (65536/360 ≈ 182.444).


### `3A` Read the EN Pins Status

| Command    | Answer                    | Comment |
|------------|---------------------------|---------|
| `002#3A`   | `  can0  002   [3]  3A 01 3D` |
| **Where**  |                           |
|     | Byte1: `3A` (Indicates the response is for the 3A command) |
|    | Byte2: `01` (Value, 32-bit integer in little-endian format) | Enabled: Value = `1` , Disabled: Value = `0` |
|    | Byte6: `3D` (CRC, checksum for the response data) |


### `3B` Read the Go Back to Zero Status When Power On

| Command    | Answer                    |
|------------|---------------------------|
| `002#3B`   | `can0  002   [3]  3B 02 3F`|
| **Where**  |                           |
|   | Byte2: `02` (Status) |
|            | `status = 0` Going to zero |
|            | `status = 1` Go back to zero success |
|            | `status = 2` Go back to zero fail |


### `3D` Release the Motor Shaft Locked-Rotor Protection State

| Command    | Answer                    |
|------------|---------------------------|
| `002#3D`   | `can0  002   [3]  3D 00 3F`|
| **Where**  |                           |
|  | Byte2: `00` (Status) |
|            | `status = 1` Release success |
|            | `status = 0` Release fail    |

### `3E` Read the Motor Shaft Protection State

| Command    | Answer                    |
|------------|---------------------------|
| `002#3E`   | `can0  002   [3]  3E 00 40`|
| **Where**  |                           |
|  | Byte2: `00` (Status) |
|            | `status = 1` Protected     |
|            | `status = 0` Not protected |


### `80 00` Calibrate the Encoder (Same as the "Cal" option on screen)

| Command    | Answer                    |
|------------|---------------------------|
| `002#8000` | `can0  002   [3]  80 00 82` <br> `can0  002   [3]  80 01 83` |
| **Where**  |                           |
|            | Byte2: `00`, `01`, `02` (Status) |
|            | `status = 0` Calibrating… |
|            | `status = 1` Calibrated success |
|            | `status = 2` Calibrating fail |

**Note**: The motor must be unloaded


### `82` Set the Work Mode (Same as the "Mode" option on screen)

| Command    | Answer                    |
|------------|---------------------------|
| `002#8204` | `can0  002   [3]  82 01 85`|
| **Where**  |                           |
| Byte2: `04` (Mode) | Byte2: `01` (Status)  |
|            | `status = 1` Set success  |
|            | `status = 0` Set fail     |


| Mode     | Name   | Interface   | Max RPM   |  Work Current    | Encoder |
|----------|--------|-------------|-----------|------------------| --------|
| 0        |CR_OPEN | Pulse       | 400       | Fix, `Ma`        | Without |
| 1        |CR_CLOSE | Pulse       | 1500       | Fix, `Ma`        | With   |
| 2        |CR_vFOC | Pulse       | 3000       | Adaptive, Max current = `Ma`        | With   |
| 3        |SR_OPEN | Serial       | 400       | Fix, `Ma`        | Without |
| 4       |SR_CLOSE | Serial       | 1500       | Fix, `Ma`        | With   |
| 5        |SR_vFOC | Serial       | 3000       | Adaptive, Max current = `Ma`        | With   |

(Default: CR_vFOC)

### `83` Set the current （Same as the "Ma" option on screen）
| Command       | Answer                    |
|---------------|---------------------------|
| `002#8300AA`  | `can0  002   [3]  83 01 86`|
| **Where**     |                           |
| Bytes2-3: `00 AA` (Value 0000-0BB8, 16-bit hex integer) | Byte2: `01` (Status) |
|               | `status = 1` Set success  |
|               | `status = 0` Set fail     |

**Device Information:**
- **SERVO42D**: Maximum Current = 3000mA
- **SERVO57D**: Maximum Current = 5200mA

### `84` Set subdivision （Same as the "MStep" option on screen）

| Command    | Answer                       |
|------------|------------------------------|
|`002#8410`  |`can0  002   [3]  84 01 87`   |
| Where      |                              |
|Byte2 : `10` Value 00-ff (16 bit hex integer) | Byte2 : `01` Status   |
|              |  status =1 Set success    |
|            |   status =0 Set fail    |

### `85` Set the Active of the EN Pin

| Command    | Answer                    | Comment |
|------------|---------------------------|---------|
| `002#8500` | `can0  002   [3]  85 01 88` |
| **Where**  |                           |
|Byte2: `00` | | `00` - Active Low (L), `01` - Active High (H), `02` - Active Always (Hold)|
|  | Byte1: `85` (Indicates the response is for the 85 command) |
| |Byte2: `01` (Status) | Status = `1` Set success, Status = `0` Set fail |
| |Byte3: `88` (CRC, checksum for the response data) | |

### `86` Set the Direction of Motor Rotation

| Command    | Answer                    | Comment |
|------------|---------------------------|---------|
| `002#8600` | `can0  002   [3]  86 01 89` |
| **Where**  |                           |
|  Byte2: `01` (Direction) | |`00` - CW (Clockwise), `01` - CCW (Counterclockwise) |
|            | Byte2: `01` (Status) | Status = `1` Set success, Status = `0` Set fail |

### `87` Set Auto Turn Off the Screen Function

| Command    | Answer                    | Comment |
|------------|---------------------------|---------|
| `002#8700` | `can0  002   [3]  87 01 8A` |
| **Where**  |                           |
|Byte2: `01` (Enable)| |  enable = `01` enabled, enable = `00` disabled|
|            | Byte2: `01` (Status) | Status = `1` Set success, Status = `0` Set fail |

If set to Enable, the screen will automatically turn off after about 15 seconds, and any button can wake up the screen again

### `88` Set the Motor Shaft Locked-Rotor Protection Function

| Command    | Answer                    | Comment |
|------------|---------------------------|---------|
| `002#8800` | `can0  002   [3]  88 01 8B` |
| **Where**  |                           |
| Byte2: `01` (Enable) | | `01` enabled protection, `00` disabled protection |
|            | Byte2: `01` (Status) | Status = `1` Set success, Status = `0` Set fail |

Note: You can release the protection status by pressing the Enter button or sending the serial command.

### `89` Set the Subdivision Interpolation Function

| Command    | Answer                    | Comment |
|------------|---------------------------|---------|
| `002#8900` | `can0  002   [3]  89 01 8C` |
| **Where**  |                           |
| Byte2: `00` | | `00` - Disabled Interpolation Function, `01` - Enabled Interpolation Function |
|            | Byte2: `01` (Status) | Status = `1` Set success, Status = `0` Set fail |

### `8A` Set the CAN Bit Rate

| Command    | Answer                    | Comment |
|------------|---------------------------|---------|
| `002#8A00` | `can0  002   [3]  8A 01 8D` |
| **Where**  |                           |
| Byte2: `00` (Bit Rate) | | `00` - 125K, `01` - 250K, `02` - 500K |
|            | Byte2: `01` (Status) | Status = `1` Set success, Status = `0` Set fail |

### `8B` Set the CAN ID

| Command    | Answer                    | Comment |
|------------|---------------------------|---------|
| `002#8B00` | `can0  002   [3]  8B 01 8E` |
| **Where**  |                           |
| Byte2: `00` (CAN ID) | | CAN ID = `00` to `7FF` (`0` is the broadcast address) |
|            | Byte2: `01` (Status) | Status = `1` Set success, Status = `0` Set fail |

**Notes:**
1. The new CAN ID will be displayed on the screen under the CanID option.
2. CAN ID `0` is used as the broadcast address.

### `8C` Set the Slave Respond

| Command    | Answer                    | Comment |
|------------|---------------------------|---------|
| `002#8C00` | `can0  002   [3]  8C 01 8F` |
| **Where**  |                           |
| Byte2: `00` (Enable) | | `00` - Disable respond, `01` - Enable respond |
|            | Byte2: `01` (Status) | Status = `1` Set success, Status = `0` Set fail |

**Note:**  
- If the response is disabled, you can still query the running status of the motor using the `F1` command.

### `8D` Set the Group ID

| Command    | Answer                    | Comment |
|------------|---------------------------|---------|
| `002#8D00` | `can0  002   [3]  8D 01 90` |
| **Where**  |                           |
| Byte2: `00` (Group ID) | | `01` - `7FF` (Group ID range) |
|            | Byte2: `01` (Status) | Status = `1` Set success, Status = `0` Set fail |

**Example Setup:**

| Motor   | Broadcast ID | Slave ID | Group ID |
|---------|--------------|----------|----------|
| Motor 1 | `0`          | `1`      | `0x50`   |
| Motor 2 | `0`          | `2`      | `0x50`   |
| Motor 3 | `0`          | `3`      | `0x50`   |
| Motor 4 | `0`          | `4`      | `0x51`   |
| Motor 5 | `0`          | `5`      | `0x51`   |
| Motor 6 | `0`          | `6`      | `0x51`   |

**Example Commands:**

| Command                                | Description                     |
|----------------------------------------|---------------------------------|
| `01 FD 01 2C 64 00 00 0C 80 1B`        | Motor 1 will rotate one turn.   |
| `00 FD 01 2C 64 00 00 0C 80 1A`        | Motors 1-6 will rotate one turn.|
| `50 FD 01 2C 64 00 00 0C 80 6A`        | Motors 1-3 will rotate one turn.|
| `51 FD 01 2C 64 00 00 0C 80 6B`        | Motors 4-6 will rotate one turn.|

### `90` Set the Parameter of Home

| Command    | Answer                    | Comment |
|------------|---------------------------|---------|
| `002#90000000` | `can0  002   [3]  90 00 92` |
| **Where**  |                           |
| Byte2: `00` (homeTrig) | | `0` - Low, `1` - High |
| Byte3: `00` (homeDir)  | | `0` - CW, `1` - CCW |
| Byte4: `00` (homeSpeed) | | `0~3000` RPM (Speed of going home) |
| |Byte2: `01` (Status) | `0` - Set fail, `1` - Set success |

### `91` Go Home

| Command    | Answer                    | Comment |
|------------|---------------------------|---------|
| `002#91`   | `can0  002   [3]  91 01 94` |
| **Where**  |                           |
| | Byte2: `00` (Status)  | `0` - Go home fail, `1` - Go home start, `2` - Go home success |

### `92` Set Current Axis to Zero

| Command    | Answer                    | Comment |
|------------|---------------------------|---------|
| `002#92`   | `can0  002   [3]  92 01 95` |
| **Where**  |                           |
| |Byte2: `01` (Status) | `0` - Set fail, `1` - Set success |

### `3F` Restore the Default Parameter

| Command    | Answer                    | Comment |
|------------|---------------------------|---------|
| `002#3F`   | `can0  002   [3]  3F 01 42` |
| **Where**  |                           |
|| Byte2: `01` (Status)  | `0` - Restore fail, `1` - Restore success |

**Notes:**
- After restoring the parameters, the system will reboot and you will need to calibrate the motor.
- Press the “Next” key, then power on the motor to restore the default parameters.

### `F1` Query the Motor Status

| Command    | Answer                    | Comment |
|------------|---------------------------|---------|
| `002#F1`    | `can0  01   [2]  F1 01`   |         |
| **Where**  |                           |
| | Byte2: `01` (Status) | `0` - Query fail, `1` - Motor stop, `2` - Motor speed up, `3` - Motor speed down, `4` - Motor full speed, `5` - Motor is homing |

### `F3` Enable/Disable the Motor

| Command    | Answer                    | Comment |
|------------|---------------------------|---------|
| `002#F301` | `can0  01   [3]  F3 01 01` |         |
| **Where**  |                           |
| Byte2: `01` (Enable) | | `00` - Disable, `01` - Enable |
| | Byte2: `01` (Status) | `0` - Set fail, `1` - Set success |


### `F6` Run the motor in speed mode

| Command         | Answer                    | Comment |
|-----------------|---------------------------|---------|
| `002#F6014002`  | `can0  01   [3]  F6 01 01` | the motor rotates forward at acc=2, speed=320RPM        |
| `002#F6814002`  | `can0  01   [3]  F6 81 02` | the motor reverses at acc=2, speed=320RPM        |
| **Where**       |                           |
| Byte2 higher bit: `01` (Direction) | | `0x` - CW (Clockwise), `8x` - CCW (Counter-Clockwise) |
| Byte2 lower+ Byte3: `01 40` (Speed) | | `Speed = 40 + (Lower 4 bits of Byte2)` (Range: 0-3000 RPM) |
| Byte4: `02` (Acceleration) | | `0-255` (Acceleration value) |
| | Byte2: `01` (Status) | `status = 1` run success, `status = 0` run fail. |


### `F6` Stop the Motor in Speed Mode

| Command         | Answer                    | Comment |
|-----------------|---------------------------|---------|
| `002#F6000002`  | `can0  01   [3]  F6 00 02` | Decelerate and stop the motor slowly with acc=2 |
| `002#F6000000`  | `can0  01   [3]  F6 00 00` | Immediate stop with acc=0 |
| **Where**       |                           |
| Byte2: `00` (Direction) | | `00` - No direction change, stops motor |
| Byte3: `00` (Speed) | | `00` - Speed is not relevant for stopping |
| Byte4: `02` (Acceleration) | | `0-255` (Acceleration value for deceleration) |
| | Byte2: `01` (Status) | `status = 0` stop the motor fail, `status = 1` start to stop the motor, `status = 2` stop the motor success |

Notes
- If the motor is running faster than 1000 RPM, it is not advisable to use an immediate stop as it may cause mechanical issues. Use deceleration (`acc` ≠ 0) for safer stopping.

### `FF` Save/Clear the Parameter in Speed Mode

| Command         | Answer                    | Comment |
|-----------------|---------------------------|---------|
| `002#FF00C8`    | `can0  01   [3]  FF C8 01` | Save parameters |
| `002#FF00CA`    | `can0  01   [3]  FF CA 01` | Clear parameters |
| **Where**       |                           |
| Byte2: `C8` (State) | | `C8` - Save parameters, `CA` - Clear parameters |
| | Byte2: `01` (Status) | `status = 1` success, `status = 0` fail |

Notes
- After saving or clearing parameters, the motor can rotate clockwise or counterclockwise at a constant speed when powered on.

### `FD` Run the Motor in Position Mode1

| Command          | Answer                    | Comment |
|------------------|---------------------------|---------|
| `002#FD01400200FA00` | `can0  002   [3]  FD 00 FF` | The motor rotates 20 times forward with acc=2, speed=320RPM (16 subdivisions) |
| `002#FD81400200FA00` | `can0  002   [3]  FD 00 FF` | The motor rotates 20 times reverse with acc=2, speed=320RPM (16 subdivisions) |
| **Where**        |                           |
| Byte2 higher bit: `01` (Direction) | | `0x` - CW (Clockwise), `8x` - CCW (Counter-Clockwise) |
| Byte3 lower + Byte4: `40 02` (Speed) | | `Speed = 40 + (Lower 4 bits of Byte2)` (Range: 0-3000 RPM) |
| Byte5-7: `00 FA 00` (Pulses) | | `Pulses = 0x00FA00` (Number of steps for the motor to run) |
| | Byte2 : `00` Status | status = 0 run fail, status = 1 run starting…., status = 2 run complete.|

### `FD` Stop the Motor in Position Mode1

| Command          | Answer                    | Comment |
|------------------|---------------------------|---------|
| `002#FD000004000000` | `can0  01   [3]  FD 00 01` | The motor stops with deceleration, acc=4 |
| `002#FD000000000000` | `can0  01   [3]  FD 00 02` | The motor stops immediately |
| **Where**        |                           |
| Byte2 higher bit: `00` (Direction) | | Direction is not used for stopping |
| Byte3 lower + Byte4: `00 00` (Speed) | | Speed is not used for stopping |
| Byte5-7: `04 00 00` (Acceleration) | | `acc = 4` for slow deceleration, `acc = 0` for immediate stop |
| | Byte2 : `00` Status | status = 0 stop the motor fail, status = 1 stop the motor starting…, status = 2 stop the motor complete|

### `F4` Run the Motor in Position Mode2

| Command                  | Answer                    | Comment |
|--------------------------|---------------------------|---------|
| `002#F402580200400091`   | `can0  01   [3]  F4 01 01` | The motor moves relative to the axis, speed = 600 RPM, acc = 2 |
| `002#F40258FF00C00009`   | `can0  01   [3]  F4 02 01` | The motor moves relative to the axis in the reverse direction, speed = 600 RPM, acc = 2 |
| **Where**                |                           |
| Byte2: `02` (Speed)      |                           | `Speed = 02 * 100 + 58` (RPM, Range: 0-3000) |
| Byte3: `58` (Speed)      |                           | `Speed = 58 + (Byte2 * 100)` (RPM, Range: 0-3000) |
| Byte4: `02` (Acceleration) |                           | `0-255` (Acceleration value) |
| Byte5-7: `00 40 00` or `FF C0 00` (Relative Axis) | | `Relative axis value` (24-bit signed integer) |
| |Byte2: `01` (Status)              | `0` - Run fail, `1` - Run starting, `2` - Run complete |

Notes
- The axis value can be read by command "31".
- In this mode, the axis error is about ±15.
- It is suggested to run with 64 subdivisions.

### `FD` Stop the Motor in Position Mode2

| Command                  | Answer                    | Comment |
|--------------------------|---------------------------|---------|
| `002#FD0000000400`     | `can0  01   [3]  FD 02 02` | Stop the motor with deceleration, acc = 4 |
| `002#FD0000000000`     | `can0  01   [3]  FD 02 02` | Stop the motor immediately |
| **Where**                |                           |
| Byte2: `00` (Direction)  |                           | `0` - No change in direction (for stopping) |
| Byte3: `00` (Speed)      |                           | `Speed = 00` (Not relevant for stop command) |
| Byte4: `00` (Acceleration) |                           | `0-255` (Acceleration value, used for deceleration) |
| Byte5-7: `00 00 00` or `00 00 00` (Pulses) | | `Pulses` (Not relevant for stop command) |
| |                           | `02` - Deceleration and stop, `FE` - Immediate stop |
| | Byte2: `02` (Status)         | `0` - Stop fail, `1` - Stop starting, `2` - Stop complete |

Notes
- The stop command can stop the motor slowly with deceleration (`acc ≠ 0`) or immediately (`acc = 0`).
- If the motor is rotating more than 1000 RPM, it is not recommended to stop the motor immediately.

### `F5` Run the Motor in Position Mode3

| Command                  | Answer                    | Comment |
|--------------------------|---------------------------|---------|
| `002#F5025802004000`   | `can0  01   [3]  F5 01 01` | The motor moves to absolute axis 0x4000, speed = 600 RPM, acc = 2 |
| `002#F5025802FFC000`   | `can0  01   [3]  F5 02 02` | The motor moves to absolute axis -0x4000, speed = 600 RPM, acc = 2 |
| **Where**                |                           |
| Byte2: `02` (Speed)      |                           | `Speed = 02 * 100 + 58` (RPM, Range: 0-3000) |
| Byte3: `58` (Speed)      |                           | `Speed = 58 + (Byte2 * 100)` (RPM, Range: 0-3000) |
| Byte4: `02` (Acceleration) |                           | `0-255` (Acceleration value) |
| Byte5-7: `00 40 00` or `FF C0 00` (Absolute Axis) | | `Absolute axis value` (24-bit signed integer) |
| | Byte2: `01` (Status)         | `0` - Run fail, `1` - Run starting, `2` - Run complete |

Notes
- The axis value can be read by command "31".
- In this mode, the axis error is about ±15.
- It is suggested to run with 64 subdivisions.

### `F5` Stop the Motor in Position Mode3

| Command                  | Answer                    | Comment |
|--------------------------|---------------------------|---------|
| `002#F50000040000`     | `can0  01   [3]  F5 01 01` | Stop the motor with deceleration, acc=4 |
| `002#F50000000000`     | `can0  01   [3]  F5 02 02` | Stop the motor immediately, acc=0 |
| **Where**                |                           |
| Byte2: `00` (Speed)      |                           | Not used in the stop command |
| Byte3: `00` (Acceleration) |                           | `0-255` (Deceleration value) |
| Byte4: `00` (Absolute Axis) |                           | Not used in the stop command |
| Byte5-7: `00 00 00` (Unused) |                           | Not used in the stop command |
| | Byte2: `01` (Status)         | `0` - Stop fail, `1` - Stop starting, `2` - Stop complete |

Notes
- If the motor is rotating more than 1000 RPM, it is not a good idea to stop the motor immediately.
- The uplink frame provides status feedback on the stop command.

