-----

| Title         | Tools Media image2string                            |
| ------------- | --------------------------------------------------- |
| Created @     | `2020-04-04T05:38:22Z`                              |
| Last Modify @ | `2022-12-21T15:05:24Z`                              |
| Labels        | \`\`                                                |
| Edit @        | [here](https://github.com/junxnone/linux/issues/36) |

-----

# 图像转字符串工具

## Brief

  - 远程 `shell` 查看图片可以把图片转成字符串形式
  - Tools
      - asciiview
      - cacaview
      - img2txt

## asciiview

  - **Install**

<!-- end list -->

    sudo apt-get install aview imagemagick

  - **Control Hotkey**

<!-- end list -->

``` 
       a,w,d,x               Move the image one row/column.
       A,W,D,X              Move the image one page.
       Z,+    Zoom in.
       z,-    Zoom out.
       s      Save image.
       m      Change dithering mode.
       q      Quit the viewer.
       i      Turn inversion on.
       I      Turn inversion off.
       u      Select attributes.
       f      Select font.
       SPACE  Redraw screen.
       .      Increase contrast (dot).
       ,      Decrease contrast (comma).
       '      Increase gamma (single quote).
       ;      Decrease gamma (semicolon).
       >      Increase brightness.
       <      Decrease brightness.
```

### UseCase

  - **查看转码后的效果**

<!-- end list -->

    asciiview your_image.png

  - 按 `s` 保存时可以调整 width 和 height 调整输出 宽高

> 太大会无法正常显示

  - **去掉前后的空格**

<!-- end list -->

``` 
sed 's/^[ ].\{40\}//g;s/[ ].\{40\}$//g' your_output.txt 
```

## cacaview

  - **Install caca-utils**

<!-- end list -->

    sudo apt install caca-utils

  - **cacaview 查看图片**

<!-- end list -->

    cacaview xxxx.png

  - **cacaview 查看文件夹**

<!-- end list -->

    cacaview data/image/*
