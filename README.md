# klipper orangepi 4
 
first github expierence so  formatting isnt great, this purely created as reference for my future self, but shared as it may help someone.
 
####################
Install via KIAUH 
##################
Use KIAUH to install klipper, moonraker and Fluidd as normal https://github.com/th33xitus/KIAUH

following prompts there isnt anything different except the following changes.

###############
TMC2209
###############
Intial setup for SKR1.4 is set to Tmc2208, need to change to Tmc2209

to be inserted after each stepper inside printer.cfg:

#
[tmc2209 extruder]
uart_pin: P1.4 # every different stepper has a different uart pin, although by changing resistors on each stepstick its possible to address 4 drivers on one pin
interpolate: true
run_current: 0.5
hold_current: 0.25
sense_resistor: 0.110
stealthchop_threshold: 0 # 0 to force  SPREADCYCLE / SET TO 999999 TO force STEALTHCHOP/ set 100 to switch from STEALTHCHOP TO SPREADCYCLE AT SPEEDS ABOVE 100
#



###############################
BLTOUCH with an AC heated bed
################################
Bltouch uses a Hall-effect sesnor which is prone to noisey signals at the influence of an AC heated bed, so use HOMING_HEATERS on Z axis to
reduce interference at the expense of the bed cooling whilst doing bed mesh, so keep to a small sample size 3x3 opposed to 11x11, and use fast travel speeds along x,y axis.

to be inserted in printer.cfg:
#
[homing_heaters]
steppers: stepper_z
#   
heaters: heater_bed
# 
   
POSSIBLEY LOOK AT HOMING OVERRIDE INSTEAD OF SAFE HOMING TO LIFT Z TO STOP BLTOUCH COLLISIONS



################
Display on CR10s 
#################
on original board the display uses 2 ribbon cabbles on EX1 & EX2 but to get it to work with SKR1.4 use a single cable from EX1 to EX3 on display 
an Ender3

to be set in printer.cfg:
##
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
ORANGEPI-4 CPU TEMPERATURES
##########################
 the below values work for orangepi4, I have orange pi zero lts commands elsewhere will add them here soon.
to be inserted in printer.cfg:
 
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
resonance using adxl3455
##########################

RESONANCE MEASUREMENT USING ORANGE PI 4

Orange PI differs from Raspberry pi in GPIO pins and SPI names so changes need to be made.


https://neonaut.neocities.org/blog/2018/orange-pi-i2c-spi-setup.html
Enabling SPI
Enable the hardware through config:
sudo armbian-config
switch on SPI in System>Hardware>SPI-spidev   & save 

Update /etc/modules:
sudo nano /etc/modules

to include the following line:
spi-dev

Update /boot/armbianEnv.txt :
sudo nano /boot/armbianEnv.txt

to include the following:
param_spidev_spi_bus=1           #origianl is 0 but the resonance measurement.py module expects 1 so easier to fix here
param_spidev_max_freq=100000000


The commands to verify the SPI bus should return output like:
$ ls /dev/spi*
/dev/spidev1.0
$ ls -l /dev/spi*
crw——- 1 root root 153, 0 Aug 25 11:05 /dev/spidev1.0

$ lsmod | grep -i spi
spidev 20480 1

Connected pins on ORANGEPI4:

reference to ORANGEPI4 MANUAL PAGE 75

http://www.orangepi.org/downloadresources/ 
SCROLL DOWN TO ORANGEPI 4 MANUAL , SELECT GOOGLEDRIVE

 orangepi4 ---------------ADXL345
pin 2 5V                  vcc
pin 9 GND                 gnd             
pin 19 SPI1_TXD           sda
pin 21 SPI1_RXD           sdo
pin 23 SPI1_CLK           scl
pin 23 SPI1_CS            cs

ADXL345 markings may be slightly different, tx on pi should go to rx on adxl & rx on pi should go tx on adxl the rest should be straight connections

https://www.klipper3d.org/Measuring_Resonances.html
other changes from klipper website which is for a raspberrypi include:
 
 Armbian will only work with Python3, unless we find an older version we have to use this workaround:
sudo apt install python3-numpy python3-matplotlib
 
 we need to edit ~/klipper/scripts :
nano ~/klipper/scripts/cal.py
 
 change:
 #!/usr/bin/env python2
 
 to: 
 #!/usr/bin/env python3
 
 (this will allow numpy matp-lotlib etc to work ,there may be other *.py *.sh apps that also need to be setup to run python3)

 to be inserted into printer.cfg:
#
[mcu rpi]
serial: /tmp/klipper_host_mcu


[resonance_tester]
accel_chip: adxl345
probe_points:
    150,150,20  # an example

[adxl345]
cs_pin:rpi:None
spi_speed: 5000000
spi_bus:spidev1.0


########################
WEBCAM section
#######################
WEBCAM /dev/video* 
seems to bounce around in number from 0,2,3,4 maybe issue with usb and mjpegstreamer?? used some other streamer on octocitrico?

 
to be inserted/changed in webcam.txt:
camera="usb"

camera_usb_options="-d /dev/video0 -r 640x480 -f 10 -rot 270"cameravideo* to /dev/video0 
 then "-d /dev/video0 -r 640x480 1-f 10 -rot 270"cameravideo* to /dev/video1
 "-d /dev/video0 -r 640x480 2-f 10 -rot 270"cameravideo* to /dev/video2
 

create copies of webcam.txt and rename to webcam1.txt,webcam2.txt,webcam3.txt .... etc
 
 cd ~/klipper_config/
 
 cp webcam.txt webcam1.txt
 cp webcam.txt webcam2.txt
 cp webcam.txt webcam3.txt
 
 use fluidd to edit each webcam*.txt
 
 change camera="auto" to camer="usb"
 
 uncomment
 camera_usb_options="-d /dev/video0 -r 640x480 -f 10 -rot 270"
 
 and starting with webcam.txt increment  /dev/video* to /dev/video0 
 then  edit webcam1.txt to  /dev/video* to /dev/video1
 then  edit webcam2.txt to  /dev/video* to /dev/video2
 then  edit webcam3.txt to  /dev/video* to /dev/video3
 
 that seems to cope with the random assigning of numbers to webcam, atleast untill I get more time to look at the problem.
 
 
 
 
##################
 end of changes
 #################
things to do:
 
 sort out webcam t edit o a stable ntod wiringop or op.gpio to enable relay to shut off printer and power switch localy to turn printer on for maintenance
 
 sort out add switch tton pitoded files :

 sort out 
 
 that seems to cope with the random assigning of numbers to webcam, atleast untill I get more time to look at the problem.
printer.cfg 
3
bltouch.cfg (had to include Z homing offsets within printer.cfg as SAVE_CONFIG only writes updates if it can find settings ebcam.txt
kiauh_macros.cfg
extruder.cfg (had to include extruder within printer.cfg as SAVE_CONFIG only writes updates if it can find settings inside printer.cfg)


