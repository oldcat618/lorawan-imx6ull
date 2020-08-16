
# WirelessRoad GW-IMX6ULL OPENWRT

[收集的固件详细说明](README.details.md)

###固件装配程序

*安装您的分布软件包。为Ubuntu控制台执行命令：
> sudo apt-get update

> sudo apt-get install git-core build-essential libssl-dev libncurses5-dev unzip gawk zlib1g-dev subversion mercurial

**2.** 使用命令克隆源库:

> git clone https://github.com/wireless-road/lorawan-imx6ull

**3.** 进入源目录:

> cd lorawan-imx6ull

**4.** 选择所有可用的命令包:
> ./scripts/feeds update -a

> ./scripts/feeds install -a

**5.** 复制设备的openwrt配置文件 GW-IMX6ULL:

> cp openwrt-configs/gw-imx6ull ./.config

**6.**这个步骤可以跳过，如果你不需要额外的选项。

为精细配置，请执行命令：
> make menuconfig

**7.**预先下载所有必要的源代码包的命令:
> make download

**8.** 选择下面的一个选项，并开始一个安装程序的命令:

> make

使用4线程编译:

> make -j 4

**9.**当你完成了你的安装程序，在目录中的bin/targets/imx6ull/cortexa7/“”将有固件和启动图像写入/更新：

`openwrt-imx6ull-cortexa7-wirelessroad_gw-imx6ull-squashfs.sdcard.bin` - SD/MMC-映像.

`openwrt-imx6ull-cortexa7-wirelessroad_gw-imx6ull-squashfs.mtd-sysupgrade.bin` - 固件图像用于spi-flash，该文件用于远程更新。

`openwrt-imx6ull-cortexa7-wirelessroad_gw-imx6ull-squashfs.mtd-factory.bin` - spi-flash图像记录在一个单一的文件生产或完全更新固件和启动程序。

`openwrt-imx6ull-cortexa7-wirelessroad_gw-imx6ull-squashfs.u-boot.bin` - uboot启动映像.


### 更新和写入图像。

原始记录

使用固件，您必须将图像写入外部存储器）（SD卡）命令
> dd if=openwrt-imx6ull-cortexa7-wirelessroad_gw-imx6ull-squashfs.sdcard.bin of=/dev/sdX

而不是/dev sdx指定一个路径的SD卡设备。

После этого необходимо загрузить устройство с openwrt на SD-карте. Далее скачиваем файл прошивки для SPI-NOR (MTD) на устройство. Для этого можно воспользоваться FTP- или HTTP-сервером на локальном компьютере, или воспользоваться SCP-протоколом.

###### Загрузка с FTP- или HTTP-сервера.

Положите файл прошивки в директорию, к которой даёт доступ сервер. Для FTP используйте пользователя anonymous для обмена файлами по этому протоколу.
Допустим, IP-адрес сервера 192.168.1.100. Скачиваем файл в директорию /tmp на устройстве командой (для FTP):
> wget ftp://192.168.1.100/openwrt-imx6ull-cortexa7-wirelessroad_gw-imx6ull-squashfs.mtd-factory.bin -O /tmp/openwrt-imx6ull-cortexa7-wirelessroad_gw-imx6ull-squashfs.mtd-factory.bin

Или командой (для HTTP):
> wget http://192.168.1.100/openwrt-imx6ull-cortexa7-wirelessroad_gw-imx6ull-squashfs.mtd-factory.bin -O /tmp/openwrt-imx6ull-cortexa7-wirelessroad_gw-imx6ull-squashfs.mtd-factory.bin

Далее нам необходимо записать прошивку на SPI-NOR (MTD) флеш командой
> mtd write /tmp/openwrt-imx6ull-cortexa7-wirelessroad_gw-imx6ull-squashfs.mtd-factory.bin factory

Важно, команда mtd из состава openwrt перед записью проверяет название разделов в файле /proc/mtd, последний аргумент в команде указывает название раздела для записи, т.к. блочное устройство может отличаться при каждой загрузке.
После выполнения этой команды устройство готово для работы без SD-карты.

###### Загрузка с использованием SCP.

Допустим, IP-адрес устройства после загрузки 192.168.1.1.
Для выгрузки файла на устройство необходимо на компьютере выполнить команду
> scp openwrt-imx6ull-cortexa7-wirelessroad_gw-imx6ull-squashfs.mtd-factory.bin root@192.168.1.1:/tmp/

После этого необходимо выполнить команду записи прошивки на флеш с самого устройства.
> mtd write openwrt-imx6ull-cortexa7-wirelessroad_gw-imx6ull-squashfs.mtd-factory.bin factory

##### Обновление.

Как загружать файлы на устройство смотрите в предыдущем разделе.

Для обновления UBoot раздела скачайте файл `openwrt-imx6ull-cortexa7-wirelessroad_gw-imx6ull-squashfs.u-boot.bin` на устройство в директорию /tmp и выполните команду
> mtd write /tmp/openwrt-imx6ull-cortexa7-wirelessroad_gw-imx6ull-squashfs.u-boot.bin u-boot

Для обновления прошивки загрузите файл прошивки `openwrt-imx6ull-cortexa7-wirelessroad_gw-imx6ull-squashfs.mtd-sysupgrade.bin` на устройство с использованием wget (FTP или HTTP протокол) или scp, как в предыдущем разделе (первичная запись) и выполните команду 
> sysupgrade -с /tmp/openwrt-imx6ull-cortexa7-wirelessroad_gw-imx6ull-squashfs.mtd-sysupgrade.bin

Команда sysupgrade обновит прошивку с сохранением текущей конфигурации устройства. Ключ '-c' указывает команде найти все изменения в директории /etc/ на устройстве и сохранить их для следующей загрузки (по-умолчанию сохраняются только файлы из списка /etc/sysupgrade.conf, в основном, это конфигурационные файлы в /etc/config/).
