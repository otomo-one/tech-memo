==============================
iptablesの設定
==============================

ポートの設定などを行うソフトウェア。

設定確認
----------------------
::

    # iptables --list

設定例
---------------

例えば、

 * サーバから出て行くパケットは全部通す
 * サーバから出て戻ってきた（関連のある）パケットは全部通す
 * sshで接続するためにtcpかつポート22番は通す
 * Webページの設置のためにtcpかつhttp(ポート80番)は通す
 * ping等のためにicmpプロトコルは通す
 * それ以外の入ってくるパケットは破棄する

を設定してみる。 
環境はCentOS 6.3。 
shファイル。 

::

    # cat iptables.sh
    # 初期化
    iptables --flush

    # 受信
    # 過去に関連あり
    iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
    # ssh
    iptables -A INPUT -p tcp --dport 22 -j ACCEPT
    # http
    iptables -A INPUT -p tcp --dport 80 -j ACCEPT
    # icmp
    iptables -A INPUT -p icmp -j ACCEPT
    # のこりは拒否
    iptables -A INPUT -j DROP

設定を確認すると

::

    # iptables --list
    Chain INPUT (policy ACCEPT)
    target     prot opt source               destination
    ACCEPT     all  --  anywhere             anywhere            state RELATED,ESTABLISHED
    ACCEPT     tcp  --  anywhere             anywhere            tcp dpt:ssh
    ACCEPT     tcp  --  anywhere             anywhere            tcp dpt:http
    ACCEPT     icmp --  anywhere             anywhere
    DROP       all  --  anywhere             anywhere

    Chain FORWARD (policy ACCEPT)
    target     prot opt source               destination

    Chain OUTPUT (policy ACCEPT)
    target     prot opt source               destination

ポリシーがACCEPTなのでOUTPUTは特にコマンドは必要ない。

設定ファイルの保存
-----------------------------

リブートすると設定が消えてしまうので保存するには

::

    # service iptables save

| とする。
| 設定ファイルは/etc/rc.d/init.d/にあるiptablesファイル。
| ちなみにiptablesファイルを直接編集することもできる。


参考文献
--------------------------
 * `iptablesの設定　入門編 <http://d.hatena.ne.jp/yamasahi/20100206/1265444193>`_
