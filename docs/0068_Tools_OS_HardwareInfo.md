---
Title | Tools OS HardwareInfo
-- | --
Created @ | `2019-02-20T04:32:30Z`
Updated @| `2024-08-19T08:51:53Z`
Labels | ``
Edit @| [here](https://github.com/junxnone/linux/issues/68)

---
# Hardware Information  查看工具

command | description
-- | -- 
lshw | all hardware
dmidecode |  all hardware
lscpu | CPU
fdisk | Disk
smartctl | Disk
df  | Disk
lsblk | block Device
lspci | PCI Device
lsusb | usb Device
inxi |

## BIOS

```
 sudo dmidecode
```
```
 sudo dmidecode -s bios-version
```

```
BIOS Information
        Vendor: American Megatrends International, LLC.
        Version: 5.19
        Release Date: 03/25/2021
        Address: 0xF0000
        Runtime Size: 64 kB
        ROM Size: 0 MB
        Characteristics:
                PCI is supported
                BIOS is upgradeable
                BIOS shadowing is allowed
                Boot from CD is supported
                Selectable boot is supported
                BIOS ROM is socketed
                EDD is supported
                Japanese floppy for NEC 9800 1.2 MB is supported (int 13h)
                Japanese floppy for Toshiba 1.2 MB is supported (int 13h)
                5.25"/360 kB floppy services are supported (int 13h)
                5.25"/1.2 MB floppy services are supported (int 13h)
                3.5"/720 kB floppy services are supported (int 13h)
                3.5"/2.88 MB floppy services are supported (int 13h)
                Print screen service is supported (int 5h)
                Serial services are supported (int 14h)
                Printer services are supported (int 17h)
                CGA/mono video services are supported (int 10h)
                ACPI is supported
                USB legacy is supported
                BIOS boot specification is supported
                Targeted content distribution is supported
                UEFI is supported
        BIOS Revision: 5.19

```

## CPU
```
lscpu
```
```
cat /proc/cpuinfo
```
```
dmidecode --type processor
```

## BaseBoard
```
dmidecode --type baseboard|head 
```
- 支持的最大 `Memory`
```
sudo dmidecode |grep "Maximum Capacity" 
```
```
sudo  dmidecode --type memory |head 
```

## Memory
```
free -h
```
```
sudo dmidecode |grep -A 6 "Memory Device"
```
```
sudo dmidecode -t memory
```
- 查看 `单条内存大小`
```
sudo dmidecode --type memory |grep "Size"
```
- 查看 `内存型号`
```
dmidecode --type memory |grep "Part Number"
```
- 可以扩展内存条数目
```
dmidecode --type memory |grep "Number Of Devices"
```

## Disk
```
sudo fdisk -l|grep Disk
```
```
lsblk
```
```
df -h
```
```
sudo smartctl --all /dev/sda
```

```
smartctl 7.2 2020-12-30 r5155 [x86_64-linux-6.5.0-44-generic] (local build)
Copyright (C) 2002-20, Bruce Allen, Christian Franke, www.smartmontools.org

=== START OF INFORMATION SECTION ===
Device Model:     Samsung SSD 870 EVO 1TB
Serial Number:    S75DNX0W810483F
LU WWN Device Id: 5 002538 f43810ed1
Firmware Version: SVT03B6Q
User Capacity:    1,000,204,886,016 bytes [1.00 TB]
Sector Size:      512 bytes logical/physical
Rotation Rate:    Solid State Device
Form Factor:      2.5 inches
TRIM Command:     Available, deterministic, zeroed
Device is:        Not in smartctl database [for details use: -P showall]
ATA Version is:   ACS-4 T13/BSR INCITS 529 revision 5
SATA Version is:  SATA 3.3, 6.0 Gb/s (current: 6.0 Gb/s)
Local Time is:    Tue Aug 20 08:54:37 2024 CST
SMART support is: Available - device has SMART capability.
SMART support is: Enabled

......
```



```
sudo smartctl --all /dev/sda -d megaraid,N
```
> N is 0,1...


## PCI
```
lspci
```

## USB
```
lsusb
```

# inxi
```
inxi -F
System:    Host: xxxx Kernel: 4.15.0-64-generic x86_64 bits: 64 Console: tty 10
           Distro: Ubuntu 18.04.2 LTS
Machine:   Device: server System: Dell product: PowerEdge T630 serial: N/A
           Mobo: Dell model: 0NT78X v: A10 serial: N/A BIOS: Dell v: 2.8.0 date: 05/23/2018
CPU(s):    2 10 core Intel Xeon E5-2630 v4s (-MT-MCP-SMP-) cache: 51200 KB
           clock speeds: max: 3100 MHz 1: 1201 MHz 2: 1202 MHz 3: 1201 MHz 4: 1202 MHz 5: 1203 MHz 6: 1198 MHz
           7: 1200 MHz 8: 1199 MHz 9: 1201 MHz 10: 1198 MHz 11: 1201 MHz 12: 1205 MHz 13: 1204 MHz 14: 1199 MHz
           15: 1463 MHz 16: 1199 MHz 17: 1201 MHz 18: 1199 MHz 19: 1202 MHz 20: 1199 MHz 21: 1198 MHz
           22: 1198 MHz 23: 1200 MHz 24: 1198 MHz 25: 1198 MHz 26: 1199 MHz 27: 1200 MHz 28: 1198 MHz
           29: 1199 MHz 30: 1210 MHz 31: 1199 MHz 32: 1198 MHz 33: 1198 MHz 34: 1199 MHz 35: 1602 MHz
           36: 1198 MHz 37: 1198 MHz 38: 1198 MHz 39: 1198 MHz 40: 1199 MHz
Graphics:  Card-1: NVIDIA GP102 [GeForce GTX 1080 Ti]
           Card-2: NVIDIA GP102 [GeForce GTX 1080 Ti]
           Card-3: Matrox Systems G200eR2
           Card-4: NVIDIA GP102 [GeForce GTX 1080 Ti]
           Card-5: NVIDIA GP102 [GeForce GTX 1080 Ti]
           Display Server: N/A driver: (unloaded: nvidia) tty size: 189x49 Advanced Data: N/A out of X
Audio:     Card 4x NVIDIA GP102 HDMI Audio Controller
           driver: snd_hda_intelsnd_hda_intelsnd_hda_intelsnd_hda_intel
           Sound: Advanced Linux Sound Architecture v: k4.15.0-64-generic
Network:   Card-1: Intel I350 Gigabit Network Connection driver: igb
           IF: eno1 state: up speed: 1000 Mbps duplex: full mac: d0:94:66:67:16:0e
           Card-2: Intel I350 Gigabit Network Connection driver: igb
           IF: eno2 state: down mac: d0:94:66:67:16:0f
Drives:    HDD Total Size: 20402.9GB (51.5% used)
           ID-1: /dev/sdc model: SSDSC2KB960G8R size: 960.2GB
           ID-2: /dev/sde model: PERC_H330_Adp size: 479.6GB
           ID-3: /dev/sdd model: SSDSC2KB960G8R size: 960.2GB
           ID-4: /dev/sda model: ST8000NM0185 size: 8001.6GB
           ID-5: /dev/sdb model: ST8000NM0185 size: 8001.6GB
           ID-6: /dev/sdf model: PERC_H330_Adp size: 1999.8GB
Partition: ID-1: / size: 439G used: 281G (68%) fs: ext4 dev: /dev/sde2
RAID:      No RAID devices: /proc/mdstat, md_mod kernel module present
Sensors:   System Temperatures: cpu: 25.0C mobo: N/A
           Fan Speeds (in rpm): cpu: N/A
Info:      Processes: 590 Uptime: 7 days Memory: 10682.5/273860.6MB Init: systemd runlevel: 5
           Client: Shell (bash) inxi: 2.3.56
```

## Reference
- [在Linux上查询物理机信息-不用去拆机器了](https://www.cnblogs.com/operationhome/p/12486702.html)

