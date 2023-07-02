-----

| Title     | Ubuntu SetDefaultKernel                              |
| --------- | ---------------------------------------------------- |
| Created @ | `2023-03-20T09:21:31Z`                               |
| Updated @ | `2023-03-20T09:22:58Z`                               |
| Labels    | \`\`                                                 |
| Edit @    | [here](https://github.com/junxnone/linux/issues/114) |

-----

# 设置默认 Kernel

  - **OS**: Ubuntu 22.04

## 查看安装了哪些 Kernel

  - 查看文件 `/boot/grub/grub.cfg`

<!-- end list -->

    cat /boot/grub/grub.cfg | grep menuentry

    $ cat /boot/grub/grub.cfg | grep menuentry
    if [ x"${feature_menuentry_id}" = xy ]; then
      menuentry_id_option="--id"
      menuentry_id_option=""
    export menuentry_id_option
    menuentry 'Ubuntu' --class ubuntu --class gnu-linux --class gnu --class os $menuentry_id_option 'gnulinux-simple-a103aef9-a8e9-4ac1-82f8-1dd8440c9ed1' {
    submenu 'Advanced options for Ubuntu' $menuentry_id_option 'gnulinux-advanced-a103aef9-a8e9-4ac1-82f8-1dd8440c9ed1' {
            menuentry 'Ubuntu, with Linux 5.19.0-35-generic' --class ubuntu --class gnu-linux --class gnu --class os $menuentry_id_option 'gnulinux-5.19.0-35-generic-advanced-a103aef9-a8e9-4ac1-82f8-1dd8440c9ed1' {
            menuentry 'Ubuntu, with Linux 5.19.0-35-generic (recovery mode)' --class ubuntu --class gnu-linux --class gnu --class os $menuentry_id_option 'gnulinux-5.19.0-35-generic-recovery-a103aef9-a8e9-4ac1-82f8-1dd8440c9ed1' {
            menuentry 'Ubuntu, with Linux 5.17.0-1020-oem' --class ubuntu --class gnu-linux --class gnu --class os $menuentry_id_option 'gnulinux-5.17.0-1020-oem-advanced-a103aef9-a8e9-4ac1-82f8-1dd8440c9ed1' {
            menuentry 'Ubuntu, with Linux 5.17.0-1020-oem (recovery mode)' --class ubuntu --class gnu-linux --class gnu --class os $menuentry_id_option 'gnulinux-5.17.0-1020-oem-recovery-a103aef9-a8e9-4ac1-82f8-1dd8440c9ed1' {
            menuentry 'Ubuntu, with Linux 5.15.0-67-generic' --class ubuntu --class gnu-linux --class gnu --class os $menuentry_id_option 'gnulinux-5.15.0-67-generic-advanced-a103aef9-a8e9-4ac1-82f8-1dd8440c9ed1' {
            menuentry 'Ubuntu, with Linux 5.15.0-67-generic (recovery mode)' --class ubuntu --class gnu-linux --class gnu --class os $menuentry_id_option 'gnulinux-5.15.0-67-generic-recovery-a103aef9-a8e9-4ac1-82f8-1dd8440c9ed1' {
    menuentry 'UEFI Firmware Settings' $menuentry_id_option 'uefi-firmware' {

## 设置 第 N 个 Kernel

  - `/etc/default/grub`

<!-- end list -->

    GRUB_DEFAULT="1> 0"

> **1**: 第一个菜单 **0**: 第二个菜单第一项

  - 更改 `0` 对应的内核,即 `submenu` 中 `menuentry` 对应的位置

## 更新 Grub

    sudo update-grub

## Reboot

    sudo reboot

## Reference

  - [Ubuntu安装、卸载和更换默认kernel](https://www.cnblogs.com/ArsenalfanInECNU/p/16952333.html)
