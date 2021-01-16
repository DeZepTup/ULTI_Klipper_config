#### Установка FLUIDD
1. Скачать образ (https://github.com/cadriel/FluiddPI) и залить его на SD при помощи balenaEtcher (https://www.balena.io/etcher/)
2. Настроить WiFi в fluiddpi-wpa-supplicant.txt
3. Воткнуть SD карточку в RPi, включить, после загрузки - посмотреть IP подключенного устройства в админке роута и подключиться по ssh (https://www.putty.org)/
Login: pi Password: raspberry
4. Обновиться
sudo apt-get update
sudo apt-get upgrade
sudo raspi-config
>Update
5. Настроить RPi через raspi-config (Hostname, Password, Timezone, etc)

#### Установка/обновление Klipper



ls /dev/serial/by-id/*

