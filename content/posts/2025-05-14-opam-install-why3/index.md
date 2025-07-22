---
title: 用Opam安装Frama-C
description: 解决why3安装失败的问题
date: 2025-05-14
tags: [Opam, Debug]
---

## 系统信息

* linux
* gcc 15.1.1
* opam 2.3.0

## 问题表现

```bash
#=== ERROR while compiling why3.1.7.2 =========================================#
# context     2.3.0 | linux/x86_64 |  | git+https://mirrors.sjtug.sjtu.edu.cn/git/opam-repository.git
# path        ~/.opam/frama-c/.opam-switch/build/why3.1.7.2
# command     ~/.opam/opam-init/hooks/sandbox.sh build make -j27 all byte
# exit-code   2
# env-file    ~/.opam/log/why3-234433-04074f.env
# output-file ~/.opam/log/why3-234433-04074f.out
### output ###
# src/server/cpulimit-unix.c:95:23: error: assignment to ‘__sighandler_t’ {aka ‘void (*)(int)’} from incompatible pointer type ‘void (*)(void)’ [-Wincompatible-pointer-types]
# [...]
# src/server/cpulimit-unix.c:45:6: note: ‘wallclock_timelimit_reached’ declared here
#    45 | void wallclock_timelimit_reached() {
#       |      ^~~~~~~~~~~~~~~~~~~~~~~~~~~
# In file included from src/server/cpulimit-unix.c:22:
# /usr/include/signal.h:72:16: note: ‘__sighandler_t’ declared here
#    72 | typedef void (*__sighandler_t) (int);
#       |                ^~~~~~~~~~~~~~
# Ocamlopt src/util/weakhtbl.ml
# make: *** [Makefile:752: src/server/cpulimit-unix.o] Error 1
# make: *** Waiting for unfinished jobs....
# Ocamlc   src/util/diffmap.mli



<><> Error report <><><><><><><><><><><><><><><><><><><><><><><><><><><><><><><>
┌─ The following actions failed
│ λ build why3 1.7.2
└─
```

## 报错原因

gcc版本太高

## 解决方案

1. 首先进入到build目录

    ```bash
    cd ~/.opam/frama-c/.opam-switch/build/why3.1.7.2
    ```

2. 然后手动编译why3

    要么改Makefile，加上 `-Wno-incompatible-pointer-types`

    要么

    ```bash
    sed -i 's/void wallclock_timelimit_reached()/void wallclock_timelimit_reached(int signum)/' src/server/cpulimit-unix.c
    ```

3. 接着使用该目录下编译好的why3进行编译并安装

    ```bash
    opam pin add why3.1.7.2 .
    ```

    参考：https://opam.ocaml.org/doc/Usage.html#opam-pin

4. 最后正常安装Frama-C就好了

    ```bash
    opam install frama-c
    ```
