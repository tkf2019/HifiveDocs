## 移植zCore

#### 移植uCore-SMP

-   重现陶天骅学长的工作，进一步了解环境配置和编译流程

#### 移植zCore

-   将[`zCore2Hifive`](https://github.com/tkf2019/zCore2HiFive)克隆到本地，打开zCore

-   在`zCore/src/platform/riscv/consts.rs`中将物理地址的起始位置修改为`0x8000_0000`，并添加对应feature

    ```makefile
    # zCore/src/platform/riscv/consts.rs
    if #[cfg(feature = "board_fu740")] {
        pub const KERNEL_OFFSET: usize = 0xFFFF_FFFF_8000_0000;
        pub const PHYS_MEMORY_BASE: usize = 0x8000_0000;
        pub const PHYS_MEMORY_END: usize = 0xC000_0000;
    }
    # Makefile
    ifeq ($(PLATFORM), fu740)
    	features += board_fu740 link_user_img
    ```

-   在`zCore/src/platform/riscv/entry.rs`中添加boot entry

-   执行以下命令生成镜像

    ```shell
    make riscv-image
    cd zCore
    make MODE=release LINUX=1 ARCH=riscv64 PLATFORM=fu740
    make fu740 MODE=release LINUX=1 ARCH=riscv64 PLATFORM=fu740
    ```

-   在官网下载编译好的[freedom-u-sdk](https://github.com/sifive/freedom-u-sdk/releases/download/2021.09.00/demo-coreip-cli-unmatched-2021.09.00.rootfs.wic.xz)，windows上可以用rufus将其装入SD卡中

-   打开SD卡，将制作好的zCore镜像`zcore-fu740`复制到/boot文件夹下，修改/boot中的配置文件extlinux/extlinux.conf，将默认的image.gz改为zCore镜像的文件名`zcore-fu740`

<img src="../img/img-6.png" style="zoom:50%;" />

-   将SD卡插入主机，连接串口（串口相关参见中文文档）后启动并进入u-boot，执行`printenv`查看`boot_targets`，mmc0为SD卡启动，执行`setenv boot_targets mmc0`，再执行`boot`即可启动zCore