title: sudo时让环境变量继续有效
categories: Backend
date: 2017-10-14 21:31:12
tags: [Linux, 问答系列]
---

今天在执行一个python脚本时，需要使用到一个环境变量，并且需要sudo权限才能执行。设置了环境变量后，在sudo执行脚本时还是找不到该环境变量。

因为sudo是一个新的环境，默认不会使用已有的环境变量。如果要sudo使用环境变量，有以下几种方式可以实现。

## 1. `sudo`时指定环境变量

```bash
➜  ~ sudo JUST_TEST='hello, test' bash -c 'echo $JUST_TEST'
hello, test
```

<!-- more -->

## 2. 使用`sudo`的`-E`选项

通过`man sudo`查看sudo的用法，有一个`-E`选项，说明如下：

```text
-E, --preserve-env
            Indicates to the security policy that the user wishes to preserve
            their existing environment variables.  The security policy may return
            an error if the user does not have permission to preserve the
            environment.
```

`-E`选项表示用户希望使用已有的环境变量，如：

```bash
➜  ~ export JUST_TEST2='hello, test2'
➜  ~ sudo -E bash -c 'echo $JUST_TEST2'
hello, test2
```

## 3. 通过`visudo`在sudoers中设置环境变量

在`/etc/sudoers`中设置的环境变量，`sudo`是可以使用的。但为了安全，最好不要直接编辑`/etc/sudoers`文件，而应该使用`visudo`命令修改，该命令在修改`/etc/sudoers`文件之前会进行语法检查。

```bash
➜  ~ export JUST_TEST3='hello, test3'
➜  ~sudo visudo
  Defaults        env_keep= "JUST_TEST3"
➜  ~ sudo bash -c 'echo $JUST_TEST3'
hello, test3
```

## 参考

+ [How to keep Environment Variables when Using SUDO](https://stackoverflow.com/questions/8633461/how-to-keep-environment-variables-when-using-sudo)

