# Прошивка OpenWrt
Для великой победы, нам необходима прошивка OpenWrt на роутере и доступ по ssh к нему, ниже пример с гайдом для роутера Xiaomi Router AX3200 RB01:

Нам потребуется:
- Компьютер с windows
- Роутер подключенный к компьютеру кабелем
- Сама прошивка(сам делал только с версией 23.05.5, с ней точно все работает как описано) https://firmware-selector.openwrt.org/?version=23.05.5&target=mediatek%2Fmt7622&id=xiaomi_redmi-router-ax6s
- Утилита для легкой прошивки https://github.com/openwrt-xiaomi/xmir-patcher

Как установить прошивку:
1) Забрасываем фаил с прошивкой в папку firmware в xmir-patcher и запускаем !START.bat
2) В окошке утилиты нужно убедиться, что в пункте 1 указан адрес до веб интерфейса роутера, если его там нет или это не он, то устанавливаем
3) Далее выбираем пункт 2 и вводим пароль от веб интерфейса роутера
4) Теперь выбираем пункт 8, а затем пункт 7 чтобы установить доступ по SSH к роутеру
5) Далее выбираем 0, чтобы вернуться в начальное меню и выбираем 7, чтобы установить прошивку на роутер
6) Дожидаемся перезагрузки роутера и закрываем xmir-patcher
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

Хороший видео гайд, рассказывающий то же самое: https://www.youtube.com/watch?v=jhKnUGJv81c
