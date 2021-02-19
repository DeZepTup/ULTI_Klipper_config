## Репозиторий
Данная конфигурация может не подойти к любому ULTI, однако имеет структурированый вид, а потому удобна в качестве темплейта.
Eсть предложения? Пишите!
- ./macros - склад макросов, подгружается вся директория
- printer-basic.cfg - базовый медленный вариант конфига. На текущий момент актуален он.
- printer.cfg - временно не актуальный конфиг. Будет обновлен после калибровки с акселлерометром.

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

>sudo echo "pi ALL=(ALL:ALL) NOPASSWD:ALL" >> /etc/sudoers


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

Следующая настройка под вопросом, если используется калибровка через щуп - универсальнее использовать Z_OFFSET для различных материалов.

>Z_ENDSTOP_CALIBRATE


# Testing
- https://www.klipper3d.org/Pressure_Advance.html
- PA with reduced retraction (1.75mm and 25mm/s) - Temp Tower - PA with lower FACTOR (maybe standart settings - 100 or 80 mm\s, layer 0.1 or 0.2)

# TODO
- https://www.klipper3d.org/Resonance_Compensation.html
- Проверить на тесте https://www.klipper3d.org/skew_correction.html
- Settings per material (retract, z_offset)

# Useful notes
- Firmware retraction - no wipe, not sure
- https://www.klipper3d.org/Slicers.html
- Temp tower
>TUNING_TOWER COMMAND="SET_HEATER_TEMPERATURE HEATER=extruder" PARAMETER=TARGET START=250 FACTOR=-5 BAND=10


# Изменения по версиям Klipper
- https://www.klipper3d.org/Rotation_Distance.html