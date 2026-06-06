# Прошивка OpenWrt
Для великой победы, нам необходима прошивка OpenWrt на роутере и доступ по ssh к нему, ниже пример с гайдом для роутера Xiaomi Router AX3200 RB01(**ПРИМЕР ТОЛЬКО ДЛЯ ЭТОГО РОУТЕРА, ЛЮБОЙ ДРУГОЙ ВЫ ТАКИМ ОБРАЗОМ МОЖЕТЕ ПРЕВРАТИТЬ В КИРПИЧ, гайд по раскирпичиванию [ЗДЕСЬ](https://github.com/Ultraheal/great-win-of-the-horde/blob/main/anti-queer-pitch.md)**, можете найти отдельный гайд по openWRT на свою модель, их в интернете полно), а также вот ссылка на то же самое, только в формате [ВИДЕО](https://www.youtube.com/watch?v=jhKnUGJv81c)

Нам потребуется:
- Компьютер с windows
- Роутер подключенный к компьютеру кабелем
- Сама прошивка openWRT
- Утилита для легкой прошивки xmir-patcher

Для удобства все утилиты и прошивки я разместил в репозитории, но если есть желание можете скачать их из официальных источников:
1) [xmir-patcher](https://github.com/openwrt-xiaomi/xmir-patcher)
2) [MIWIFIRepairTool](http://bigota.miwifi.com/xiaoqiang/tools/MIWIFIRepairTool.x86.zip)
3) [openWRT 23.05.5](https://firmware-selector.openwrt.org/?version=23.05.5&target=mediatek%2Fmt7622&id=xiaomi_redmi-router-ax6s)
4) [openWRT 24.10.5](https://firmware-selector.openwrt.org/?version=24.10.5&target=mediatek%2Fmt7622&id=xiaomi_redmi-router-ax6s)
5) [стоковая прошивка](http://cdn.awsde0-fusion.fds.api.mi-img.com/xiaoqiang/rom/rb01/miwifi_rb01_firmware_d8243_1.0.83_INT.bin)

Как установить прошивку:
1) Забрасываем фаил с прошивкой в папку firmware в xmir-patcher и запускаем !START.bat
2) В окошке утилиты нужно убедиться, что в пункте 1 "Set IP-address" указан адрес до веб интерфейса роутера "current value: 192.168.31.1", если его там нет или это не он, то устанавливаем
3) Далее выбираем пункт 2 "Connect to device (install exploit)" и вводим пароль от веб интерфейса роутера
4) Теперь выбираем пункт 6 "Install permanent SSH" чтобы установить доступ по SSH к роутеру
5) Далее выбираем пункт 7 "Install firmware (from directory "firmware")", чтобы установить прошивку на роутер
6) Дожидаемся перезагрузки роутера и закрываем xmir-patcher (выбрать пункт 0 или закрыть терминал)
7) Далее необходимо зайти в веб интерфейс, адрес роутера сменился на 192.168.1.1 логин/пароль - root/root
8) В желтом окошке нам предлагается сменить пароль со стандартного, лучше подчиниться
9) Далее необходимо подключиться к роутеру по ssh с помощью любого ssh клиента или терминала ssh root@192.168.1.1 - вводим пароль, который только что изменили
10) Далее необходимо ввести список команд по очереди:
    ```
    fw_setenv boot_fw1 "run boot_rd_img;bootm"
    fw_setenv flag_try_sys1_failed 0
    fw_setenv flag_try_sys2_failed 0
    fw_setenv flag_boot_rootfs 0
    fw_setenv flag_boot_success 1
    fw_setenv flag_last_success 1
    ```
11) Переходим обратно в веб интерфейс во вкладку System > Startup > Local Startup и добавляем в окошко следующий код
    ```
    # Put your custom commands here that should be executed once
    # the system init finished. By default this file does nothing.
    
    fw_setenv flag_try_sys1_failed 0
    fw_setenv flag_try_sys2_failed 0
    
    exit 0
    ```
    Сохраняем, идем в System > Reboot и перезагружаем роутер
12) Далее можем проверить, что изменения применились, для этого подключаемся к роутеру по ssh и вводим команду fw_printenv . В результате мы должны увидеть что flag_try_sys1_failed и flag_try_sys2_failed равны 0

# Обновление openWRT с 23.05.5 на 24.10.5
Если у вас стояла версия 23.05.5, то обновиться на 24.10.5 простым способом не получится, для начала придется откатиться на стоковую прошивку, а затем поставить версию 24.10.5 способом выше.
Откат на стоковую прошивку описан [ЗДЕСЬ](https://github.com/Ultraheal/great-win-of-the-horde/blob/main/anti-queer-pitch.md)(он же гайд по раскрипичиванию)

# Настройка Wi-Fi на openWRT
У openWRT своеобразная настройка wi-fi, в которой есть нюансы. Настройка происходит во вкладке network -> wireless там нужно кнопкой edit отредактировать стандартное подключение(либо создать его кнопкой add, если его нет). Вверху Operating frequency - если mode не выбран, то ваша точка доступа работать не будет, выберите нужное вам. Внизу во вкладке General setup, поле ESSID - это название вашей wi-fi точки. Во вкладке Wireless security, в поле Encryption задается тип шифрования, а в поле key пароль от вашей точки. Нюанс в следующем, если у вас есть устройства "умного дома", то они могут не захотеть подключаться к вашей wi-fi точке из-за специфичного шифрования, рекомендую попробовать шифрование WPA2-PSK, с ним у меня устройства подключались корректно
