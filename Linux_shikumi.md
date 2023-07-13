
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



