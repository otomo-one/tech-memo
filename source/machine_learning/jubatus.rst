=============
Jubatus
=============

機械学習×大規模分散処理なOSS。

サーバ側
============
| 環境：CentOS 6.3

インストール
-------------------
| yumがあるのでそれに従う

::

    # rpm -Uvh http://download.jubat.us/yum/rhel/6/stable/x86_64/jubatus-release-6-1.el6.x86_64.rpm
    # yum install jubatus

起動
-----------------
| チュートリアル用のクライアントを動かすので--nameでtutorialを指定する

::

    # jubaclassifier --name=tutorial

クラスタ構成
---------------------
分散処理をするためにはZooKeeperというのを使うらしい。


クライアント側
=========================
| 環境:Windows7(msys入り)、Python 2.7

前準備
---------------
| Windows+Pythonで動かすには
| VisualStudio2008とpythonのパッケージ管理pip or easy_installが必要なのであらかじめインストールしておく。

ライブラリ等のインストール
-----------------------------------------
| msgpack-rpc-pythonをインストールする。pipにも入っているがエラーを吐いたので
| githubのページから落としてきて

::

    # python setup.py install

| とした。

| 次にjubatusクライアントのインストール。
| こちらはpipで入れられた。

::

    # pip install jubatus

チュートリアルクライアント
----------------------------------------

| チュートリアル用のクライアントプログラムが公開されているので
| 利用する。

::

    # git clone git://github.com/jubatus/jubatus-tutorial-python.git
    # cd jubatus-tutorial-python
    # wget http://people.csail.mit.edu/jrennie/20Newsgroups/20news-bydate.tar.gz
    # tar -xvzf 20news-bydate.tar.gz
    # python tutorial.py

| tutorial.pyの中にIPアドレスを設定する項目があるので適宜変更する。
| 学習とテストが行われて結果が表示される。

参考文献
-----------------

 * `Jubatusを公開しました <http://research.preferred.jp/2011/10/jubatus/>`_ 
 * `msgpack-rpc-python <https://github.com/msgpack/msgpack-rpc-python>`_