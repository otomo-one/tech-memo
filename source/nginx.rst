=======
nginx
=======

概要
------

HTTP、リバースプロキシ、メールプロキシサーバソフトウェア。
非同期でイベント駆動なリクエスト処理といったらnginxと思っていたら
Apache 2.4.1(2012年2月)でも似たようなアーキテクチャが選択できるようになっていたらしい。

インストール
--------------

Ubuntu 12.04 LTSの環境にインストールしてみる。

::

    sudo apt-get install nginx

バージョン

::

    $ nginx -v
    nginx version: nginx/1.1.19

設定ファイル
--------------

/etc/nginx/に設定ファイルnginx.confが置かれている。
詳しくは `nginx連載3回目: nginxの設定、その1 <http://heartbeats.jp/hbblog/2012/02/nginx03.html>`_ 参照。
デフォルト設定ファイルを参考に設定ファイルを作ってみた。

設定例

::

    $ cat /etc/nginx/nginx.conf
    user www-data;
    worker_processes 4;
    pid /var/run/nginx.pid;

    events {
    	worker_connections 768;
    }

    http {
    	sendfile on;
    	tcp_nopush on;
    	tcp_nodelay on;
    	keepalive_timeout 65;
    	types_hash_max_size 2048;

    	include /etc/nginx/mime.types;
    	default_type application/octet-stream;

    	access_log /var/log/nginx/access.log;
    	error_log /var/log/nginx/error.log;

    	gzip on;
    	gzip_disable "msie6";

    	include /etc/nginx/conf.d/*.conf;
    }

バーチャルサーバの設定
----------------------------

前述の

::

    /etc/nginx/conf.d/*.conf;

はバーチャルサーバの設定を読み込んでいる。
詳しくは `nginx連載4回目: nginxの設定、その2 - バーチャルサーバの設定  <http://heartbeats.jp/hbblog/2012/04/nginx04.html>`_ 参照。
default.confという名前で設定ファイルを作ってみた。

設定例

::

    $ cat /etc/nginx/conf.d/default.conf 
    server{
    	listen		80;
    	server_name	localhost;

    	location / {
    		root /home/otomo/www/test/html;
    		index index.html;
    	}
    }

プロセスの起動
----------------

上記設定を行うととりあえずは動くので起動してみる。

起動

::

    $ sudo /etc/init.d/nginx start

再起動

::

    $ sudo /etc/init.d/nginx restart

設定ファイル読み込み

::

    $ sudo /etc/init.d/nginx reload

停止

::

    $ sudo /etc/init.d/nginx stop

ブラウザを起動してlocalhostへアクセスすれば/home/otomo/www/test/html/index.htmlが表示される。

参考文献
----------

 * http://nginx.org/ja/
 * `nginx連載3回目: nginxの設定、その1 <http://heartbeats.jp/hbblog/2012/02/nginx03.html>`_ 
 * `nginx連載4回目: nginxの設定、その2 - バーチャルサーバの設定  <http://heartbeats.jp/hbblog/2012/04/nginx04.html>`_
