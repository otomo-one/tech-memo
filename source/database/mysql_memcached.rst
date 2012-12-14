=================
MySQLとmemcached
=================

MySQLとmemcachedをpythonで使う
--------------------------------------

| ・環境
| サーバ：centos6.3+MySQL+memcached
| クライアント：windows7+python2.7

mysqlのインストール
--------------------------
::

    yum install mysql-server

文字コード設定
--------------------
::

    vim /etc/my.cnf

::

    character-set-server = utf8

に変更

初期設定①
-----------------
::

    mysql_install_db

起動
--------------
::

    /etc/rc.d/init.d/mysqld start

初期設定②
-----------------------
| rootパスワードの設定
| リモートでrootログインの禁止
| testデータベースの削除
| など

::
    
    mysql_secure_installation

ユーザ作成
-------------------
初期設定で設定したパスワードでrootにログイン

::
    
    mysql -u root -p

| grant構文でユーザを作成する
| 参考文献：http://dev.mysql.com/doc/refman/5.1/ja/grant.html

::

    grant all on test.* to 'test'@'%' identified by '****';
    flush privileges;

ユーザの確認
::

    select user from mysql.user;
    exit

テーブルの作成
------------------------
| ユーザ作成で設定したパスワードでログイン

::

    mysql -u test -p

::

    create database test;
    show databases;

テーブル作成

::

    use test;
    create table test(name varchar(50),value int);


memcached
==============================

インストール
---------------
::

    yum install memcached

起動
--------------
::

    memcached &

セキュリティ設定はiptablesでやったほうがよさそう


PythonでMySQL＋memcached
==============================
| mysql+memcachedで読み込み
| 参考文献：http://dev.mysql.com/doc/mysql-ha-scalability/en/ha-memcached-interfaces-python.html

MySQLdb
--------------

pipだとVisual Studio 9.0のcl.exeが必要みたいでエラー出る。
easy_installで

::

    easy_install mysql-python

python-memcached
----------------------

::

    pip install python-memcached

書き込みスクリプト
-----------------------
::

    #coding:utf-8
    import MySQLdb,random

    # ユーザ名とパスワードを入力
    us = raw_input('user?: ')
    pw = raw_input('password?: ')

    # DBに接続
    db = MySQLdb.connect(host='192.168.65.10',user=us, passwd=pw, db='test', charset='utf8')
    cur = db.cursor()

    # INSERT
    for i in xrange(3000):
    	name = 'user%s' % str(i)
    	value = int(random.random()*1000)
    	cur.execute("INSERT INTO test(name, value) VALUES ('%s','%s')" % (name,str(value)) )
    	
    db.commit()

読み込みスクリプト
---------------------------
::

    #coding:utf-8
    import MySQLdb,memcache,time,random

    # ユーザ名とパスワードを入力
    us = raw_input('user?: ')
    pw = raw_input('password?: ')

    # DBに接続
    db = MySQLdb.connect(host='192.168.65.10',user=us, passwd=pw, db='test', charset='utf8')
    cur = db.cursor()


    # 計測
    memca_time = 0.0
    mysql_time = 0.0
    # SELECT
    memc = memcache.Client(['192.168.65.10:11211'], debug=1)
    for i in xrange(1000):
    	maxvalue = str(int(random.random()*1000))
    	# memcachedに問い合わせ
    	start = time.clock()
    	result = memc.get(maxvalue)
    	end = time.clock()
    	memca_time += end-start
    	# mysqlに問い合わせ
    	start = time.clock()
    	cur.execute('SELECT name,value FROM test WHERE value<=%s ORDER BY value DESC LIMIT 5' % maxvalue)
    	end = time.clock()
    	mysql_time += end-start
    	# 結果がなければ格納する
    	if not result:
    		start = time.clock()
    		rows = cur.fetchall()
    		memc.set(maxvalue,rows,60)
    		end = time.clock()
    		memca_time += end-start
    	else:
    		rows = result
    	print 'maxvalue is %s' % maxvalue
    	for row in rows:
    		print row[0],row[1]

    print 'memcached time:',memca_time
    print 'mysql time:',mysql_time


結果
---------------------
| memcachedはハッシュみたいなので重めのO(1)で
| MySQLはインデックスが効いていればB木系列なのでO(logn)でしょうか
| (nはデータの個数)

+------------------+------------------+
|memcached time[s] |   0.736795192833 |
+------------------+------------------+
|mysql time[s]     |   1.85960996652  |
+------------------+------------------+
