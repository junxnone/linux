---
Title | Tools Media mediainfo
-- | --
Created @ | `2018-10-16T02:24:52Z`
Updated @| `2023-03-08T09:43:04Z`
Labels | ``
Edit @| [here](https://github.com/junxnone/linux/issues/53)

---
# MediaInfo

- 查看多媒体信息的工具
- 支持格式化输出
  - HTML
  - XML
  - JSON
  - ...
- 

## Install on Ubuntu

```
sudo apt install mediainfo
sudo apt install mediainfo-gui
```

## UseCase

```
$ mediainfo 0.png 
General
Complete name                            : 0.png
Format                                   : PNG
Format/Info                              : Portable Network Graphic
File size                                : 6.39 KiB

Image
Format                                   : PNG
Format/Info                              : Portable Network Graphic
Format_Compression                       : LZ77
Width                                    : 96 pixels
Height                                   : 97 pixels
Bit depth                                : 32 bits
Compression mode                         : Lossless
Stream size                              : 6.39 KiB (100%)
```

### 查看 mediatrace info

```
$ mediainfo -f   --Details=1 0.png 
0000 -------------------------
0000 ---   PNG, accepted   ---
0000 -------------------------
0000 File header (8 bytes)
0000  Signature:                             2303741511 (0x89504E47)
0004  ByteOrder:                             218765834 (0x0D0A1A0A)
0008 IHDR - Image header (21 bytes)
0008  Header (8 bytes)
0008   Length:                               13 (0x0000000D)
000C   Chunk Type:                           IHDR
0010  Width:                                 96 (0x00000060)
0014  Height:                                97 (0x00000061)
0018  Bit depth:                             8 (0x08)
0019  Colour type:                           6 (0x06) - Truecolour with alpha
001A  Compression method:                    0 (0x00)
001B  Filter method:                         0 (0x00)
001C  Interlace method:                      0 (0x00)
001D ------------------------
001D ---   PNG, filling   ---
001D ------------------------
001D -------------------------
001D ---   PNG, finished   ---
001D -------------------------
001D CRC:                                    700753053 (0x29C4A49D)
```


### `-f` 查看更全面的信息

```
$ mediainfo -f 0.png 
General
Count                                    : 331
Count of stream of this kind             : 1
Kind of stream                           : General
Kind of stream                           : General
Stream identifier                        : 0
Count of image streams                   : 1
Image_Format_List                        : PNG
Image_Format_WithHint_List               : PNG
Codecs Image                             : PNG
Complete name                            : 0.png
File name extension                      : 0.png
File name                                : 0
File extension                           : png
Format                                   : PNG
Format                                   : PNG
Format/Info                              : Portable Network Graphic
Format/Extensions usually used           : png pns
Commercial name                          : PNG
Internet media type                      : image/png
File size                                : 6540
File size                                : 6.39 KiB
File size                                : 6 KiB
File size                                : 6.4 KiB
File size                                : 6.39 KiB
File size                                : 6.387 KiB
Stream size                              : 0
Stream size                              : 0.00 Byte (0%)
Stream size                              :  Byte0
Stream size                              : 0.0 Byte
Stream size                              : 0.00 Byte
Stream size                              : 0.000 Byte
Stream size                              : 0.00 Byte (0%)
Proportion of this stream                : 0.00000
File last modification date              : UTC 2022-07-15 08:06:30
File last modification date (local)      : 2022-07-15 16:06:30

Image
Count                                    : 124
Count of stream of this kind             : 1
Kind of stream                           : Image
Kind of stream                           : Image
Stream identifier                        : 0
Format                                   : PNG
Format                                   : PNG
Format/Info                              : Portable Network Graphic
Commercial name                          : PNG
Format_Compression                       : LZ77
Internet media type                      : image/png
Width                                    : 96
Width                                    : 96 pixels
Height                                   : 97
Height                                   : 97 pixels
Bit depth                                : 32
Bit depth                                : 32 bits
Compression mode                         : Lossless
Compression mode                         : Lossless
Stream size                              : 6540
Stream size                              : 6.39 KiB (100%)
Stream size                              : 6 KiB
Stream size                              : 6.4 KiB
Stream size                              : 6.39 KiB
Stream size                              : 6.387 KiB
Stream size                              : 6.39 KiB (100%)
Proportion of this stream                : 1.00000
```


## Reference
- [MediaInfo Github](https://github.com/MediaArea/MediaInfo)
- [mediaarea - mediainfo](https://mediaarea.net/en/MediaInfo)



