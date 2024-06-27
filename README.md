# Детаљне информације о пројекту
Овај репозиторијум демонстрира повезивање U-Blox LEA-6S GPS Click плоче на De1-SoC плочу и приказује једноставну апликацију за ишчитавање релевантних *GPS* података.

Циљ приложеног садржаја је да омогући додатне информације о:
1. Повезивању *GPS Click* компоненте кориштењем *UART* интерфејса (*RX* и *TX* пинови) на *De1-SoC* плочу
2. Кориштењу *gpsd* алата за рад са *GPS* подацима
3. Подешавању *Buildroot* окружења тако да се омогући детектовање примљених података
4. Ишчитавању релевантних информација добијених од *GPS* уређаја

## Текст задатка

Обезбиједити подршку за комуникацију са GPS уређајем који се налази на GPS Click плочици (интегрисано коло LEA-6S). Направити апликацију која ће кориснику да приказује све релевантне податке о тренутној локацији и осталим информацијама које обезбјеђује овај уређај.


## Предуслови за израду пројекта

За успјешно извођење демонстрираног пројекта неопходно је сљедеће:
- развојна плоча *De1-SoC* са припадајућим напајањем и каблом за повезивање рачунара са *UART* интерфејсом на плочи
- припремљен [*toolchain*](https://github.com/etf-unibl/urs-2024/blob/main/lab-01/lab-01.md) за кроскомпајлирање софтвера за циљну платформу
- припремљено [*Buildroot*](https://github.com/etf-unibl/urs-2024/blob/main/lab-08/lab-08.md) окружење 
- *SD* картица
- *Ethernet* мрежни интерфејс на развојној платформи и мрежни кабл за повезивање са *De1-SoC* плочом
- *GPS Click* плоча
- Антена
- 4 кабла за повезивање релевантних пинова *GPS Click* плоче и *De1-SoC* плоче

## Информације о *GPS Click* плочи

За више информација о *GPS Click* плочи неопходно је посјетити [GPS Click](https://www.mikroe.com/gps-click).

Да би се детаљније упознали са компонентом неопходно је прочитати [*datasheet*](https://download.mikroe.com/documents/datasheets/LEA-6S_datasheet.pdf).

Поред секције *1.10 Protocols and interfaces* која ће се спомињати и у наставку текста, битна секција је и *7 Default settings* гдје видимо да за серијски порт треба да се поставе сљедећи параметри:

```
9600 Baud
8 bits
No parity bit
1 stop bit
```

Дати параметри могу да се поставе кориштењем *stty* сљедећом командом за жељени интерфејс (у овом случају за */dev/ttyS1* иако су они предефинисано били постављени на свим неопходним мјестима, па извршавање команде није било потребно).

```
stty -F /dev/ttyS1 9600 cs8 -cstopb -parenb
```

> [!TIP]
> За више информација о *stty* може се искористити команда *man stty*.


## Повезивање *GPS Click* компоненте на *De1-SoC* плочу

На слици испод је приказан начин повезивања *GPS Click* на *De1-SoC* плочу.

![ploca-gps](https://github.com/novicatepic/-------/assets/62095336/ca5be09b-2423-4071-b9c6-9a075bd03c38)

Као што се може видјети са слике, спојени су уземљење и напајање (3.3V), те је TX пин *GPS Click* плоче спојен на *RX* пин *De1-SoC* плоче, као и *RX* пин *GPS Click* плоче на *TX* пин *De1-SoC* плоче.

На страници бр. 13 [*сљедећег линка*](https://people.ece.cornell.edu/land/courses/ece5760/DE1_SOC/DE1-SoC%20schematic.pdf) могу да се виде пинови за напајање и уземљене за *GPIO1* контролер.

Антена је такође спојена на *GPS Click*.

*Ethernet* мрежни кабл је искориштен да би се употребом *SCP* протокола могли копирати фајлови на циљну платформу.

На крају, искориштен је и кабл за повезивање лаптопа са *UART* интерфејсом на *De1-SoC* плочи.

Детаљнији приказ *GPS Click* плоче може да се види на слици испод.

![GPS Click](https://github.com/novicatepic/-------/assets/62095336/637ecddd-b29c-4b88-9581-3ec74d967232)


## Прилагођавање конфигурације SPL-а и U-Boot окружења

Конфигурацију *SPL-а* вршимо примјеном *patch* фајла подешавањем опције *Bootloaders -> U-Boot -> Custom U-Boot patches* у конфигурацији *Buildroot-а*, гдје постављамо путању у односу на на *board* фолдер, у овом случају на *board/terasic/de1soc_cyclone5/patches/de1-soc-handoff.patch*.

На слици испод је приказано гдје у конфигурацији *Buildroot-а* након покретања команде *make menuconfig* из кровног директоријума може да се подеси дата опција.

![bootloader](https://github.com/novicatepic/-------/assets/62095336/b03437fe-a0ba-41f0-8bb5-c235835b107e)

Такође, неопходно је измијенити садржај *boot-env.txt* фајла у складу са приложеним фајлом на овом репозиторијуму, а фајл треба да се налази у *board/terasic/de1soc_cyclone5* директоријуму.

## Подешавање *Buildroot* окружења употребом *menuconfig-а* и модификавија *rootfs overlay-а*

Да би се омогућио *gpsd* алат, неопходно је у коријеном фолдеру *Buildroot* покренути команду

```
make menuconfig
```

Након тога, неопходно је изабрати сљедећу опцију:

*Target Packages -> Hardware Handling -> gpsd*

Сљедећи корак је да се изабере */dev/ttyS1* у оквиру опције *Where to look for GPSes*, те да се укључе *client debugging support* и *profilling support* као двије опционе ставке (нису кориштене у раду), те протоколи приказани на слици.

Да би се детаљније упознали са кориштеним протоколима за *GPS Click*, неопходно је посјетити секцију *1.10 Protocols and interfaces* у [*datasheet-у*](https://download.mikroe.com/documents/datasheets/LEA-6S_datasheet.pdf).

Протоколи који су укључени на слици а не налазе се у *datasheet-у* су укључени аутоматски након укључивања неких од тражених протокола.

Коначно, неопходно је командом *Save* сачувати измјене и напустити *menuconfig*.

Приказ изабраних опција можемо видјети и на сликама испод.

![buildroot-1](https://github.com/novicatepic/-------/assets/62095336/f319fcf2-e674-452a-bcdd-e88a0130f5ce)

![buildroot-2](https://github.com/novicatepic/-------/assets/62095336/d61a4ecf-f7cf-4d95-a9e8-16c361b099f8)


Да би се модификовао *rootfs overlay*, неопходно је унутар постојеће *rootfs overlay* инфраструктуре која је референцирана у [*лаборатијској вјежби број 8*](https://github.com/etf-unibl/urs-2024/blob/main/lab-08/lab-08.md) додати фајл унутар */etc/default* чији је назив *gpsd*.

Разлог за модификацију овог фајла јесте чињеница да он није аутоматски креиран на циљној платформи, а с обзиром да је постојао на развојној платформи закључак је био да га је неопходно било накнадно додати. Без постојања овог фајла програм који користи алат *gpsd*, при чему *gpsd* ради на порту 2947 није могао да чита *GPS* податке.

Садржај фајла треба да буде гдје је најбитније навести */dev/ttyS1* који је наведен и у *Buildroot* опцији *Where to look for GPSes*:

```
DEVICES="/dev/ttyS1"
GPSD_OPTIONS=""
USBAUTO="true"
```

На крају, неопходно је покренути сљедеће команде како би се примијениле све измјене:

```
make clean
make
```

Задњи корак јесте актуелизовање садржаја *SD* картице употребом *dd* алатке, те је због тога неопходно прећи у *output/images* директоријум (унутар *Buildroot* директоријума) и извршити сљедећу команду:

```
sudo dd if=sdcard.img of=/dev/sdb bs=1M
```

> [!CAUTION]
> Обратите посебну пажњу на то да изаберете исправан уређај, јер слично име може да има главни хард диск рачунара на којем радите. Поменута команда може да проузрокује брисање комплетног фајл система рачунара на којем радите ако изаберете погрешан уређај.

Након актуелизовања садржаја *SD* картице, унутар *FAT32* партиције на *SD* картици неопходно је копирати фајл *socfpga.rbf*.

> [!TIP]
> У случају да нема довољно простора на *FAT32* партицији неопходно је у фајлу *board/terasic/de1soc_cyclone5/genimage.cfg* унутар *image boot.vfat* секције повећати *size*, нпр. на *24M* и покренути команду *make* из *Buildroot* директоријума.


## Кроскомпајлирање апликације и копирање апликације на развојну платформу

Унутар фајла *gps.c* остављени су коментари (на енглеском језику, јер је и код тако написан) који објашњавају дати код.

Да бисмо кроскомпајлирали изворни фајл *gps.c*, неопходно је да експортујемо путању до *toolchain* алата сљедећом командом (под претпоставком да се налазимо у *GPSClick-Project* фолдеру):

```
source ../set-environment.sh
```

Након тога, кроскомпајлирање се врши наредном командом

```
arm-linux-gcc gps.c -o gps -I<path_to_buildroot>/output/staging/usr/include -L<path_to_buildroot>/output/staging/usr/lib -lgps -ldbus-1 -lsystemd -lcap
```

Разлог за употребу ове команде је проблем са кроскомпајлирањем библиотеке која се укључује са *include "gps.h"* јер је кроскомпајлирање базирано на *SCons* гдје је неопходно модификовати *SConstruct* фајл који је написан у *Python* програмском језику.

Да би се ријешио проблем, унутар *Buildroot-а" већ постоји дата библиотека због укључивања *gpsd-а*, те ју је само неопходно пронаћи и укључити и друге библиотеке од којих она зависи при кроскомпајлирању.

> [!NOTE]
У оквиру *GPS-Click* остављен је и фајл *gps-manual.c* којим је демонстрирано читање података, без додатног парсирања, директним приступом интерфејсу гдје се примају *GPS* подаци у одговарајућим форматима. Овакав приступ није препоручљив.

Извршни фајл на циљну платформу копирамо употребом *SCP* протокола јер смо у лабораторијској вјежби број 8 (претходно референцираној) омогућили *SSH* конекцију:

```
scp gps root@192.168.21.100:/home
```

Примјер исписа након успјешног копирања фајла на циљну платформу:

```
gps                                           100%   12KB   2.3MB/s   00:00
```


## Извршавање апликације на циљној платформи


Након покретања плоче и пријављивања на систем, неопходно се позиционирати у фолдер *home* јер је ту прекопиран програм и покренути његово извршавање:

Командом *ls* можемо верификовати постојаност извршног фајла на циљној платформи.

```
cd /home
ls
./gps
```

Примјер извршавања програма приказан је на слици испод.

![result](https://github.com/novicatepic/-------/assets/62095336/d810f871-210a-444c-912f-536934297a5e)

Са слике се могу видјети испис времена, географске ширине (*Latitude*), географске дужине (*Longitude*), надморске висине (*Longitude*), брзине у метрима у секунди (*Speed*), параметара *track*, *climb*, кориштени *fix mode* (0 означава да није кориштен, 1 да је кориштен *2D Fix*, а 2 да је кориштен *3D Fix*), *DOP* (Dilution of Precision) који мјери квалитет *GPS* сигнала.

Поред тога, могу да се испишу и сателити што је покривено у програму *gps.c*, али понекад се извршавањем програма на циљној платформи не добију и ти резултати. Параметри за сателите који се могу исписати су број видљивих сателита, *PRN* (Pseudo Random Noise) сваког сателита, јачина сигнала сателита, *azimuth* и параметар *elevation*.


## Корисне опције 


У датом пројекту није било неопходно модификовати *DTS* фајл, али да је то био случај, након измјене *DTS* фајла унутар *Buildroot-а* је могуће покренути команду чиме ће се поново изградити *kernel*, али и примијенити модификовани *DTS* фајл: 

```
make linux-rebuild
```

Уколико је неопходно ажурирати опције за *kernel*, што је поготово корисно за драјвере, постоји опција: 

```
make linux-menuconfig
```

У случају визуелизације *GPS* података користан програм на *Windows* оперативном систему је *u-center*.

Програми *PuTTY* и *Tera Term* на *Windows* оперативном систему могу да послуже за провјеру исправног функционисања *GPS* уређаја и пријема података.
