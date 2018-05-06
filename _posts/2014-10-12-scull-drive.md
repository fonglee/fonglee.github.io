---
layout: post
title: scull 驱动模块
excerpt: "scull 驱动模块"
modified: 2014-10-12
tags: [c]
comments: true
image:
  feature: sample-image-4.jpg
  credit: WeGraphics
  creditlink: http://wegraphics.net/downloads/free-ultimate-blurred-background-pack/
---

# 驱动程序
  
    scull_debug.c

其中旧版linux中的`init_mutex`函数已经被废除了，新版本使用`sema_init`函数
    
    scull_debug.h

    Makefile


# 测试程序

    scull_test：
 
    Makefile：




## 输入命令行，挂载设备

    insmod scull_debug.ko scull_nr_devs=1 scull_quantum=6 scull_qset=2

## 输入：
    cat  /proc/devices
## 输出：
    Character devices:
      1 mem
      4 /dev/vc/0
      4 tty
      4 ttyS
      5 /dev/tty
      5 /dev/console
      5 /dev/ptmx
      5 ttyprintk
      6 lp
      7 vcs
    10 misc
    13 input
    14 sound
    21 sg
    29 fb
    99 ppdev
    108 ppp
    116 alsa
    128 ptm
    136 pts
    180 usb
    189 usb_device
    216 rfcomm
    226 drm
    250 scull
    251 hidraw
    252 usbmon
    253 bsg
    254 rtc

    Block devices:
      1 ramdisk
      2 fd
    259 blkext
      7 loop
      8 sd
      9 md
    11 sr
    65 sd
    66 sd
    67 sd
    68 sd
    69 sd
    70 sd
    71 sd
    128 sd
    129 sd
    130 sd
    131 sd
    132 sd
    133 sd
    134 sd
    135 sd
    252 device-mapper
    253 virtblk
    254 mdp

## 输入命令行，建立设备文件scull0
    mknod /dev/scull0 c 250 0
## 运行测试程序：
    ./scull_test 
## 返回
    write error! code=6
    write error! code=6
    write error! code=6
    write ok! code=2
    read error! code=6
    read error! code=6
    read error! code=6
    read ok! code=2
    [0]=0 [1]=1 [2]=2 [3]=3 [4]=4
    [5]=5 [6]=6 [7]=7 [8]=8 [9]=9
    [10]=10 [11]=11 [12]=12 [13]=13 [14]=14
    [15]=15 [16]=16 [17]=17 [18]=18 [19]=19

## 输入：
    ls /proc/
## 返回：
    1      1146   15533  1995  2344  2467  2605  356   50  68    979          filesystems    mounts         timer_list
    10     1157   15667  2     2348  2468  2623  372   51  7     99           fs             mpt            timer_stats
    100    1158   15720  2051  2349  2493  2638  373   52  7071  995          interrupts     mtrr           tty
    1012   1167   15831  21    2350  2497  2642  38    53  71    acpi         iomem          net            uptime
    1023   12     15909  22    2352  25    2649  39    54  7212  asound       ioports        pagetypeinfo   version
    1043   120    16     2250  2370  2501  2654  40    55  745   buddyinfo    irq            partitions     version_signature
    10669  13     16021  2261  2373  2506  2659  41    56  746   bus          kallsyms       sched_debug    vmallocinfo
    1077   1341   16495  2296  2374  2520  27    42    57  791   cgroups      kcore          schedstat      vmstat
    10781  1364   16651  2299  2381  2552  2714  43    58  8     cmdline      key-users      scsi           zoneinfo
    1084   1379   16766  23    2383  2554  2737  44    59  830   consoles     kmsg           scullmem
    11     13807  16877  2300  2393  2564  2742  4446  6   9     cpuinfo      kpagecount     scullseq
    1102   1383   16967  2312  24    2566  2754  4454  60  916   crypto       kpageflags     self
    1103   13918  17     2320  2408  2568  2771  45    61  917   devices      latency_stats  slabinfo
    1108   14     17059  2322  2433  258   2772  457   62  951   diskstats    loadavg        softirqs
    11317  14509  1794   2328  2439  2586  2776  46    63  961   dma          locks          stat
    1132   1462   18     2332  2440  2587  2777  461   64  967   dri          mdstat         swaps
    1135   14642  1857   2341  2448  2588  2957  47    65  972   driver       meminfo        sys
    1137   15     1862   2342  2450  26    3     48    66  973   execdomains  misc           sysrq-trigger
    11450  15532  1980   2343  2466  260   35    49    67  974   fb           modules        sysvipc

## 输入：
    cat /proc/scullmem
## 输出：
    Device 0: qset 2, q 6, sz 20
      item at f1976868, qset at f1976ff0
      item at f1976498, qset at f1976650
           0: f1976cc8
           1: f1976310

## 输入：
    cat /proc/scullseq
## 输出：
    Device 0: qset 2, q 6, sz 20
      item at f1976868, qset at f1976ff0
      item at f1976498, qset at f1976650
           0: f1976cc8
           1: f1976310

## 重新执行
    ./scull_test
## 输出：
    write error! code=6
    write error! code=6
    write error! code=6
    write ok! code=2
    read error! code=6
    read error! code=6
    read error! code=6
    read ok! code=2
    [0]=0 [1]=1 [2]=2 [3]=3 [4]=4
    [5]=5 [6]=6 [7]=7 [8]=8 [9]=9
    [10]=10 [11]=11 [12]=12 [13]=13 [14]=14
    [15]=15 [16]=16 [17]=17 [18]=18 [19]=19

## 输入：
    cat /proc/scullmem
## 输出：
    Device 0: qset 2, q 6, sz 20
      item at f1976498, qset at f1976650
      item at f1976868, qset at f1976ff0
           0: f1976720
           1: f1976828

## 输入：
    cat /proc/scullseq
## 输出：
    Device 0: qset 2, q 6, sz 20
      item at f1976498, qset at f1976650
      item at f1976868, qset at f1976ff0
           0: f1976720
           1: f1976828

# 分析
    insmod scull_debug.ko scull_nr_devs=1 scull_quantum=6 scull_qset=2
将调用驱动程序文件`scull_debug.c`里面的`int scull_init_module(void)`函数，此函数里面将执行，`result = alloc_chrdev_region(&dev, scull_minor, scull_nr_devs, "scull")`;此函数为向内核注册驱动程序，设备号动态分配，dev返回分配的设备编号，`scull_minor`为要使用的被请求的第一个此设备号为0，`scul_nr_devs`为所请求设备编号的个数，“scull”为设备的名称，它将出现在/proc/devices 和sysfs中。
`scull_major = MAJOR(dev)`;获得设备的主设备号
`scull_devices = kmalloc(scull_nr_devs * sizeof(struct scull_dev), GFP_KERNEL); 此函数调用将试图分配scull_nr_devs * sizeof(struct scull_dev)`个字节大小的内存，其返回值指向该内存的指针，分配失败返回NULL。
`memset(scull_devices, 0, scull_nr_devs * sizeof(struct scull_dev));` 填充0

          scull_devices[0].quantum =6
          scull_devices[0].qset = 2;
          init_MUTEX(&scull_devices[0].sem);信息量互斥锁
          scull_setup_cdev(&scull_devices[0], 0);

## scull_setup_cdev：

`devno = MKDEV(scull_major, scull_minor + index)`;将主设备号和此设备号转换为dev_t型
`cdev_init(&dev->cdev, &scull_fops)`;将cdev结构嵌入到自己的设备特定结构中，初始化
`err = cdev_add (&dev->cdev, devno, 1); `告诉内核该结构的信息，devno设备对应的第一个编号，1表示和该设备关联的设备编号的数量

`sculltest = open("/dev/scull0",O_WRONLY );`将调用驱动程序中的`int scull_open(struct inode *inode, struct file *filp)``

其中 
`dev = container_of(inode->i_cdev, struct scull_dev, cdev); inode->i_cdev`表示字符设备内核的内部结构，该cdev结构包含在`struct scull_dev`结构中，然后返回包含该字段的scull_dev结构指针。

测试程序中
`write(sculltest , &buffer1[0] , 20) `会调用驱动程序中的
`ssize_t scull_write(struct file *filp, const char __user *buf, size_t count,loff_t *f_pos)`函数, *buf对应&buffer1[20-i]，count对应i

    quantum=6，qset=2
    itemsize=12
    item=*f_pos/20，一开始*f_pos=0，所以item=0
    rest=0
    s_pos=0/6=0
    q_pos=0/6=0

`scull_follow(dev,0);` 一个沿链表前行得到正确`scull_set`指针的函数
其实这个函数的实质是：如果已经存在这个`scull_set`，就返回这个`scull_set`的指针。如果不存在这个`scull_set`，一边沿链表为`scull_set`分配空间一边沿链表前行，直到所需要的`scull_set`被分配到空间并初始化为止，就返回这个`scull_set`的指针。

`if (count > quantum - q_pos)
          count = quantum - q_pos;
count=20 > 6-0 `所以count=6

`if (copy_from_user(dptr->data[s_pos]+q_pos, buf, count)) {
          retval = -EFAULT;
          goto out;
}`
即`copy_from_user(dptr->data[0]+0, &buffer1[20-i], 6) `实现内核与用户空间之间的数据拷贝

`*f_pos += count;  *f_pos=6`

继续写
    
    write(sculltest , &buffer1[6] ,14) `

    item = 0
    rest=6

    s_pos=6/6=1
    q_pos=6%6=0

    scull_follow(dev, 0);

    count =6 - 0=6

    copy_from_user(dptr->data[1]+0, &buffer1[20-i], 6)
    *f_pos=12
然后继续执行
    
    write(sculltest , &buffer1[12] ,8) 

    item = 1
    rest=0

    s_pos=0/6=0
    q_pos=0%6=0

    scull_follow(dev, 0);

    count =6 - 0=6

    copy_from_user(dptr->data[0]+0, &buffer1[20-i], 6)
    *f_pos=18
继续执行
    
    write(sculltest , &buffer1[18] ,2) 
    item = 1
    rest=6

    s_pos=6/6=1
    q_pos=6%6=0

    scull_follow(dev, 0);

    count =2

    copy_from_user(dptr->data[1]+0, &buffer1[20-i], 2)
    *f_pos=20
        
执行 `cat /proc/scullseq` 首先会调用
    
    static int scull_proc_open(struct inode *inode, struct file *file)
    {
         return seq_open(file, &scull_seq_ops);
    }
该函数将文件连接到`seq_file`结构 `seq_operations scull_seq_ops`

从而顺序查询device0-device1-·····
首先调用
    
    static void *scull_seq_start(struct seq_file *s, loff_t *pos)
    {
         if (*pos >= scull_nr_devs)
              return NULL;   /* No more to read */
         return scull_devices + *pos;
    }
返回：scull_devices +0
然后`static int scull_seq_show(struct seq_file *s, void *v)`函数 其中*v先前对start或者next调用所返回的`scull_devices + *pos;`
这个参数传给了`static int scull_seq_show(struct seq_file *s, void *v)`

从`create_proc_read_entry("scullmem", 0 /* default mode */,
               NULL /* parent dir */, scull_read_procmem,
               NULL /* client data */);`可以看出
 `cat /proc/scullmem`则会调用`int scull_read_procmem(char *buf, char **start, off_t offset,int count, int *eof, void *data)`函数。






