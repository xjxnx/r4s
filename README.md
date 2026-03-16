# r4s
镜像扩容方案a（只有squashfs固件有效）
进入刚才新建的目录(目录名改为自己的目录名)   cd  /mnt/usb1_6-5
解压缩(文件名也要改为自己的文件名)       gzip -kd immortalwrt.img.gz
给img镜像末尾填充1GB空数据，bs=1M表示单位，count=1000 表示1000个1M就是1G，这个数字大家可以根据需要随意修改。   dd if=/dev/zero bs=1M count=1000 >> immortalwrt.img
为了将空间真正赋予具体的分区，使用分区工具parted操作映像。这条指令执行时间很长，一定要耐心等到终端上出现（parted），才能执行后面的指令。    parted immortalwrt.img
查看分区情况。注意：如果映像为efi启动格式的话，中间会有两次提示，按要求回答OK和Fix即可。                                            print
调整分区大小，将第 2 个分区扩展至映像文件的 100%                                               resizepart 2 100%
再次查看分区情况。正常情况下这时第二个分区会增加1G容量                                    print
退出分区工具                                                                          quit
把映像重新压缩为gz格式（压缩之前要把之前上传的旧的映像压缩包删除）                     gzip -k immortalwrt.img

######################################################################
在系统中扩容方案b
进入软件仓库，安装 diskman工具。                       软件包名称：luci-i18n-diskman-zh-cn 
EXT4格式:（扩容系统根目录）
配置openwrt网络。
更新软件商店。
安装 diskman 工具。
在挂载点设置中，禁用“自动挂载”选项。
创建一个 2GB 大小的新分区（根据需求调整大小），并将其格式化为 ext4 文件系统。
将新分区挂载为系统根目录。
复制提示的命令到记事本中。
将命令中的 /dev/sda1 替换为实际的分区名称。
在 SSH 中执行替换后的命令，回车运行直到完成。
重启openwrt


SQUASHFS格式：（扩容overlay分区）
配置openwrt网络。
更新软件商店。
安装 diskman 工具。
在挂载点设置中，禁用“自动挂载”选项。
创建一个 2GB 大小的新分区（根据需求调整大小），并将其格式化为 ext4 文件系统。
将新分区挂载为/mnt/sda3 （目的是拷贝overlay文件用，sda3根据自己情况随意命名）
SSH连接openwrt后执行命令：cp -r /overlay/* /mnt/sda3 （将原overlay下的配置拷贝至新分区）
查看新分区目录是否拷贝成功
删除刚刚的/mnt/sda3 挂载
将新分区重新挂载为overlay
重启openwrt

################################################################################
# 1、解压 ImmortalWrt 镜像压缩包
# 把 .img.gz 文件解压成 .img 镜像文件
gzip -d immortalwrt.img.gz

# 2、给镜像文件增加 8GB 空间
# 只是扩大镜像文件大小，不会真正写入数据
truncate -s +8G immortalwrt.img

# 3、打开 parted 分区工具，修改镜像里的分区
parted immortalwrt.img

# 4、把第3个分区（rootfs）扩展到镜像末尾
resizepart 3 100%

# 5、查看当前分区信息，确认扩容成功
print

# 6、退出 parted
quit

# 7、重新压缩镜像文件
# 把 img 再压缩成 img.gz，方便保存或下载
gzip /root/immortalwrt-24.10.5-876-5.28.img
