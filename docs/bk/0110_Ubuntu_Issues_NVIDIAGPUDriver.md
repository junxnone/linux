-----

| Title     | Ubuntu Issues NVIDIAGPUDriver                        |
| --------- | ---------------------------------------------------- |
| Created @ | `2022-05-24T02:51:47Z`                               |
| Updated @ | `2023-01-17T12:35:31Z`                               |
| Labels    | \`\`                                                 |
| Edit @    | [here](https://github.com/junxnone/linux/issues/110) |

-----

# NVIDIA GPU Driver 问题

## Failed to initialize NVML

    Failed to initialize NVML: Driver/library version mismatch

### Solution

  - 1 查看哪些占用了 nvidia mod, kill 掉相关进程

<!-- end list -->

    sudo lsof -n -w /dev/nvidia*

  - 2 卸载动态驱动 module

<!-- end list -->

    sudo rmmod nvidia

  - 3 重新安装官网下载的驱动

## NVIDIA-SMI has failed because ...

    NVIDIA-SMI has failed because it couldn't communicate with the NVIDIA driver. Make sure that the latest NVIDIA driver is installed and running.

  - Ubuntu会自动更新内核，但是nvidia driver
    安装时只是编译安装了当前内核版本的driver，所以更换了内核导致driver启动不了
  - apt 安装的 NVIDIA Driver 也会出现 `nvidia-smi` 不能使用的情况

### Solution

1.  停止自动更新

<!-- end list -->

    vi /etc/apt/apt.conf.d/10periodic
    vi /etc/apt/apt.conf.d/20auto-upgrades

所有值改为 ‘ 0 ’

    APT::Periodic::Update-Package-Lists "0";
    APT::Periodic::Unattended-Upgrade "0";

    APT::Periodic::Update-Package-Lists "0";
    APT::Periodic::Download-Upgradeable-Packages "0";
    APT::Periodic::AutocleanInterval "0";

2.  重装 nvidia driver(手动安装需要)

3.  reboot
