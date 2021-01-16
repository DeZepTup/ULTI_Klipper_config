## Установка FLUIDD
- Скачать образ (https://github.com/cadriel/FluiddPI) и залить его на SD при помощи balenaEtcher (https://www.balena.io/etcher/)
- Настроить WiFi в fluiddpi-wpa-supplicant.txt
- Воткнуть SD карточку в RPi, включить, после загрузки - посмотреть IP подключенного устройства в админке роута и подключиться по ssh (https://www.putty.org)/
Login: pi Password: raspberry
- Обновиться
sudo apt-get update
sudo apt-get upgrade
sudo raspi-config
>Update
- Настроить RPi через raspi-config (Hostname, Password, Timezone, etc)

## Установка/обновление Klipper
git clone https://github.com/KevinOConnor/klipper
cd ~/klipper/
make menuconfig
Выбрать LPC1768
make
ls /dev/serial/by-id/*
Должно быть /dev/serial/by-id/usb-Klipper_lpc1768_08200012801C3DAF134B975CC02000F5-if00

## Прошивка
sudo service klipper stop
make flash FLASH_DEVICE=/dev/serial/by-id/usb-Klipper_lpc1768_08200012801C3DAF134B975CC02000F5-if00
sudo service klipper start
- Либо через SD