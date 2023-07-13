
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

論理CPU0で、無限ループするプログラム inf-loop.py を実行したら、%user が100となっていた。

```
ishii_tdd@PCS27515:~/hoge02/Linux_shikumi/1-1$ cat inf-loop.py
#!/usr/bin/python3
while True:
    pass
ishii_tdd@PCS27515:~/hoge02/Linux_shikumi/1-1$ taskset -c 0 ./inf-loop.py &
[1] 14348
ishii_tdd@PCS27515:~/hoge02/Linux_shikumi/1-1$ sar -P 0 2 3
Linux 5.15.90.1-microsoft-standard-WSL2 (PCS27515)      07/13/2023      _x86_64_        (8 CPU)

10:56:49 AM     CPU     %user     %nice   %system   %iowait    %steal     %idle
10:56:51 AM       0    100.00      0.00      0.00      0.00      0.00      0.00
10:56:53 AM       0    100.00      0.00      0.00      0.00      0.00      0.00
10:56:55 AM       0    100.00      0.00      0.00      0.00      0.00      0.00
Average:          0    100.00      0.00      0.00      0.00      0.00      0.00
ishii_tdd@PCS27515:~/hoge02/Linux_shikumi/1-1$ ps -a
    PID TTY          TIME CMD
   1177 pts/1    00:00:00 bash
  14348 pts/0    00:00:42 inf-loop.py
  14352 pts/0    00:00:00 ps
ishii_tdd@PCS27515:~/hoge02/Linux_shikumi/1-1$ kill 14348
ishii_tdd@PCS27515:~/hoge02/Linux_shikumi/1-1$ ps -a
    PID TTY          TIME CMD
   1177 pts/1    00:00:00 bash
  14353 pts/0    00:00:00 ps
[1]+  Terminated              taskset -c 0 ./inf-loop.py
```

### ライブラリ

Pythonプログラムを実行するとき、libcがリンクされている。つまり内部的に標準Cライブラリが使用されていることが分かる。

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

静的ライブラリと共有ライブラリ
- 容量（サイズ）
- 共有ライブラリがリンクされているかが異なる

```
ishii_tdd@PCS27515:~/hoge02/Linux_shikumi/1-1$ cat pause.c
#include <unistd.h>
int main(void) {
        pause();
        return 0;
}
ishii_tdd@PCS27515:~/hoge02/Linux_shikumi/1-1$ cc -static -o pause pause.c
ishii_tdd@PCS27515:~/hoge02/Linux_shikumi/1-1$ ls -rlt pause
-rwxr-xr-x 1 ishii_tdd ishii_tdd 871832 Jul 13 11:32 pause
ishii_tdd@PCS27515:~/hoge02/Linux_shikumi/1-1$ ldd pause
        not a dynamic executable
ishii_tdd@PCS27515:~/hoge02/Linux_shikumi/1-1$ cc -o pause2 pause.c
ishii_tdd@PCS27515:~/hoge02/Linux_shikumi/1-1$ ls -rlt pause2
-rwxr-xr-x 1 ishii_tdd ishii_tdd 16696 Jul 13 11:32 pause2
ishii_tdd@PCS27515:~/hoge02/Linux_shikumi/1-1$ ldd pause2
        linux-vdso.so.1 (0x00007ffe692eb000)
        libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007fb97a054000)
        /lib64/ld-linux-x86-64.so.2 (0x00007fb97a25d000)
```


## 第2章 プロセス管理（基礎編）

### プロセスの確認

```
ishii_tdd@PCS27515:~/hoge02/Linux_shikumi/1-1$ ps aux | grep ishii_t+
ishii_t+     922  0.0  0.1  16248  9616 pts/0    Ss   09:23   0:00 -bash
ishii_t+    1133  0.0  0.1  19212  9832 ?        Ss   09:23   0:00 /lib/systemd/systemd --user
ishii_t+    1134  0.0  0.0 171724  3716 ?        S    09:23   0:00 (sd-pam)
ishii_t+    1163  0.0  0.1 281404 14048 ?        Ssl  09:23   0:00 /usr/bin/pulseaudio --daemonize=no --log-target=journal
ishii_t+    1177  0.0  0.1  15644  8772 pts/1    S+   09:23   0:00 -bash
ishii_t+    1355  0.0  0.0   7116  4020 ?        Ss   09:23   0:00 /usr/bin/dbus-daemon --session --address=systemd: --nofork --nopidfile --systemd-activation --syslog-only
ishii_t+    1522  0.0  0.2  74260 20896 ?        S    09:23   0:00 fcitx
ishii_t+    1534  0.0  0.0   7116  3044 ?        Ss   09:23   0:00 /usr/bin/dbus-daemon --syslog --fork --print-pid 4 --print-address 6 --config-file /usr/share/fcitx/dbus/daemon.conf
ishii_t+    1538  0.0  0.0   5392   200 ?        SN   09:23   0:00 /usr/bin/fcitx-dbus-watcher unix:abstract=/tmp/dbus-Q9XSs0L0b5,guid=732f9d58ea0dd495f08b56bc64af43f3 1534
ishii_t+    1539  0.0  0.3  73664 30876 ?        SLl  09:23   0:00 /usr/lib/mozc/mozc_server
ishii_t+   14479  0.0  0.0   9136   576 pts/0    T    11:38   0:00 wc -l
ishii_t+   14541  0.0  0.0  12532  3220 pts/0    R+   11:39   0:00 ps aux
ishii_t+   14542  0.0  0.0  10076   652 pts/0    S+   11:39   0:00 grep --color=auto ishii_t+
ishii_tdd@PCS27515:~/hoge02/Linux_shikumi/1-1$ ps aux --no-header | wc -l
61
ishii_tdd@PCS27515:~/hoge02/Linux_shikumi/1-1$ ls | wc -l
8
```

1. fork()関数を呼び出してプロセスを分岐させる。
1. 子プロセス用の領域を確保して、親プロセスのメモリをコピーする。
1. 親プロセス、子プロセスは両方ともfork()関数から解放される。
1. 親プロセスと子プロセスでは、fork()関数の戻り値が異なる。
1. fork()の戻り値に関しては、親プロセスでは子プロセスのPIDとなり、子プロセスでは0となります。

```
ishii_tdd@PCS27515:~/hoge02/Linux_shikumi/2-1$ cat fork.py
#!/usr/bin/python3
import os, sys
# fork()関数を呼び出してプロセスを分岐させる。
ret = os.fork()
# 子プロセス用の領域を確保して、親プロセスのメモリをコピーする。
# 親プロセス、子プロセスは両方ともfork()関数から解放される。
# 親プロセスと子プロセスでは、fork()関数の戻り値が異なる。

# 子プロセスの戻り値は、0
# 子プロセスが動いている。
if ret == 0:
        print("子プロセス, pid={}, 親プロセス, pid={}".format(os.getpid(), os.getppid()))
        exit()
# 親プロセスの戻り値は、子プロセスのpid
# 親プロセスが動いている
elif ret > 0:
        print("親プロセス, pid={}, 子プロセス, pid={}".format(os.getpid(), ret))
        exit()
sys.exit(1)
ishii_tdd@PCS27515:~/hoge02/Linux_shikumi/2-1$ ./fork.py
親プロセス, pid=14833, 子プロセス, pid=14834
子プロセス, pid=14834, 親プロセス, pid=14833
```

1. os.execv() 関数 を呼び出す。
1. 現在のプロセスのメモリを、新しいプロセスのデータで上書きする。
1. 新しいプロセスの最初に実行すべき命令(エントリーポイント)を実行開始する。


```
ishii_tdd@PCS27515:~/hoge02/Linux_shikumi/2-1$ cat fork-and-exec.py
#!/usr/bin/python3
import os, sys
# fork()関数を呼び出してプロセスを分岐させる。
ret = os.fork()
# 子プロセス用の領域を確保して、親プロセスのメモリをコピーする。
# 親プロセス、子プロセスは両方ともfork()関数から解放される。
# 親プロセスと子プロセスでは、fork()関数の戻り値が異なる。

# 子プロセスの戻り値は、0
# 子プロセスが動いている。
if ret == 0:
        print("子プロセス, pid={}, 親プロセス, pid={}".format(os.getpid(), os.getppid()))
        os.execv("/bin/ls", ["/bin/ls", "-la"])
        exit()
# 親プロセスの戻り値は、子プロセスのpid
# 親プロセスが動いている
elif ret > 0:
        print("親プロセス, pid={}, 子プロセス, pid={}".format(os.getpid(), ret))
        os.execv("/bin/echo", ["/bin/echo", "pid={} からよろしく".format(os.getpid())])
        exit()
sys.exit(1)
ishii_tdd@PCS27515:~/hoge02/Linux_shikumi/2-1$ ./fork-and-exec.py
親プロセス, pid=14988, 子プロセス, pid=14989
子プロセス, pid=14989, 親プロセス, pid=14988
pid=14988 からよろしく
ishii_tdd@PCS27515:~/hoge02/Linux_shikumi/2-1$ total 16
drwxr-xr-x 2 ishii_tdd ishii_tdd 4096 Jul 13 14:32 .
drwxr-xr-x 4 ishii_tdd ishii_tdd 4096 Jul 13 13:13 ..
-rwxr-xr-x 1 ishii_tdd ishii_tdd  913 Jul 13 14:32 fork-and-exec.py
-rwxr-xr-x 1 ishii_tdd ishii_tdd  785 Jul 13 14:02 fork.py
```


```
ishii_tdd@PCS27515:~/hoge02/Linux_shikumi/2-1$ cc -o pause -no-pie pause.c
ishii_tdd@PCS27515:~/hoge02/Linux_shikumi/2-1$ readelf -h pause
ELF Header:
  Magic:   7f 45 4c 46 02 01 01 00 00 00 00 00 00 00 00 00
  Class:                             ELF64
  Data:                              2's complement, little endian
  Version:                           1 (current)
  OS/ABI:                            UNIX - System V
  ABI Version:                       0
  Type:                              EXEC (Executable file)
  Machine:                           Advanced Micro Devices X86-64
  Version:                           0x1
  Entry point address:               0x401050
  Start of program headers:          64 (bytes into file)
  Start of section headers:          14648 (bytes into file)
  Flags:                             0x0
  Size of this header:               64 (bytes)
  Size of program headers:           56 (bytes)
  Number of program headers:         13
  Size of section headers:           64 (bytes)
  Number of section headers:         31
  Section header string table index: 30

ishii_tdd@PCS27515:~/hoge02/Linux_shikumi/2-1$ readelf -S pause
There are 31 section headers, starting at offset 0x3938:

Section Headers:
  [Nr] Name              Type             Address           Offset
       Size              EntSize          Flags  Link  Info  Align
  [ 0]                   NULL             0000000000000000  00000000
       0000000000000000  0000000000000000           0     0     0
  [ 1] .interp           PROGBITS         0000000000400318  00000318
       000000000000001c  0000000000000000   A       0     0     1
  [ 2] .note.gnu.propert NOTE             0000000000400338  00000338
       0000000000000020  0000000000000000   A       0     0     8
  [ 3] .note.gnu.build-i NOTE             0000000000400358  00000358
       0000000000000024  0000000000000000   A       0     0     4
  [ 4] .note.ABI-tag     NOTE             000000000040037c  0000037c
       0000000000000020  0000000000000000   A       0     0     4
  [ 5] .gnu.hash         GNU_HASH         00000000004003a0  000003a0

```

プログラムから作成したプロセスのメモリマップは、/proc/<pid>/mapsのファイルによって得られる。
用が済んだら、pauseプロセスをkillする。


```
ishii_tdd@PCS27515:~/hoge02/Linux_shikumi/2-1$ ./pause &
[2] 15059
ishii_tdd@PCS27515:~/hoge02/Linux_shikumi/2-1$ cat /proc/15059/maps
00400000-00401000 r--p 00000000 08:20 348522                             /home/ishii_tdd/hoge02/Linux_shikumi/2-1/pause
00401000-00402000 r-xp 00001000 08:20 348522                             /home/ishii_tdd/hoge02/Linux_shikumi/2-1/pause
00402000-00403000 r--p 00002000 08:20 348522                             /home/ishii_tdd/hoge02/Linux_shikumi/2-1/pause
00403000-00404000 r--p 00002000 08:20 348522                             /home/ishii_tdd/hoge02/Linux_shikumi/2-1/pause
00404000-00405000 rw-p 00003000 08:20 348522                             /home/ishii_tdd/hoge02/Linux_shikumi/2-1/pause
7fdc48b8b000-7fdc48bad000 r--p 00000000 08:20 12573                      /usr/lib/x86_64-linux-gnu/libc-2.31.so
7fdc48bad000-7fdc48d25000 r-xp 00022000 08:20 12573                      /usr/lib/x86_64-linux-gnu/libc-2.31.so
7fdc48d25000-7fdc48d73000 r--p 0019a000 08:20 12573                      /usr/lib/x86_64-linux-gnu/libc-2.31.so
7fdc48d73000-7fdc48d77000 r--p 001e7000 08:20 12573                      /usr/lib/x86_64-linux-gnu/libc-2.31.so
7fdc48d77000-7fdc48d79000 rw-p 001eb000 08:20 12573                      /usr/lib/x86_64-linux-gnu/libc-2.31.so
7fdc48d79000-7fdc48d7f000 rw-p 00000000 00:00 0
7fdc48d8f000-7fdc48d90000 r--p 00000000 08:20 12438                      /usr/lib/x86_64-linux-gnu/ld-2.31.so
7fdc48d90000-7fdc48db3000 r-xp 00001000 08:20 12438                      /usr/lib/x86_64-linux-gnu/ld-2.31.so
```

プロセスの親子関係
システムの初期化について
1. コンピュータの電源を入れる
2. BIOSやUFEIなどのファームウェアが起動して、ハードウェアを初期化する
3. ファームウェアがGRUBなどのブートローダを起動する
4. ブートローダがOSカーネル（Linuxカーネル）を起動する
5. Linuxカーネルがinit プロセスを起動する。
6. initプロセスが子プロセス.....さらに子プロセスに伝えていく

```
ishii_tdd@PCS27515:~/hoge02/Linux_shikumi/2-1$ pstree -p
systemd(1)─┬─ModemManager(340)─┬─{ModemManager}(360)
           │                   └─{ModemManager}(366)
           ├─NetworkManager(282)─┬─{NetworkManager}(358)
           │                     └─{NetworkManager}(361)
           ├─accounts-daemon(279)─┬─{accounts-daemon}(284)
           │                      └─{accounts-daemon}(337)
           ├─agetty(442)
           ├─atd(435)
           ├─avahi-daemon(280)───avahi-daemon(319)
           ├─containerd(10354)─┬─{containerd}(10357)
           │                   ├─{containerd}(10358)
           │                   ├─{containerd}(10359)
           │                   ├─{containerd}(10360)
```

プロセスの終了

親プロセスがwait()やwaitpid()といったシステムコールを呼び出すことで以下の情報が得られる。
- プロセスの戻り値。exit()関数の引数を256で割ったあまり。
- シグナルによって終了したか否か
- 終了までにどれだけのCPU時間を使ったか

```
ishii_tdd@PCS27515:~/hoge02/Linux_shikumi/2-1$ cat wait-ret.sh
#!/bin/bash
false &
# falseコマンドの終了を待ち合わせる。falseコマンドのPIDは$!変数から得られる。
wait $!
# wait後のfalseプロセスの戻り値は$?から得られる。
echo "falseコマンドが終了しました。: $?"
ishii_tdd@PCS27515:~/hoge02/Linux_shikumi/2-1$ ./wait-ret.sh
falseコマンドが終了しました。: 1
```

### ゾンビプロセスと孤児プロセス

親プロセスが子プロセスを生成してから、wait()関数などのシステムコールを呼ぶまで、子プロセスはシステム上に残り続ける
これを「ゾンビプロセス」という。

親プロセスがwait()関数などのシステムコールを呼ぶまえに、実行を終了した場合、その子プロセスは「孤児プロセス」となる。


シグナル

```
ishii_tdd@PCS27515:~/hoge02/Linux_shikumi/2-1$ ./intignore.py
^C^C^C^Z
[4]+  Stopped                 ./intignore.py
ishii_tdd@PCS27515:~/hoge02/Linux_shikumi/2-1$ ps -aux | grep intignore
ishii_t+   15345  5.6  0.1  19268  9416 pts/0    T    15:52   0:04 /usr/bin/python3 ./intignore.py
ishii_t+   15347  2.9  0.1  19268  9296 pts/0    T    15:53   0:01 /usr/bin/python3 ./intignore.py
ishii_t+   15348  7.2  0.1  19268  9296 pts/0    T    15:53   0:02 /usr/bin/python3 ./intignore.py
ishii_t+   15351  0.0  0.0  10076   648 pts/0    S+   15:53   0:00 grep --color=auto intignore
```

