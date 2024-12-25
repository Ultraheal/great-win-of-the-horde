# Прошивка OpenWrt
Для великой победы, нам необходима прошивка OpenWrt на роутере и доступ по ssh к нему, ниже пример с гайдом для роутера Xiaomi Router AX3200 RB01(пример только для этого роутера, любой другой вы таким образом можете превратить в кирпич), а также вот ссылка на то же самое, только в формате видео https://www.youtube.com/watch?v=jhKnUGJv81c :

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

# Точечная маршрутизация и обновляемый по крону список доменов
Есть великолепный гайд, из которого ниже я вынесу некоторые важные моменты, и свои дополнения https://itdog.info/tochechnaya-marshrutizaciya-po-domenam-na-routere-s-openwrt/

А также имеется его частичная видео-версия - https://youtu.be/Otv-kMzGOSU?si=5hf_vnCy_UbqdBpP , где показана настройка маршрутизации через sing-box(об этом дальше):

## Туннель через Sing-box или туннель через Amnezia
В гайде есть shell-скрипт для автоматической настройки туннеля:

 ```sh <(wget -O - https://raw.githubusercontent.com/itdoginfo/domain-routing-openwrt/master/getdomains-install.sh)```

Достаточно зайти в роутер по ssh и запустить его в консоли, также в гайде есть полная ручная настройка для особых параноиков. Скрипт сделает всё за вас, и туннелирование, и маршрутные листы, и их обновление по крону. При запуске скрипта будет предложен выбор sing-box или amneziaWG

Моя персональная рекомендация - AmneziaWG. При запуске, выбрав AmneziaWG скрипт предложит ввести всякие разные ключи и параметры для конфигурации туннеля, это все можно заполнить рандомными данными(потом их можно отредактировать), если у вас еще не поднят сервак с контейнером амнезии. Либо же если у вас уже арендован сервер и развернут впн AmneziaWG на нем, то можно заполнить настоящими данными

Собственно почему рекомендую амнезию - в сравнении с протоколами поддерживаемыми sing-box (shadowsocks2022, vless, vmess и прочие) амнезия предлагает лучшую скорость(ну, по моим ощущениям, я попробовал все варианты), и сама технология амнезии кажется мне более перспективной в дальнейшей устойчивости к блокировкам

# Развертывание VPN контейнеров на собственном сервере
Нам потребуется:
- Арендовать какой-нибудь сервер на удобном хостинге(подойдет любой самый дешевый, нас интересует только ширина канала, по которому мы будем гонять трафик - поэтому железо не играет роли, только соединение)
- Выбрать серверу любую UNIX операционку, CentOs, Ubuntu, RockyLinux - тут на ваш вкус

Я в данный момент использую PQ.Hosting, пока что за время использования нареканий не выявил, соединение неплохое, ничего не лагает.

Можете выбрать хостинг на ваш вкус и цвет, или же зарегаться на этом, можно по моей рефералке, мне там какая-то копеечка перепадет: https://pq.hosting/?from=914169

## Путь через Sing-box
Итак, если мы установили туннель через sing-box из скрипта в пункте выше - то можно использовать тоже гайд от itdog - также текстовая и видео версии:

https://itdog.info/kak-podnyat-proksi-server-s-tekhnologiej-shadowsocks2022-s-pomoshchyu-docker-sozdayom-neskolko-akkauntov-upravlyaem-vsem-cherez-komandnuyu-stroku/

https://www.youtube.com/watch?v=toG46r6G6Xg&ab_channel=ITDog

После выполненных махинаций, нам необходимо зайти в наш роутер по ssh и отредактировать конфигурацию sing-box:

```sh
nano /etc/sing-box/config.json
```

И перезагрузить sing-box и нетворк:

```
service sing-box restart && service network restart
```

Как альтернативу docker image из гайда можно использовать кое-что другое, но об этом чуть ниже, это пойдет в пункт с VPN для мобильных устройств, там свои особенности

## Путь через Amnezia
Если мы установили туннель через AmneziaWG из скрипта выше, то тут все несколько проще:

Есть много разных docker image с AmneziaWG, я остановил свой выбор на этом - https://github.com/spcfox/amnezia-wg-easy

Там легко развернуть всё парой команд, и плюсом у вас будет админка, в которой можно редактировать клиентов, подключать, отключать, раздавать конфиги и все такое

После того как развернули, заходим в админку, создаем клиента, и загружаем файлик с конфигом оттуда

Затем бежим в веб интерфейс нашего роутера и переходим в Network > Interfaces

Здесь мы должны увидеть туннель, который мы создавали ранее скриптом, по дефолту он называется awg0

В нем мы должны отредактировать конфигурацию кнопочкой Edit

Во вкладке General Settings внизу есть кнопка Load configuration, туда мы можем загрузить недавно созданный в админке конфиг. Лично у меня он импортировался криво, и мне пришлось ручками вписывать значения из этого конфига в соответствующие поля, может вам повезет больше. Но если нет, просто откройте конфиг блокнотом, и перепишите данные ручками

Сохраняем все это дело и перезагружаем роутер System > Reboot

# И кажется что на этом всё... Но, нет
Скорее всего, после всей проделанной работы всё у вас будет работать замечательно, кроме голосовых каналов в Discord, чтобы это дело подправить - делаем следующее подключившись к нашему роутеру по ssh:

- Добавляем кастомные домены дискорда, которых нет в основных обновляемых списках маршрутов
- Добавляем ip-адреса и подсети голосовых каналов discord бонусом к списку доменов

## Добавление кастомных доменов
Открываем конфиг любым консольным текстовым редактором например nano, который скрипт устанавливает попутно:

```nano /etc/config/dhcp```

И в конец файла добавляем следующее:

```
config ipset
        list name 'vpn_domains'
        list domain 'media.discordapp.net'
        list domain 'gateway.discord.gg'
        list domain 'cdn.discordapp.com'
        list domain 'images-ext-1.discordapp.net'
        list domain 'discord.app'
        list domain 'discordcdn.com'
        list domain 'cloudflare-ech.com'
```

И после делаем ```service dnsmasq restart```

## Добавление ip-адресов и подсетей
Открываем другой конфиг:

```nano /etc/config/firewall```

И добавляем в конец списка:

```
config ipset
  option name 'vpn_subnet'
  option match 'dst_net'
  list entry '31.13.24.0/21'
  list entry '31.13.64.0/18'
  list entry '45.64.40.0/22'
  list entry '57.141.0.0/24'
  list entry '57.141.3.0/24'
  list entry '57.141.5.0/24'
  list entry '57.141.7.0/24'
  list entry '57.141.8.0/24'
  list entry '57.141.10.0/24'
  list entry '57.141.13.0/24'
  list entry '57.144.0.0/14'
  list entry '66.220.144.0/20'
  list entry '69.63.176.0/20'
  list entry '69.171.224.0/19'
  list entry '74.119.76.0/22'
  list entry '102.132.96.0/20'
  list entry '103.4.96.0/22'
  list entry '129.134.0.0/17'
  list entry '157.240.0.0/17'
  list entry '157.240.192.0/18'
  list entry '163.70.128.0/17'
  list entry '173.252.64.0/18'
  list entry '179.60.192.0/22'
  list entry '185.60.216.0/22'
  list entry '185.89.216.0/22'
  list entry '204.15.20.0/22'
  list entry '138.128.136.0/21'
  list entry '162.158.0.0/15'
  list entry '172.64.0.0/13'
  list entry '34.0.0.0/15'
  list entry '34.2.0.0/16'
  list entry '34.3.0.0/23'
  list entry '34.3.2.0/24'
  list entry '35.192.0.0/12'
  list entry '35.208.0.0/12'
  list entry '35.224.0.0/12'
  list entry '35.240.0.0/13'
  list entry '5.200.14.128/25'
  list entry '66.22.192.0/18'

config rule
  option name 'mark_subnet'
  option src 'lan'
  option dest '*'
  option proto 'all'
  option ipset 'vpn_subnet'
  option set_mark '0x1'
  option target 'MARK'
  option family 'ipv4'
```

После перезагружаем фаирволл:

```service firewall restart```

Айпишники дискордовских голосовых каналов находятся в конце списка, начиная с 138.128.136.0/21 - остальное это всякие айпишники для картиночек и голосовух в соцсетях, в целом тоже полезное, можете оставить, можете убрать

# Мобильные устройства и VPN при работе от мобильного интернета
Выше мы рассмотрели как сделать всё красиво если мы находимся дома и сидим на интернете от нашего роутера. Но иногда мы все таки выходим из дома(невероятно, но факт)

К моему глубочайшему сожалению, оказывается мобильные сети, и сети обычных "домашних" интернет-провайдеров работают совершенно по-разному, и блокировки протоколов VPN тоже работают неожиданно. В этой теме я не разобрался до конца, но возникающие проблемы решил:

Конкретно в случае с моим оператором(Мегафон) мой замечательный конфиг от AmneziaWG работал с перебоями на мобильном интернете, но там где плохо работает один протокол - хорошо работает какой-нибудь другой, оставалось только найти

Так на моем сервере, рядышком с контейнером амнезии поселился контейнер с https://github.com/MHSanaei/3x-ui это мультипротокольный сервис, который так же имеет админку, и позволяет поднять несколько различных современных протоколов VPN, также менеджить клиентов как в админке амнезии и все такое.

Есть отличный подробный гайд https://habr.com/ru/articles/735536/ который рассказывает, как всё это дело очень секьюрно настроить, также в гайде предлагаются альтернативы репозиторию, который я указал выше

Так вот, в случае с моим мобильным интернетом мне помог протокол VLESS с XTLS-Reality

Но и тут есть ложка дёгтя, протокол сам по себе довольно медленный(но очень секьюрный) + мобильный интернет не чемпион в гонке скоростей, в совокупности с мобильного интернета я получил видосы на ютубе в 720p без лагов. В целом неплохо, но наверное можно и лучше

В качестве клиентов для VPN на мобильных устройствах с этим протоколом для Android предлагаю обратить внимание на v2rayNG, а для iOS на FoXray. Но тут тоже, каждому на вкус и цвет.

Только дома VPN клиенты на мобилках не забывайте отключать, VPN то на роутере и так есть

# Итоги
В результате, что мы имеем:
- Собственный сервер с 2мя поднятыми админками, с различными VPN протоколами
- Домашний интернет работающий через роутер с точечной маршрутизацией, заблокированные маршруты в VPN, обычные же без него
- Мобильный интернет через приложения-VPN клиенты

# Полезные ссылки
Во всем этом очень помогли разобраться репозитории itdog, возможно вам там тоже что-нибудь когда-нибудь пригодится:

https://github.com/itdoginfo

А так же его телеграм чат, там много тем, папочек, вкладочек с целым вагоном полезной информации, да и просто можно спросить что-нибудь у чуваков в чате:

https://t.me/itdoginfo
