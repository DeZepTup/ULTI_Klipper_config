## Установка FLUIDD
- Скачать образ (https://github.com/cadriel/FluiddPI) и залить его на SD при помощи balenaEtcher (https://www.balena.io/etcher/)
- Настроить WiFi в fluiddpi-wpa-supplicant.txt
- Воткнуть SD карточку в RPi, включить, после загрузки - посмотреть IP подключенного устройства в админке роута и подключиться по ssh (https://www.putty.org)/
>Login: pi Password: raspberry
- Обновиться
>sudo apt-get update

>sudo apt-get upgrade

>sudo raspi-config [>Update]
- Настроить RPi через raspi-config (Hostname, Password, Timezone, etc)

## Установка/обновление Klipper
>cd ~/klipper

>git pull

>~/klipper/scripts/install-octopi.sh (даже с Fluidd)

>ls -lrt /dev/serial/by-id/
lrwxrwxrwx 1 root root 13 Jan 17 22:12 usb-marlinfw.org_Marlin_USB_Device_0F017012AF4818C85D18BAFAF50020C3-if00 -> ../../ttyACM0

>make menuconfig

[x] Enable extra low-level configuration options
    Micro-controller Architecture (LPC176x (Smoothieboard))  --->
    Processor model (lpc1768 (100 MHz))  --->
[x] Target board uses Smoothieware bootloader
[x] Use USB for communication (instead of serial)
    USB ids  --->
[ ] Specify a custom step pulse duration
()  GPIO pins to set at micro-controller startup

>make clean

>make

>sudo service klipper stop

###### http://smoothieware.org/flashing-the-bootloader
>make flash FLASH_DEVICE=/dev/ttyACM0

- Либо через SD. [Удлинитель](https://habr.com/ru/post/206394/) поможет с перепрошивками! 

>ls -lrt /dev/serial/by-id/
lrwxrwxrwx 1 root root 13 Jan 17 23:20 usb-Klipper_lpc1768_1270010FC81848AFFABA185DC32000F5-if00 -> ../../ttyACM0

>sudo service klipper start

## Конфигурирование
- Открываем printer.cfg, корректируем в нем /dev/serial/by-id/
- Заходим на FLUIDD
- Restart

## Калибровка
- PID_CALIBRATE HEATER=extruder TARGET=230
- https://www.klipper3d.org/Endstop_Phase.html
- https://www.klipper3d.org/Manual_Level.html
>ENDSTOP_PHASE_CALIBRATE 

>BED_SCREWS_ADJUST

>ENDSTOP_PHASE_CALIBRATE 

>Z_ENDSTOP_CALIBRATE

>BED_SCREWS_ADJUST

>ENDSTOP_PHASE_CALIBRATE

# To be continued...

- https://www.klipper3d.org/Rotation_Distance.html
- https://www.klipper3d.org/Slicers.html
- https://www.klipper3d.org/Resonance_Compensation.html
- https://www.klipper3d.org/Pressure_Advance.html