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
```
