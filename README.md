## 0. Репозиторий
Данная конфигурация может не подойти к любому ULTI, однако имеет структурированый вид, а потому удобна в качестве темплейта.  
Eсть предложения? Пишите! Тусовка на [отдельном канале в Telegram](https://t.me/ulti_klipper).
- ./macros - склад макросов, подгружается вся директория
- printer-basic.cfg - базовый медленный вариант конфига. На текущий момент актуален он.
- printer.cfg - временно не актуальный конфиг. Будет обновлен после калибровки с акселлерометром.

## 1. Установка FLUIDD
- Скачать [образ](https://github.com/cadriel/FluiddPI) и залить его на SD при помощи [balenaEtcher](https://www.balena.io/etcher/)
- Настроить WiFi в fluiddpi-wpa-supplicant.txt
- Воткнуть SD карточку в RPi, включить, после загрузки - посмотреть IP подключенного устройства в админке роута и подключиться по ssh (https://www.putty.org)/
>Login: pi Password: raspberry
- Обновиться
>sudo apt-get update

>sudo apt-get upgrade

>sudo raspi-config [>Update]
- Настроить RPi через raspi-config (Hostname, Password, Timezone, etc)

>sudo echo "pi ALL=(ALL:ALL) NOPASSWD:ALL" >> /etc/sudoers


## 2. Установка/обновление Klipper
>cd ~/klipper  
>git pull  
>~/klipper/scripts/install-octopi.sh `(даже с Fluidd)`   
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

- Прошивка SKR напрямую потребует [другой бутлоадер для LPC1768](http://smoothieware.org/flashing-the-bootloader)  
>make flash FLASH_DEVICE=/dev/ttyACM0    
- Проще прошиться через SD. [Удлинитель](https://habr.com/ru/post/206394/) поможет!  
Закидываем на SD файл **klipper.bin**, переименовав его в **firmware.bin**. Включаем SKR, проверяем.  
>ls -lrt /dev/serial/by-id/  

lrwxrwxrwx 1 root root 13 Jan 17 23:20 usb-**Klipper**_lpc1768_1270010FC81848AFFABA185DC32000F5-if00 -> ../../ttyACM0  
>sudo service klipper start  

## 3. Конфигурирование
- Открываем printer.cfg, корректируем в нем раздел MCU - ставим свой/dev/serial/by-id/...
- Заходим на FLUIDD
- Restart

## 4. Базовая калибровка
- Калибруем PID экструдера. 
>PID_CALIBRATE HEATER=extruder TARGET=230
- Калибруем [концевики](https://www.klipper3d.org/Endstop_Phase.html)
>ENDSTOP_PHASE_CALIBRATE 
- Выравниваем [стол](https://www.klipper3d.org/Manual_Level.html)
>BED_SCREWS_ADJUST
- Еще раз проверяем концевики
>ENDSTOP_PHASE_CALIBRATE 
- Следующая настройка под вопросом, если используется калибровка через щуп - универсальнее использовать Z_OFFSET для различных материалов.
>Z_ENDSTOP_CALIBRATE
- Проверяем [Extruder rotation distance](https://www.klipper3d.org/Rotation_Distance.html)
- Печатаем пустотелый кубик для калибровки Flow



## 5. Guide in progress - Pressure Advance
- https://www.klipper3d.org/Pressure_Advance.html
- PA with reduced retraction (1.75mm and 25mm/s)
- Temp Tower 
>TUNING_TOWER COMMAND=SET_PRESSURE_ADVANCE PARAMETER=ADVANCE START=0.3 FACTOR=.002
- PA with lower FACTOR and short range (168 per tower, so 0.3-0.636)
- Fine retraction calibration (tower)
- PA check with regular printing settings (maybe standart settings - 100 or 80 mm\s, layer 0.1 or 0.2)
- Benchy and Cube =)

# TODO
- Fan tower
- https://www.klipper3d.org/Resonance_Compensation.html
- Проверить на тесте https://www.klipper3d.org/skew_correction.html
- Max smoothing (from accelerometer log)
- Settings per material (retract, z_offset)
- Accel tower
- Speed tower
- Square Corner Velocity tower
- Fine-tuning Input Shaping (maybe with PA)

# Useful notes
- Firmware retraction - no wipe, not sure
- https://www.klipper3d.org/Slicers.html
- Temp tower (to verify, bug)
>TUNING_TOWER COMMAND="SET_HEATER_TEMPERATURE HEATER=extruder" PARAMETER=TARGET START=250 FACTOR=-5 BAND=10
