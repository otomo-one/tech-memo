===========
BIND
===========

| Berkeley Internet Name Domainの略
| 提供する機能はネームサーバとそのキャッシュサーバ

DNSの仕組み
=============
http://www.fc-lab.com/network/system/dns/


bind インストール
===================
bindの本体
::

    yum install bind


キャッシュネームサーバ
=========================
| 1度問い合わせを行うと一定期間ローカルに問い合わせをキャッシュして上位ネームサーバの軽減を行うソフトウェア
| 参考:http://www.searchman.info/server/sev1070.html 
| 
| caching-nameserver インストール

::

    yum install caching-server

BINDのchroot
==============
| Linuxシステムファイルの中にBINDだけが使う小さなファイルシステムを仮想的に作ることで
| 攻撃を受けて進入された場合に被害をchrootの/var/namedの中だけにくいとめることができるようになる 
| 参考：http://linux.kororo.jp/cont/server/bind_chroot.php
| 
| 自分で設定することもできるし、パッケージをインストールすることも出来る
|
| bind-chroot インストール

::

    yum install bind-chroot

BINDの設定
============
| chroot以下の設定
|
| ./etc/にnamed.confがあるので設定する
| named.confのステートメント一覧
| http://www.nina.jp/server/redhat/bind/named.conf.html
| 
| directoryからゾーンファイルを読み込む
| chroot環境なので実際には/var/named/chroot/directoryとなる
| 設定例

-----------
named.conf
-----------

::

    options{
            version "null";
            directory "/var/named";
    };
    
    include "/etc/rndc.key";
    
    controls {
            inet 127.0.0.1 allow {localhost;} keys {rndc-key;};
    };
    
    zone "." IN {
            type hint;
            file "named.ca";
    };
    
    zone "localhost" IN {
            type master;
            file "local.zone";
    };
    
    zone "0.0.127.in-addr.arpa" IN {
            type master;
            file "local.rev";
    };
    
    zone "test.net" IN {
            type master;
            file "test.zone";
    };
    
    zone "65.168.192.in-addr.arpa" IN {
            type master;
            file "test.rev";
    };
    
    logging{
            channel default{
                    file "/var/log/named/named.log";
            };
            category default {default;};
    };


------------------
local.zone
------------------
::

    $TTL 86400
    @ IN SOA localhost. root.localhost. (
            2013010400 ; serial
            28800 ; refresh 8h
            14400 ; retry 4h
            604800 ; expire 1w
            86400 ; default_ttl 24h
    );
      IN NS         localhost.
      IN A          127.0.0.1


-----------
local.rev
-----------

::

    $TTL 86400
    @       IN SOA localhost. root.localhost. (
            2013010400 ; serial
            28800 ;
            14400 ;
            604800 ;
            86400 ;
    );
            IN NS   localhost.
    1       IN PTR  localhost.


| SELinuxが有効だとnamed.caとかが読み込めないとかあるので設定中は
| /etc/selinux/config
| からSELINUX=disabledにする
|
| named.caはルートサーバの情報で公開しているところに従って設定する
|
| rndcはbindの制御ツールで設定の再読み込みや停止が出来る
| ポートの設定でrndcのポートも空けておかないと動かないので注意
|
| ルータ----windows----vmware（dns）
| みたいな構成の場合
| 設定したドメインに関してのみルータからwindowsに送って
| windows側のvmwareのアダプタの設定でdnsをvmwareに設定してやると繋がる
