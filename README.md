## 0. Репозиторий
Данная конфигурация может не подойти к любому ULTI, однако имеет структурированый вид, а потому удобна в качестве темплейта.  
Eсть предложения? Пишите! Тусовка на [отдельном канале в Telegram](https://t.me/ulti_klipper).
- ./macros - склад макросов, подгружается вся директория. Положить на RPi рядом с конфигом.  
### Если у вас Rocker Feeder:
- printer-rocker_feeder-classic.cfg - Классическая конфигурация, оригинальный Rocker Feeder.
- printer-rocker_feeder-performance.cfg - Версия без интерполейта и без снижения тока на hold (согласно последним данным, может влиять на точность), оригинальный Rocker Feeder. Проверьте, что не перегреваются моторы.
### Если у вас BMG:
- printer-BMG-interpolate.cfg - Конфигурация с интерполейтом, но без снижения тока на hold, BMG от Triangle.
- printer-BMG-performance.cfg - Версия без интерполейта и без снижения тока на hold, BMG от Triangle.
### Если у вас BMG-Mini:
- Ждите, когда моя ленивая жопа его уже поставит.


## 1. Установка FLUIDD
- Скачать [образ](https://github.com/cadriel/FluiddPI) и залить его на SD при помощи [balenaEtcher](https://www.balena.io/etcher/)
- Настроить WiFi в **fluiddpi-wpa-supplicant.txt**
- Воткнуть SD карточку в RPi, включить, после загрузки - посмотреть IP подключенного устройства в админке роутера и подключиться по [ssh](https://www.putty.org)  
<span style="color:red">**Login: pi Password: raspberry**</span>
- Обновиться  


```sudo apt-get update ```   
```sudo apt-get upgrade  ```  
```sudo raspi-config [>Update]  ```  
- Настроить RPi через raspi-config (Hostname, Password, Timezone, etc)  

```sudo echo "pi ALL=(ALL:ALL) NOPASSWD:ALL" >> /etc/sudoers```  


## 2. Установка/обновление Klipper
```cd ~/klipper ```   
```git pull ```  
Даже если используем Fluidd, а не Octo,  
```~/klipper/scripts/install-octopi.sh```    
```ls -lrt /dev/serial/by-id/```    
Должны увидеть, что плата SKR подключена.  
>lrwxrwxrwx 1 root root 13 Jan 17 22:12 usb-marlinfw.org_Marlin_USB_Device_0F017012AF4818C85D18BAFAF50020C3-if00 -> ../../ttyACM0  

```make menuconfig```  

[<span style="color:red">**x**</span>] Enable extra low-level configuration options  
    Micro-controller Architecture (LPC176x (Smoothieboard))  --->  
    Processor model (lpc1768 (100 MHz))  --->  
[<span style="color:red">**x**</span>] Target board uses Smoothieware bootloader  
[<span style="color:red">**x**</span>] Use USB for communication (instead of serial)  
    USB ids  --->  
[ ] Specify a custom step pulse duration  
()  GPIO pins to set at micro-controller startup  

```make clean```  
```make```  
```sudo service klipper stop```  

- Прошивка SKR напрямую потребует [другой бутлоадер для LPC1768](http://smoothieware.org/flashing-the-bootloader), но позволяет не открывать подвал.  

```make flash FLASH_DEVICE=/dev/ttyACM0```  
- Проще прошиться через SD. [Удлинитель](https://habr.com/ru/post/206394/) поможет!  
Закидываем на SD файл **klipper.bin**, переименовав его в **firmware.bin**. Включаем SKR, проверяем и запоминаем свой id.  

```ls -lrt /dev/serial/by-id/```  
>lrwxrwxrwx 1 root root 13 Jan 17 23:20 usb-**Klipper**_lpc1768_1270010FC81848AFFABA185DC32000F5-if00 -> ../../ttyACM0  

```sudo service klipper start```  


## 3. Конфигурирование
- Открываем подходящий под конфигурация конфиг (.cfg), корректируем в нем раздел "# Board" (MCU) - ставим свой /dev/serial/by-id/usb-Klipper_lpc1768_**ID**
- Загружаем конфиг в директорию /home/pi/klipper_config/ с именем printer.cfg  
**Так же загружаем директорию macros**
- Заходим браузером на FLUIDD
- Отправляем в консоль FLUIDD **RESTART**


## 4. Базовая калибровка
Команды отправляются из интерфейса Octoprint/Fluidd в консоль как GCODE.
- Калибруем PID экструдера  
```PID_CALIBRATE HEATER=extruder TARGET=230```
- Калибруем PID стола  
```PID_CALIBRATE HEATER=heater_bed TARGET=110```
- Калибруем [концевики](https://www.klipper3d.org/Endstop_Phase.html)  
```ENDSTOP_PHASE_CALIBRATE ```
- Выравниваем [стол](https://www.klipper3d.org/Manual_Level.html)  
```BED_SCREWS_ADJUST```
- Еще раз проверяем концевики, стол должен останавливаться в Z=0.00  
```ENDSTOP_PHASE_CALIBRATE``` 
- Следующая настройка под вопросом, если используется калибровка через щуп - универсальнее использовать Z_OFFSET.  
Это позволит менять его для различного типа филамента через скрипт запуска, например так - START_PRINT BED_TEMP={first_layer_bed_temperature[0]} EXTRUDER_TEMP={first_layer_temperature[0]} Z_GCODE_OFFSET=**-0.03** 
```Z_ENDSTOP_CALIBRATE```
- Проверяем [Extruder rotation distance](https://www.klipper3d.org/Rotation_Distance.html)
- Печатаем пустотелый кубик для калибровки потока и устанавливаем корректное значение Flow в слайсере, даже если калибровали его до этого в Marlin
- Печатаем температурную башню если неизвестна точная температура


## 5.1 [Pressure Advance](https://www.klipper3d.org/Pressure_Advance.html)
**Прежде чем приступить, убедитесь что температура и поток откалиброваны правильно**  

На мой взгляд калибровка должна происходить примерно с тем же ретрактом, что будет использоваться при печати - это вопреки официальной инструкции, что рекомендует использовать "normal retraction amount", а после калибровки - "small value". Такой подход позволит получить более показательные и приближенные к реальным условиям печати результаты калибровки.  
По шагам:
- Калибровка PA с уменьшенным ретрактом (мои значения - 1.75mm длина и 25mm/s скорость ретракта). Шов Aligned по углу позволит легко найти точную высоту.
По углам определяем оптимальную высоту и по формуле (pressure_advance = стартовое_значение + измерянная_высота_в_мм * инкремент_фактор) высчитываем примерно нужное значение.  

```SET_VELOCITY_LIMIT SQUARE_CORNER_VELOCITY=1 ACCEL=500  ```
```TUNING_TOWER COMMAND=SET_PRESSURE_ADVANCE PARAMETER=ADVANCE START=0 FACTOR=.020 ``` 

У меня оптимальное значение в первоначальном тесте было на высоте 25мм, соответственно 0 + 25 * 0.020 = 0.5  
Устанавливаем полученное значение в прошивке в блоке "# Pressure Advance"  
Если у вас ABS и ULTI стандартной конфигурации, а так же вы используете мой конфиг - скорее всего первоначальную калибровку можно пропустить и приступить сразу к следующему шагу.
- Точная калибровка PA. Настройки аналогичны предудыщему шагу, но шов рекомендую Rear - позволит ориентироваться не только по углам, но и по шву.  
Сразу предупреждаю, что данная калибровка требовательна к зрению, так какесли вы находитесь в оптимальном диапазоне разница будет незначительна. Посветить ярким фонариком под острым углом поможет найти оптимальную высоту.  
Так как оптимальное первоначальное значение было у меня 0.5 (и у вас, скорее всего, тоже рядом), калибровка будет происходить в диапазоне от 0.4 до 0.75  

```SET_VELOCITY_LIMIT SQUARE_CORNER_VELOCITY=1 ACCEL=500 ``` 
```TUNING_TOWER COMMAND=SET_PRESSURE_ADVANCE PARAMETER=ADVANCE START=0.4 FACTOR=.007 ``` 

Оптимальная высота у меня составила 26.77мм, соответственно, pressure_advance = <start> + <measured_height> * <factor> = 0.4 + 26.77 * 0.007

- Калибровка square_corner_velocity через все тот же Tung Tower, используя модель ringing_tower.stl (перед тестом нужно завысить лимит square_corner_velocity в конфиге).  

Я не заметил никаких позитивных изменений в качестве при повышении SCV, ровно как и повышения скорости печати. После 30 появилось эхо. Если вдруг у вас иные результаты - пожалуйста сообщите. Похоже, что значения в **5** вполне достаточно при печати 100mm/s и 3000mm^2/s. Если есть желание, то - 
```TUNING_TOWER COMMAND=SET_VELOCITY_LIMIT PARAMETER=SQUARE_CORNER_VELOCITY START=5 FACTOR=1```

## 5.2 Альтернативная калибровка Pressure Advance
Выполнить печать STL/PA_calibratiob_v*.stl в режиме калибровочной башни. Слайсинг - с обычными, часто используемыми параметрами печати, а печать без изменений SCV и ускорений.
```TUNING_TOWER COMMAND=SET_PRESSURE_ADVANCE PARAMETER=ADVANCE START=0.4 FACTOR=.007 ``` 
Данная модель покажет слишком раннюю или слишком позднюю, либо недостаточную или чрезмерную компенсацию. Стоит обратить внимание не только на углы, но и на окончание периметра в прорези.
Версии различаются по ширине линии. 

# Ура! Первоначальная калибровка завершена, теперь можно печатать кораблики и кубики!


## TODO
- firmware_retraction + fine retraction calibration (tower)
- Not sure about correct retraction/deretraction acceleration - max_extrude_only_accel  
- Fan tower
- Accel tower
- Speed tower
- Написать раздел по https://www.klipper3d.org/Resonance_Compensation.html
- Проверить https://www.klipper3d.org/skew_correction.html
- Max smoothing (from accelerometer log) - автоматизировать подбор скриптом.
- Settings per material (retract, z_offset) - перенесено в слайсер
- Fine-tuning Input Shaping (maybe with PA)
- **Релизнуть конфиг PrusaSlicer**


## Useful notes
- https://www.klipper3d.org/Slicers.html


### [**На ZAV/UNI/BOLT**](https://yoomoney.ru/to/4100116514086369?from=auth&contextId=UACB_CAC_803ecb35-61a0-4692-bb9c-d2a35735c8fe)

