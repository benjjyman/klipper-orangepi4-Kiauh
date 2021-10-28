# klipper orangepi 4
 

Install

Use KIAUH to install klipper, moonraker and Fluidd as normal https://github.com/th33xitus/KIAUH

TMC2209

Intial setup for SKR1.4 is set to Tmc2208, need to change to Tmc2209


#############################
[tmc2209 extruder]
uart_pin: P1.4
interpolate: true
run_current: 0.5
hold_current: 0.25
sense_resistor: 0.110
stealthchop_threshold: 0 # 0 to force  SPREADCYCLE / SET TO 999999 TO force STEALTHCHOP/ set 100 to switch from STEALTHCHOP TO SPREADCYCLE AT SPEEDS ABOVE 100
#########################

BLTOUCH 

Bltouch uses a Hall-effect sesnor which is prone to noisey signals at the influence of an AC heated bed, so use HOMING_HEATERS on Z axis to
reduce interference at the expense of the bed cooling whilst doing bed mesh, so keep to a small sample size 3x3 opposed to 11x11, and use fast travel speeds along x,y axis.

#########################

[homing_heaters]
steppers: stepper_z
#   A comma separated list of steppers that should cause heaters to be
#   disabled. The default is to disable heaters for any homing/probing
#   move.
#   Typical example: stepper_z
heaters: heater_bed
#   A comma separated list of heaters to disable during homing/probing
#   moves. The default is to disable all heaters.
#   Typical example: extruder, heater_bed
###########################

############POSSIBLEY LOOK AT HOMING OVERRIDE INSTEAD OF SAFE HOMING TO LIFT Z TO STOP BLTOUCH COLLISIONS

Display on CR10s 

on original board the display uses 2 ribbon cabbles on EX1 & EX2 but to get it to work with SKR1.4 use a single cable from EX1 to EX3 on display 
an Ender3

##########################
[board_pins]
aliases:
    # EXP1 header
    EXP1_1=P1.30, EXP1_3=P1.18, EXP1_5=P1.20, EXP1_7=P1.22, EXP1_9=<GND>,
    EXP1_2=P0.28, EXP1_4=P1.19, EXP1_6=P1.21, EXP1_8=P1.23, EXP1_10=<5V>,
    # EXP2 header
    EXP2_1=P0.17, EXP2_3=P3.26, EXP2_5=P3.25, EXP2_7=P1.31, EXP2_9=<GND>,
    EXP2_2=P0.15, EXP2_4=P0.16, EXP2_6=P0.18, EXP2_8=<RST>, EXP2_10=<NC>
    # Pins EXP2_1, EXP2_6, EXP2_2 are also MISO, MOSI, SCK of bus "ssp0"

[display]
lcd_type: st7920
cs_pin: EXP1_7
sclk_pin: EXP1_6
sid_pin: EXP1_8
encoder_pins: ^EXP1_5, ^EXP1_3
click_pin: ^!EXP1_2

[output_pin beeper]
pin: EXP1_1


##########################

ORANGEPI CPU TEMPERATURES

##########################

[temperature_sensor orange_pi_1]
sensor_type: temperature_host
min_temp: 5
max_temp: 100


[temperature_sensor orange_pi_2]
sensor_type: temperature_host
sensor_path: /sys/class/thermal/thermal_zone1/temp
min_temp: 5
max_temp: 100

##########################


RESONANCE MEASUREMENT USING ORANGE PI 4

Orange PI differs from Raspberry pi in GPIO pins and SPI names so changes need to be made.


https://neonaut.neocities.org/blog/2018/orange-pi-i2c-spi-setup.html
Enabling SPI
Enable the hardware through config:

sudo armbian-config
switch on SPI in System>Hardware>SPI-spidev   & save 

Update /etc/modules 

sudo nano /etc/modules

to include the following line:

spi-dev

Update /boot/armbianEnv.txt 

sudo nano /boot/armbianEnv.txt

to include the following:

param_spidev_spi_bus=1           ###########origianl is 0 but the resonance measurement.py module expects 1 so easier to fix here
param_spidev_max_freq=100000000

You may need to update the device-tree overlay as well. (Info for H5.) [I don’t think I needed this for OP0 which is H2, just the OP PC2] Additionally, if you reset you’ll want to check and make sure the changes are still there.

The commands to verify the SPI bus should return output like:

$ ls /dev/spi*
/dev/spidev0.0
$ ls -l /dev/spi*
crw——- 1 root root 153, 0 Aug 25 11:05 /dev/spidev1.0


$ lsmod | grep -i spi
spidev 20480 0

Connected pins on ORANGEPI4 

ORANGEPI4 MANUAL PAGE 75

http://www.orangepi.org/downloadresources/ 
SCROLL DOWN TO ORANGEPI 4 MANUAL , SELECT GOOGLEDRIVE

pin 2 5V
pin 9 GND
pin 19 SPI1_TXD
pin 21 SPI1_RXD
pin 23 SPI1_CLK
pin 23 SPI1_CS

OTHER END CONNECTED TO ADXL345 AS MARKED

https://www.klipper3d.org/Measuring_Resonances.html
OTHER CHANGES FROM WEBSITE   ARMBIAN WILL ONLY WORK WITH PYTHON3 SO THE CHANGES BELLOW NEED TO BE INCLUDED:

sudo apt install python3-numpy python3-matplotlib

########################

[mcu rpi]
serial: /tmp/klipper_host_mcu


[resonance_tester]
accel_chip: adxl345
probe_points:
    150,150,20  # an example

[adxl345]
cs_pin:rpi:None
#   The SPI enable pin for the sensor. This parameter must be provided.
spi_speed: 5000000
#   The SPI speed (in hz) to use when communicating with the chip.
#   The default is 5000000.
spi_bus:spidev1.0
#spi_software_sclk_pin:SPI0_CLK
#spi_software_mosi_pin:SPI0_RXD
#spi_software_miso_pin:SPI0_TXD
#   See the "common SPI settings" section for a description of the
#   above parameters.
#axes_map: x,y,z
#   The accelerometer axis for each of the printer's x, y, and z axes.
#   This may be useful if the accelerometer is mounted in an
#   orientation that does not match the printer orientation. For
#   example, one could set this to "y,x,z" to swap the x and y axes.
#   It is also possible to negate an axis if the accelerometer
#   direction is reversed (eg, "x,z,-y"). The default is "x,y,z".
#rate: 3200
#   Output data rate for ADXL345. ADXL345 supports the following data
#   rates: 3200, 1600, 800, 400, 200, 100, 50, and 25. Note that it is
#   not recommended to change this rate from the default 3200, and
#   rates below 800 will considerably affect the quality of resonance
#   measurements.

########################
WEBCAM 

WEBCAM /dev/video* 
seems to bounce around in number from 0,2,3,4 maybe issue with usb and mjpegstreamer?? used some other streamer on octocitrico?

camera="usb"

camera_usb_options="-d /dev/video0 -r 640x480 -f 10 -rot 270"


included files :

printer.cfg 
bltouch.cfg (had to include Z homing offsets within printer.cfg as SAVE_CONFIG only writes updates if it can find settings inside printer.cfg)
webcam.txt
kiauh_macros.cfg
extruder.cfg (had to include extruder within printer.cfg as SAVE_CONFIG only writes updates if it can find settings inside printer.cfg)
############insert printer.cfg here

