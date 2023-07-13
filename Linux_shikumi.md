
# ［試して理解］Linuxのしくみ　―実験と図解で学ぶOS、仮想マシン、コンテナの基礎知識【増補改訂版】

## 第1章 Linuxの概要

### システムコールの発行

事前に必要なパッケージを導入
いつも使用しているユーザーを特定のグループに追加

```
ishii_tdd@PCS27515:~/hoge02/Linux_shikumi/1-1$ sudo apt update && sudo apt install binutils build-essential golang sysstat python3-matplotlib
ishii_tdd@PCS27515:~/hoge02/Linux_shikumi/1-1$ sudo apt update && sudo apt install python3-pil fonts-takao fio qemu-kvm virt-manager libvirt-clients virtinst jq docker.io containerd libvirt-daemon-system

ishii_tdd@PCS27515:~/hoge02/Linux_shikumi$ ls
ishii_tdd@PCS27515:~/hoge02/Linux_shikumi$ sudo adduser `id -un` libvirt
The user `ishii_tdd' is already a member of `libvirt'.
ishii_tdd@PCS27515:~/hoge02/Linux_shikumi$ sudo adduser `id -un` libvirt-qemu
Adding user `ishii_tdd' to group `libvirt-qemu' ...
Adding user ishii_tdd to group libvirt-qemu
Done.
ishii_tdd@PCS27515:~/hoge02/Linux_shikumi$ sudo adduser `id -un` kvm
Adding user `ishii_tdd' to group `kvm' ...
Adding user ishii_tdd to group kvm
Done.
```

hello.goを実行して、プロセスがどんなシステムコールを発行するのか、straceコマンドによって確認する。

```
ishii_tdd@PCS27515:~/hoge02/Linux_shikumi/1-1$ cat hello.go
package main

import (
        "fmt"
)

func main() {
        fmt.Println("hello world")
}
```

```
ishii_tdd@PCS27515:~/hoge02/Linux_shikumi/1-1$ go build hello.go
ishii_tdd@PCS27515:~/hoge02/Linux_shikumi/1-1$ ls
hello  hello.go
ishii_tdd@PCS27515:~/hoge02/Linux_shikumi/1-1$ ./hello
hello world
ishii_tdd@PCS27515:~/hoge02/Linux_shikumi/1-1$ strace -o hello.log ./hello
hello world
ishii_tdd@PCS27515:~/hoge02/Linux_shikumi/1-1$ ls
hello  hello.go  hello.log
ishii_tdd@PCS27515:~/hoge02/Linux_shikumi/1-1$ cat hello.log
execve("./hello", ["./hello"], 0x7ffcec4031e0 /* 35 vars */) = 0
arch_prctl(ARCH_SET_FS, 0x55e0f0)       = 0
sched_getaffinity(0, 8192, [0, 1, 2, 3, 4, 5, 6, 7]) = 32
```

他にもlsコマンド
```
ishii_tdd@PCS27515:~/hoge02/Linux_shikumi/1-1$ strace -o ls.log ls
hello  hello.go  hello.log  hello.py  hello.py.log  ls.log
ishii_tdd@PCS27515:~/hoge02/Linux_shikumi/1-1$ cat ls.log
execve("/usr/bin/ls", ["ls"], 0x7ffe1c1e6e30 /* 35 vars */) = 0
brk(NULL)                               = 0x558e386ad000
arch_prctl(0x3001 /* ARCH_??? */, 0x7ffe981079f0) = -1 EINVAL (Invalid argument)
access("/etc/ld.so.preload", R_OK)      = -1 ENOENT (No such file or directory)
openat(AT_FDCWD, "/etc/ld.so.cache", O_RDONLY|O_CLOEXEC) = 3
```

sarコマンド

```
ishii_tdd@PCS27515:~/hoge02/Linux_shikumi/1-1$ sar -P 0 2 10
Linux 5.15.90.1-microsoft-standard-WSL2 (PCS27515)      07/13/2023      _x86_64_        (8 CPU)

10:44:21 AM     CPU     %user     %nice   %system   %iowait    %steal     %idle
10:44:23 AM       0      0.00      0.00      0.00      0.00      0.00    100.00
10:44:25 AM       0      0.00      0.00      0.00      0.00      0.00    100.00
10:44:27 AM       0      0.00      0.00      0.00      0.00      0.00    100.00
10:44:29 AM       0      0.00      0.00      0.00      0.00      0.00    100.00
10:44:31 AM       0      0.00      0.00      0.00      0.00      0.00    100.00
10:44:33 AM       0      0.00      0.00      0.00      0.00      0.00    100.00
10:44:35 AM       0      0.00      0.00      0.00      0.00      0.00    100.00
10:44:37 AM       0      0.00      0.00      0.50      0.00      0.00     99.50
10:44:39 AM       0      0.00      0.00      0.00      0.00      0.00    100.00
10:44:41 AM       0      0.00      0.00      0.00      0.00      0.00    100.00
Average:          0      0.00      0.00      0.05      0.00      0.00     99.95
```


### ライブラリ
```
ishii_tdd@PCS27515:~/hoge02/Linux_shikumi/1-1$ ldd /usr/bin/python3
        linux-vdso.so.1 (0x00007fff8ad6c000)
        libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f5026e80000)
        libpthread.so.0 => /lib/x86_64-linux-gnu/libpthread.so.0 (0x00007f5026e5d000)
        libdl.so.2 => /lib/x86_64-linux-gnu/libdl.so.2 (0x00007f5026e57000)
        libutil.so.1 => /lib/x86_64-linux-gnu/libutil.so.1 (0x00007f5026e52000)
        libm.so.6 => /lib/x86_64-linux-gnu/libm.so.6 (0x00007f5026d03000)
        libexpat.so.1 => /lib/x86_64-linux-gnu/libexpat.so.1 (0x00007f5026cd5000)
        libz.so.1 => /lib/x86_64-linux-gnu/libz.so.1 (0x00007f5026cb7000)
        /lib64/ld-linux-x86-64.so.2 (0x00007f5027084000)


ishii_tdd@PCS27515:~/hoge02/Linux_shikumi/1-1$ dpkg-query -W | grep ^lib
libaa1:amd64    1.4p5-46
libaccountsservice0:amd64       0.6.55-0ubuntu12~20.04.6
libacl1:amd64   2.2.53-6
libaio1:amd64   0.3.112-5
libalgorithm-diff-perl  1.19.03-2
libalgorithm-diff-xs-perl       0.04-6
libalgorithm-merge-perl 0.08-3
libapparmor1:amd64      2.13.3-7ubuntu5.2
libappindicator3-1      12.10.1+20.04.20200408.1-0ubuntu1
libappstream4:amd64     0.12.10-2
```


