---
layout: post
title: ubuntu (14.04)使用时的一些问题
categories: tool
tags: chrome flash ubuntu
---

#### chrome安装flash
*   安装：

        sudo apt-get install pepperflashplugin-nonfree
        sudo update-pepperflashplugin-nonfree --install

*   卸载：

        sudo update-pepperflashplugin-nonfree --uninstall
        sudo apt-get remove pepperflashplugin-nonfree

*   参考：

http://blog.csdn.net/lainegates/article/details/27830333

#### 忘记root密码

    sudo passwd -u root # 启用root账户。
    sudo passwd root #按提示示进行输入，root的密码最好和其他用户的密码不同。

#### idea识别不了java文件

怎么设置都不显示，最后把其家目录下的配置删除,重新配置ok

linux许多与用户相关的配置好多都在家目录之下，以 隐藏文件夹的形式存在。

#### 开机时出现waiting for network configuration

问题原因：
使用 sudo pppoeconf 命令时，会有信息写入/etc/network/interfaces 文件内，直接导致出现了上面的问题。

问题解决：
sudo gedit /etc/network/interfaces 打开文件后，将其中除
auto lo
iface lo inet loopback
外其他内容全部删除后，重启系统就可以了。
