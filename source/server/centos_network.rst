===========================
CentOSでいろいろ設定
===========================

いろいろ触ってみたときのメモです。

ネットワーク設定ファイル
------------------------------------------------

::

    # vim /etc/sysconfig/network-scripts/ifcfg-eth0

    DEVICE="eth0"#デバイス名（ファイル名の末尾と一致するように）
    BOOTPROTO="static"#固定static,none、自動dhcp
    IPV6INIT="no"
    ONBOOT="yes"
    NETMASK=255.255.255.0
    GATEWAY=192.168.65.2#ルータのアドレス(VMWare使っているので)
    IPADDR=192.168.65.10#IPアドレス
    TYPE="Ethernet"
	
再起動

::

	# /etc/rc.d/init.d/network restart

確認

::

    # ifconfig

ssh
-------------
これは共通

::

    # yum -y install openssh-server

起動

::

    # /etc/rc.d/init.d/sshd restart


