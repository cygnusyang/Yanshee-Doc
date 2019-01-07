# 什么是NOOBS
NOOBS is designed to make it easy to select and install operating systems for the Raspberry Pi without having to worry about manually imaging your SD card.

The latest official release of NOOBS can be downloaded from http://downloads.raspberrypi.org/NOOBS_latest

For information on previous releases and version changelists, visit https://github.com/raspberrypi/noobs/releases

详细信息请看: [NOOBS](https://github.com/raspberrypi/noobs)

# 如何发布Yanshee新的NOOBS版本

## 说明
Yanshee的操作系统是基于Raspbian的重新打包．主要改动如下：
- 添加消回声驱动(ZLS38050)
- 添加ROS基础框架
- 添加若干预装软件
- 添加Yanshee-ROS一系列包

## 步骤

1. 烧录Raspbian image．
  - 如果是Linux，使用如下命令烧录系统到SD卡  
  
      ```
      dd if=<raspbian.img> of=/dev/sdX bs=8M
      ```
  - 如果是Windows，请下载对应工具烧录SD卡
2. 安装依赖
  - 下载并执行Yanshee-Build
  使用帮助请参看[README.md](https://10.10.1.34/Yanshee/Yanshee-Build/blob/master/README.md)  
  
      ```
      git clone git@gitlab.ubt.com:Yanshee/Yanshee-Build.git
      ./build.sh
      ```
3. 为NOOBS制作boot.tar.xz及root.tar.xz
  - 创建对应的挂载点  
  
      ```
      sudo mkdir -p /mnt/img1
      sudo mkdir -p /mnt/img2
      ```
  - 查看SD卡分区情况，得到/boot及/root分区

    下面的例子是SD被分配到/dev/sdc，其对应的分区是/boot对应/dev/sdc1, /root对应/dev/sdc2  
    
    ```
    fdisk -l

    Disk /dev/sdc: 14.9 GiB, 15931539456 bytes, 31116288 sectors
    Units: sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disklabel type: dos
    Disk identifier: 0xb5bfe8fa

    Device     Boot Start      End  Sectors  Size Id Type
    /dev/sdc1        8192    98045    89854 43.9M  c W95 FAT32 (LBA)
    /dev/sdc2       98304 31116287 31017984 14.8G 83 Linux
    ```
  - 挂载对应分区到挂载点  
    
    ```
    sudo mount /dev/sdc1 /mnt/img1
    sudo mount /dev/sdc2 /mnt/img2
    ```
  - 创建boot.tar.xz和root.tar.xz  
    
    ```
    cd /mnt/img1
    sudo mkdir -p /devnfs/target/
    bsdtar --numeric-owner --format gnutar -cvpf /devnfs/target/boot.tar .
    xz -9 -ev /devnfs/target/boot.tar

    cd /mnt/img2
    bsdtar --numeric-owner --format gnutar --one-file-system -cpvf /devnfs/target/root.tar .
    xz -9 -ev /devnfs/target/root.tar
    ```
4. 自定义NOOBS
  - 创建os目录  
    <b>请从老版本里面copy对应文件，再做修改.</b>
    
    ```
    os.json                      //系统描述
    partitions.json              //分区描述
    boot.tar                      //boot分区文件
    root.tar                      //root分区文件
    上面四个文件是必须的，缺一不可，其它则是不必须的：
    slides或slides_vga       //文件夹内放说明性图片,安装时在主界面以幻灯片播放,如官方提供则可由marketing.tar解压得https://downloads.raspberrypi.org/
    Logo.png                //Logo名字须与文件夹名字相同，大小40x40
    partition_setup.sh      //分区脚本，在系统安装完成后立即执行，如无则需要在cmdline.txt设定root分区位置
    ```
    os目录结构如下，请创建如下文件．  
    
    ```
    tree .
    .
    └── Raspbian
        ├── blank.tar.xz
        ├── boot.tar.xz
        ├── os.json
        ├── partition_setup.sh
        ├── partitions.json
        ├── Raspbian.png
        ├── release_notes.txt
        ├── root.tar.xz
        ├── slides_vga
        │   ├── A.png
        │   ├── B.png
        │   ├── C.png
        │   ├── D.png
        │   ├── E.png
        │   ├── F.png
        │   ├── G.png
        │   └── Thumbs.db
        └── Thumbs.db
    ```  
  - 设置自动安装
    1. 复制os目录到NOOBS的os目录中
    
    ```
    ls -l os
    total 4
    drwxrwxr-x 3 <user> <usergroup> 4096 Sep 29 09:42 Raspbian    
    ```
    2. 添加flavours.json.　请注意,flavous.json中只添加要自动安装的项.  
      
      ```
      cat flavours.json
      {
        "flavours": [
          {
            "name": "Raspbian",
            "description": "Raspbian for Yanshee"
          }
        ]
      }
      ```  
    3. 在recovery.cmdline文件里后添加silentinstall参数
    
    ```
    cat recovery.cmdline
    runinstaller quiet ramdisk_size=32768 root=/dev/ram0 init=/init vt.cur_default=1 elevator=deadline silentinstall
    ```
