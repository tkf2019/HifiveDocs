#### 移植zCore

-   

-   在`zCore/src/platform/riscv/consts.rs`中将物理地址的起始位置修改为`0x8000_0000`，并在编译时添加对应feature

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

-   打开SD卡，将制作好的zCore镜像`zcore-fu740`复制到/boot文件夹下，修改/boot中的配置文件extlinux/*.conf，将默认的image.gz改为zCore镜像的文件名`zcore-fu740`

-   将SD卡插入主机，连接串口（串口相关参见中文文档）后启动并进入u-boot，执行`printenv`查看`boot_targets`，mmc0为SD卡启动，执行`setenv boot_targets=mmc0`，再执行`boot`即可启动zCore

#### 启动S7

-   OpenSBI首先进行一系列初始化，然后进行二级跳转到uboot，由于uboot运行在S态，无法在S7上加载程序，因此要重新配置OpenSBI
-   论坛上的一个建议：可以通过向S7发送软件中断来唤醒，让S7跳转到事先准备好的中断处理程序中，最后返回到S7上运行的OS中
-   OpenSBI：系统调用；BootLoader
-   U-Boot最新版本有小问题，建议使用2019版本
