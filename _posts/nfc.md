---
title: NFC Notes
date: 2019-02-11 10:16:17
tags:
---
# NFC笔记

在淘宝上买了一个IC卡复制工具

![小黑NFC读卡器](/myimages/nfc_01_reader.png)

![小黑NFC读卡器](/myimages/nfc_01_reader2.png)

带了Windows版的软件和驱动

![软件](/myimages/nfc_02_windowssoftware.png)

带的软件里有libnfc.dll, mfoc.dll两个文件。驱动程序就是一个"Prolific Technology, Inc. PL2303 Serial Port“ USB转串口驱动。读卡器是USB接口的，安装驱动之后映射岛了COM3。
因此想到是否可以找到其他的软件使用这个读卡器。

以下记录Linux(ubuntu)下找到的方法

# Ubuntu Linux下用NFC reader破解复制M1卡

linux自带了PL2303驱动，插上读卡器后用lsusb命令可以看到
”Bus 001 Device 018: ID 067b:2303 Prolific Technology, Inc. PL2303 Serial Port“

```console
$ lsusb
Bus 002 Device 003: ID 046d:c534 Logitech, Inc. Unifying Receiver
Bus 002 Device 002: ID 8087:0020 Intel Corp. Integrated Rate Matching Hub
Bus 002 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
Bus 001 Device 005: ID 17ef:4816 Lenovo Integrated Webcam
Bus 001 Device 004: ID 0a5c:217f Broadcom Corp. BCM2045B (BDC-2.1)
Bus 001 Device 003: ID 147e:2016 Upek Biometric Touchchip/Touchstrip Fingerprint Sensor
Bus 001 Device 018: ID 067b:2303 Prolific Technology, Inc. PL2303 Serial Port
Bus 001 Device 002: ID 8087:0020 Intel Corp. Integrated Rate Matching Hub
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub

```

安装了libnfc之后，可以用以下命令看到读卡器已经可以找到，注意，必须用root权限，否则会有以下结果

```console
nfc-list
nfc-list uses libnfc 1.7.1
error
libnfc.driver.pn532_uart
Invalid serial port: /dev/ttyUSB0
nfc-list: ERROR: Unable to open NFC device: pn532_uart:/dev/ttyUSB0
```

用root权限则是以下结果

```console
sudo nfc-list
[sudo] quxr 的密码： 
nfc-list uses libnfc 1.7.1
NFC device: pn532_uart:/dev/ttyUSB0 opened

```

## libnfc安装遇到的问题

- /etc/nfc/libnfc.conf
需要把最后两行注释去掉

```aconf
device.name = "microBuilder.eu"
device.connstring = "pn532_uart:/dev/ttyUSB0"
```

如果系统没安装libnfc，可以用一下命令安装

```console
sudo apt install libnfc-bin
```

另外，需要安装libnfc-dev包，因为一会编译mfoc需要用到，mfoc在apt里找不到安装包，需要自己编译
```console
sudo apt install libnfc-dev
```

[libnfc](https://github.com/nfc-tools/libnfc)

## 安装mfoc

[mfoc](https://github.com/nfc-tools/mfoc)

```console
mkdir -p ~/work/nfc/nfc-tools/
cd ~/work/nfc/nfc-tools/
git clone https://github.com/nfc-tools/mfoc
cd mfoc
autoreconf -is
./configure
make && sudo make install
```

## 安装 libnfc-examples

```console
sudo apt install libnfc-examples
```

## nfc-cloner脚本

[nfc-cloner视频教程(youtube)](https://www.youtube.com/watch?v=c0Qsmgvj_oo)

以下脚本需要用root权限运行

[nfc-cloner.sh脚本](https://www.lostserver.com/static/nfc-cloner.sh)

```bash
#!/bin/bash
#set -x
#------------------------------------------------------
# Keld Norman, Dubex A/S, Sep, 2018 - CLONE ID CARD POC
#------------------------------------------------------
# NFC Card reader/writer used for this POC: 
# Type: ACR122U-A9 
# Product: ACR122U PICC Interface
# Manufacturer: Advanced Cardsystems Ltd. (ACS)
#
# idVendor=072f, idProduct=2200, bcdDevice= 2.07
# device strings: Mfr=1, Product=2, SerialNumber=0
# Website: www.asc.com.hk 
#
# Some good commands to know about: 
# 
# READ UID FROM CARD:  sudo nfc-anticol
# WRITE UID TO CARD:   sudo nfc-mfsetuid 1234ACD
# CLEAN (FORMAT) CARD: sudo nfc-mfsetuid -f
# 
#------------------------------------------------------
# Banner for the 1337'ish ness
#------------------------------------------------------
clear
cat <<EOF
   _____
  |A .  | _____
  | /.\ ||A ^  | _____
  |(_._)|| / \ ||A _  | _____
  |  |  || \ / || ( ) ||A_ _ |
  |____V||  .  ||(_'_)||( v )|
         |____V||  |  || \ / |
                |____V||  .  |
                       |____V|

 Dubex Redteam Card Cloner utility

EOF
#----------------------------------------
# VARIABLES
#----------------------------------------
SAVE_CARD_DUMP_FILES_HERE="./card_data"
WRITE_SIDE="A" # Can be A or B 
#----------------------------------------
# PRE: 
#----------------------------------------
if [ $(/usr/bin/id -u) -ne 0 ]; then 
 printf " ### ERROR - You must run this script with sudo or as the user root!\n\n"
 exit 1
fi
 
if [ "$(/bin/uname --kernel-release)" != "4.18.3" ]; then 
 echo ""
 echo " ### INFO - The Linux kernel must be this version or higher: Linux edge 4.18.3"
 echo "            Otherwise the native NFC card reader will crash (see it with the command dmesg)"
 echo ""
fi
# 
if [ ! -d ${SAVE_CARD_DUMP_FILES_HERE} ]; then 
 mkdir -p -m 0700 ${SAVE_CARD_DUMP_FILES_HERE} >/dev/null 2>&1
fi
#
PROGRAM="$(/usr/bin/basename ${0})"
MFOC="/usr/local/bin/mfoc"           # apt-get install mfoc
ANTICOL="/usr/bin/nfc-anticol"       # apt-get install libnfc-examples
NFC_POLL="/usr/bin/nfc-poll"         # apt-get install libnfc-examples
NFC_SETUID="/usr/bin/nfc-mfsetuid"   # apt-get install libnfc-examples
NFC_LIST="/usr/bin/nfc-list"         # apt-get install libnfc-bin
NFC_CLASSIC="/usr/bin/nfc-mfclassic" # apt-get install libnfc-bin
TIMESTAMP="$(date '+%Y_%m_%d_%H%M%S')"
#
for UTIL in ${MFOC} ${ANTICOL} ${NFC_LIST} ${NFC_POLL} ${NFC_SETUID} ${NFC_CLASSIC}; do 
 if [ ! -x ${UTIL} ];then 
  echo " ### ERROR - Utility ${UTIL} not found!"
 printf "\n Help to installing the missing utilities:\n
 /usr/local/bin/mfoc    # apt-get install mfoc
 /usr/bin/nfc-mfclassic # apt-get install libnfc-bin
 /usr/bin/nfc-list      # apt-get install libnfc-bin
 /usr/bin/nfc-anticol   # apt-get install libnfc-examples
 /usr/bin/nfc-mfssetuid # apt-get install libnfc-examples
 /usr/bin/nfc-poll      # apt-get install libnfc-examples\n\n"
  exit 1
 fi
done
#----------------------------------------
# FUNCTIONS
#----------------------------------------
function detect_card_reader() {
 CARD_READER_FOUND=0
 while [ ${CARD_READER_FOUND} -eq 0 ]; do
  READER_DATA="$(${NFC_LIST} 2>&1)"
  CARD_READER_FOUND=$(echo "${READER_DATA}"|grep -c -E "^NFC device:.*opened$")
 sleep 1
 done
 return 0
}
#----------------------------------------
# MAIN
#----------------------------------------
printf " %-55s" "Detecting card reader.."
detect_card_reader
echo "[OK]"
#
READ_CARD_LOOP=1
while [ ${READ_CARD_LOOP} -eq 1 ]; do
 printf " %-55s" "Place the card to clone on the reader now.."
 read_uid_success="FALSE"
 while [ "${read_uid_success}" == "FALSE" ]; do 
  DATA="$(${ANTICOL} 2>&1)"
  FAILED=$(echo ${DATA}|grep -c "Error: No tag available")
  if [ ${FAILED} -eq 0 ]; then 
   CARD_UID=$(echo "${DATA}"|grep "UID:"|awk '{print $2}')
   CARD_ATQA=$(echo "${DATA}"|grep "ATQA:"|awk '{print $2}')
   CARD_SAK=$(echo "${DATA}"|grep "SAK:"|awk '{print $2}')
   if [ "${CARD_UID}" == "" -o "${CARD_ATQA}" == "" -o "${CARD_SAK}" == "" ]; then 
    sleep .5
   else
    read_uid_success="SUCCESS"
   fi
  fi
 done
 echo "[OK]"
 #
 printf "\n ##### CARD DATA DUPLICATION IN PROGRESS ##### (this often takes up to 75 seconds)"
 printf "\n %-55s" "$(date) Started  cloning of ID-Card with UID: ${CARD_UID} (ATQA:${CARD_ATQA}/SAK:${CARD_SAK}) "
 ${MFOC} -O ${SAVE_CARD_DUMP_FILES_HERE:-.}/${CARD_UID}.${TIMESTAMP}.mfd > ${PROGRAM}.log 2>${PROGRAM}.error
 if [ $? -ne 0 ]; then 
  printf "\n ##### CARD DATA DUPLICATION FAILED !!!! #####\n\n"
  READ_CARD_LOOP=1
 else 
  READ_CARD_LOOP=0
  printf "\n %-55s" "$(date) Finished cloning of ID-Card with UID: ${CARD_UID} (ATQA:${CARD_ATQA}/SAK:${CARD_SAK}) "
 fi
done # LOOP IF READ FAILED
#-------------------------------------------------
# WAIT FOR REPLACED CARD
#-------------------------------------------------
sleep .5
printf "\n\n %-55s" "Remove the original card.. "
CARD_REMOVED=0
while [ ${CARD_REMOVED} -eq 0 ]; do
 ${NFC_POLL} > /dev/null 2>&1
 if [ $? -eq 0 ]; then 
  CARD_REMOVED=1
 fi
done
echo "[OK]"
#-------------------------------------------------
# WAIT FOR REPLACED CARD
#-------------------------------------------------
ALLOW_OVERWRITE=0
NEW_UID="${UID}"
while [ ${ALLOW_OVERWRITE} -eq 0 ]; do
printf " %-55s" "Place a new blank card on the NFC Writer.."
 read_new_uid_success="FALSE"
 while [ "${read_new_uid_success}" == "FALSE" ]; do 
  NEW_DATA="$(${ANTICOL} 2>&1)"
  NEW_FAILED=$(echo ${NEW_DATA}|grep -c "Error: No tag available")
  if [ ${NEW_FAILED} -eq 0 ]; then 
   NEW_CARD_UID=$(echo "${NEW_DATA}"|grep "UID:"|awk '{print $2}')
   NEW_CARD_ATQA=$(echo "${NEW_DATA}"|grep "ATQA:"|awk '{print $2}')
   NEW_CARD_SAK=$(echo "${NEW_DATA}"|grep "SAK:"|awk '{print $2}')
   if [ "${NEW_CARD_UID}" == "" -o "${NEW_CARD_ATQA}" == "" -o "${NEW_CARD_SAK}" == "" ]; then 
    sleep .5
   else
    read_new_uid_success="SUCCESS"
   fi
  fi
 done
 if [ "${CARD_UID}" == "${NEW_CARD_UID}" ]; then 
  echo "[WARNING]"
  printf "\n ### WARNING - The new card has the same UID as the old card!\n\n"
  read -r -p " QUESTION: Do You want to overwrite this card (y/n)? " response
  if [[ "$response" =~ ^([yY][eE][sS]|[yY])+$ ]]
   then
    ALLOW_OVERWRITE=1
  else
   ALLOW_OVERWRITE=0
   sleep .5
   printf "\n %-55s" "Remove the original card.. "
   CARD_REMOVED=0
   while [ ${CARD_REMOVED} -eq 0 ]; do
    ${NFC_POLL} > /dev/null 2>&1
    if [ $? -eq 0 ]; then 
     CARD_REMOVED=1
    fi
   done
   echo "[OK]"
  fi
 else
  echo "[OK]"
  ALLOW_OVERWRITE=1
 fi
done
#----------------------------------------
#  WRITE THE CLONED CARD
#----------------------------------------
printf "\n %-55s" "Writing UID: ${CARD_UID} on the new card "
${NFC_SETUID} ${CARD_UID} >> ${PROGRAM}.log 2>${PROGRAM}.error
if [ $? -ne 0 ]; then 
 echo "[ERROR]"
 printf "\n ### ERROR - An error occoured writing UID on the new card!"
 printf "\n             You need special cards to write to sector 0 (it is normally readonly)\n\n"
 exit 1
else
 echo "[OK]"
fi
printf " %-55s\n\n" "Writing (${WRITE_SIDE} side) data from original card:"
${NFC_CLASSIC} w ${WRITE_SIDE} ${SAVE_CARD_DUMP_FILES_HERE:-.}/${CARD_UID}.${TIMESTAMP}.mfd | tee -a ${PROGRAM}.log 2>${PROGRAM}.error
echo ""
#----------------------------------------
# END OF SCRIPT
#----------------------------------------
```

```console
sudo ./nfc-cloner.sh
```

## 各种UID卡

M1 卡
普通IC卡，0扇区不可以修改，其他扇区可反复擦写，我们使用的电梯卡、门禁卡等智能卡发卡商所使用的都是 M1 卡，可以理解为物业发的原卡。

UID 卡
普通复制卡，可以重复擦写所有扇区，主要应用在IC卡复制上，遇到带有防火墙的读卡器就会失效。

CUID 卡
可擦写防屏蔽卡，可以重复擦写所有扇区，UID卡复制无效的情况下使用，可以绕过防火墙。

FUID 卡
不可擦写防屏蔽卡，此卡的特点0扇区只能写入一次，写入一次变成 M1 卡，CUID 复制没用的情况下使用，可以绕过防火墙。

UFUID 卡
高级复制卡，我们就理解为是 UID 和 FUID 的合成卡，需要封卡操作，不封卡就是 UID 卡，封卡后就变为 M1 卡。

经测试,我用的读卡器配合libnfc相关工具，可以复制UID,CUID卡，其他卡没测试

## libnfc相关工具

对于写过一次的UID卡，再次写入会遇到以下异常

```console
Writing 64 blocks |nfc_initiator_transceive_bytes: Mifare Authentication Error
nfc_initiator_transceive_bytes: Mifare Authentication Error
nfc_initiator_transceive_bytes: Mifare Authentication Error
nfc_initiator_transceive_bytes: Mifare Authentication Error
nfc_initiator_transceive_bytes: Mifare Authentication Error
nfc_initiator_transceive_bytes: Mifare Authentication Error
nfc_initiator_transceive_bytes: Mifare Authentication Error
nfc_initiator_transceive_bytes: Mifare Authentication Error
nfc_initiator_transceive_bytes: Mifare Authentication Error
```

这是因为空白卡用的KEY都是00000000,FFFFFFFFFFFF,复制过之后KEY就不是默认的了

其中一个最简单的解决办法是重新格式化成空白卡

先用mfoc破解得到dump文件
sudo mfoc -O dump
再用mifare-classic-format命令和dump文件把卡格式化成空白卡
sudo mifare-classic-format dump

注1:mifare-classic-format可以通过以下命令安装
sudo apt install libfreefare-bin

注2:如果你已经知道卡的KEY，可以在mfoc命令里加上-f KnownKeys.txt参数

KnownKeys.txt文件格式如下

```text
B8527F9427A4
013940233313
```

```console
sudo mfoc -f KnownKeys.txt -O dump
sudo mifare-classic-format dump
```

## 合并卡

我有两个卡，一个门禁一个电梯。门禁系统只用到了最后4个扇区，而且还没用到卡ID。另外一个电梯系统恰巧没用到最后4个扇区。

所以我把门禁dump文件最后4个扇区的内容替换到电梯dump文件最后4个扇区。把新dump文件写到卡里就得到了一个合并的卡。

注意: dump文件是1024字节的16进制文件。Windows下编辑可以用HxD编辑器。Linux下可以用bless编辑器。Windows下的notepad++加hexeditor插件打开dump文件显示内容可能是编码有问题。Linux下vi配合:%!xxd也有同样问题。但是用xxd命令直接显示dump文件则正确。

## Linux下往空白卡写dump文件

> [Clone MiFare cards using chinesse UUID writable cards](https://gist.github.com/alphazo/3303282)

- Dump the blank chinese card card to get the keys

```console
# mfoc -f KnownKeys.txt -O blank-chinese.dmp
```

- Dump the Mifare card your want to copy

```console
# mfoc -f KnownKeys.txt -O cardtocopy.dmp
```

- Write the Chinese card with the content of the other card including UUID

```console
# nfc-mfclassic w b cardtocopy.dmp blank-chinese.dmp
or
# nfc-mfclassic w a cardtocopy.dmp blank-chinese.dmp
```

cardtocopy.dmp可以是已经准备好的dump文件

- Check that the card is the same

```console
sudo nfc-list
[sudo] quxr 的密码： 
nfc-list uses libnfc 1.7.1
NFC device: pn532_uart:/dev/ttyUSB0 opened
1 ISO14443A passive target(s) found:
ISO/IEC 14443A (106 kbps) target:
    ATQA (SENS_RES): 00  04  
       UID (NFCID1): 8d  69  08  31  
      SAK (SEL_RES): 08  
```

- Go back to blank card

跟格式化步骤相同，区别在于这个有cardtocopy.dmp文件提供KEY,另外那个是用mfoc现破解

```console
# nfc-mfclassic w b blank-chinese.dmp cardtocopy.dmp
or
# nfc-mfclassic w a blank-chinese.dmp cardtocopy.dmp
```

经比较，这个Linux下的命令与Windows下提供的软件做出来的卡内容相同。
