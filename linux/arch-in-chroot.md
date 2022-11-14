# chroot 环境中运行 archlinux

## 同 CPU 架构下

有些服务器运行的 Linux 的版本很老，升级 glibc 不方便。假如拥有 root 权限，可以新建一个 archlinux 环境，并 chroot 进入这个新环境。

1. 下载 archlinux bootstrap，例如从 aliyun 镜像中[下载](https://mirrors.aliyun.com/archlinux/iso/latest/archlinux-bootstrap-x86_64.tar.gz)。
2. 为了正确设置文件权限，用 **root 账号**解压，并注意额外的参数。解压后，修改archlinux 软件仓库 `root.x86_64/etc/pacman.d/mirrorlist`。
 
    ```bash
    sudo tar xzf archlinux-bootstrap-x86_64.tar.gz --numeric-owner
    ```

3. root 账号运行下面命令，挂载目录并进入环境

    ```bash
    # 解决 pacman 运行失败的问题，详细信息看 wiki
    mount --bind ./root.x86_64 ./root.x86_64 
    pushd ./root.x86_64
    cp /etc/resolv.conf etc
    mount -t proc /proc proc
    mount --make-rslave --rbind /sys sys
    mount --make-rslave --rbind /dev dev
    mount --make-rslave --rbind /run run    # (assuming /run exists on the system)
    #mount --rbind /tmp tmp    # gentoo wiki 也挂载了 tmp
    popd
    chroot ./root.x86_64 /bin/bash
    ```
4. 初始化 pacman。
    ```bash
    pacman-key --init
    pacman-key --populate
    ```

5. 环境中创建账号，后续入口命令改为 su。
    ```bash
    useradd -m -G wheel -s /bin/zsh -u 1234 zhou # uid 和主环境一样，如果某些group有问题，也许把id改成一样的
    chroot ./root.x86_64 su - zhou
    ```

6. 最终脚本。
    ```bash
    mount --bind ./root.x86_64 ./root.x86_64
    pushd ./root.x86_64
    cp /etc/resolv.conf etc
    mount -t proc /proc proc
    mount --make-rslave --rbind /sys sys
    mount --make-rslave --rbind /dev dev
    mount --make-rslave --rbind /run run
    mount --rbind /tmp tmp
    popd
    chroot ./root.x86_64 su - zhou
    pushd ./root.x86_64
    umount proc
    umount sys
    umount dev
    umount run
    umount tmp
    popd
    ```

## 不同 CPU 架构下
M1 MacBookPro 下，运行 ARM 版本 Linux，然后在 Linux 中可以利用 chroot 运行 x86 版本 archlinux。

和上面类似，不过 host 中安装 qemu-xxx-static，将二进制文件复制到 chroot 环境中。入口命令改成 

``` bash
chroot /path qemu-xxx-static /bin/bash
```

## TODO:
ssh chrootdirectory
https://www.tecmint.com/restrict-ssh-user-to-directory-using-chrooted-jail/
https://digilent.com/blog/using-the-chroot-ssh-trick/
https://www.debian.org/doc/manuals/securing-debian-manual/chroot-ssh-env.en.html
https://www.techrepublic.com/article/chroot-users-with-openssh-an-easier-way-to-confine-users-to-their-home-directories/


## 参考
* [Archlinux Installation Wiki](https://wiki.archlinux.org/title/Install_Arch_Linux_from_existing_Linux)

* [chroot with qemu static](https://unix.stackexchange.com/questions/41889/how-can-i-chroot-into-a-filesystem-with-a-different-architechture)
