# docker-xubuntu-2404-cn
docker运行的ubuntu2404镜像，使用xrdp进行连接，已做汉化，中文输入法可用

镜像已添加驱动，本人硬件为intel集显加nvidia独显，测试可调用显卡进行视频硬解，也可以通过wine或者steam调用集显或独显玩游戏，amd用户请自行测试

本镜像通过https://github.com/hectorm/docker-xubuntu/tree/v128修改构建，感谢原作者付出

使用方法：

unraid用户直接将templates-user下文件复制到/boot/config/plugins/dockerMan/templates-user/目录，返回webui通过模版添加容器

docker run \
  -d \
  --name=xubuntu-2404 \
  --net=eth0 \
  --ip=192.168.11.91 \
  --pids-limit 0 \
  -e TZ="Asia/Shanghai" \
  -e LANG=zh_CN.UTF-8 \
  -e LANGUAGE=zh_CN:zh \
  -e LC_ALL=zh_CN.UTF-8 \
  -e UNPRIVILEGED_USER_UID=99 \
  -e UNPRIVILEGED_USER_GID=100 \
  -e UNPRIVILEGED_USER_NAME=default \
  -e UNPRIVILEGED_USER_PASSWORD=password \
  -e UMASK=000 \
  -v /mnt/cache/appdata/xubuntu-2404:/config:rw \
  -v /mnt/cache/appdata/fonts/:/usr/share/fonts/win/:rw \
  -v /mnt/user/downloads/:/config/Downloads/:rw \
  -v /mnt/:/mnt/unraid/:rw \
  --device=/dev/dri/card1 \
  --device=/dev/fuse \
  --device=/dev/dri/renderD129 \
  --hostname xubuntu2604 \
  --shm-size=8g \
  --cap-add SYS_ADMIN \
  --cap-add MKNOD \
  --cap-add AUDIT_WRITE \
  --cap-add SYS_NICE \
  --security-opt='seccomp=unconfined' \
  --ipc='host' ghcr.io/qcnwolf/xubuntu:2404-unraid-cn

关于参数：

1.如果网络选择bridge或者host，--hostname xubuntu2604必须删除，否则无法启动。此参数主要是固定容器的主机名，防止在wine下安装夸克网盘，在容器参数修改后主机名变动导致的登录账号失效

2.UNPRIVILEGED_USER_UID，UNPRIVILEGED_USER_GID，这两个参数是xrdp登录使用账户的UID和GID，参数默认是和unraid系统nobody:user相匹配，如需改成别的，请在首次创建时修改

3.UNPRIVILEGED_USER_NAME，UNPRIVILEGED_USER_PASSWORD，这两个参数是xrdp登录使用账户用户名和密码，同样建议首次创建时修改，家目录在容器的/config，注意映射

4.中文字体镜像里已安装，需要使用windows下字体，复制C:\Windows\Fonts到宿主机，并进行映射 -v /mnt/cache/appdata/fonts/:/usr/share/fonts/win/

5.关于显卡的映射，按以下情况配置，镜像是宿主机有集显和独显的情况，默认是使用集显处理rdp输出

根据宿主机硬件情况不同可能需要容器启动后进容器控制台进行操作

sed -i 's/renderD129/renderD128/g' /opt/xrdp/etc/X11/xrdp/xorg.conf

nano /usr/local/bin/container-init

注释掉一行 sed -i 's/renderD128/renderD129/g' /opt/xrdp/etc/X11/xrdp/xorg.conf

A.宿主机只有单显卡：

a.只有intel集显，device映射/dev/dri/card0和/dev/dri/renderD128

执行上方操作，重启容器即可

b.只有nvidia独显，device映射删除/dev/dri/card*和/dev/dri/renderD12*，添加--gpus=all参数

执行上方操作，重启容器即可

B.宿主机同时有intel集显和nvidia独显，在宿主机驱动正常可调用的情况下，一般独显是card0和renderD128，集显是card1和renderD129，根据自己实际情况判断

a.只使用intel集显，device映射集显的/dev/dri/card1和/dev/dri/renderD129，模版默认就是

b.只使用nvidia独显，device映射删除/dev/dri/card*和/dev/dri/renderD12*，添加--gpus=all参数

执行上方操作，重启容器即可

c.同时使用集显和独显，device映射集显的/dev/dri/card1和/dev/dri/renderD129，同时添加--gpus=all参数

如需切换到N卡独显处理rdp输出，执行上方操作，重启容器即可

6.此模版镜像不含wine，如需要可执行自行安装或者使用镜像 ghcr.io/qcnwolf/xubuntu:2404-cn

apt update&&apt install winehq-stable winetricks

如使用wine玩游戏时无法调用独显，请通过winetricks安装dxvk或直接使用steam的proton

apt update&&apt install steam

7.镜像安装了非官方源的三方软件，不需要可直接卸载

apt purge com.xunlei.download bitcomet xnview && apt autoremove
