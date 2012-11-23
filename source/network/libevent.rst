==========
libevent
==========

シングルスレッド＋イベント駆動＋ノンブロッキング呼び出しなライブラリ。

インストール
--------------

環境はUbuntu 12.04 LTS

ライブラリは `本家 <http://libevent.org/>`_ から落としてくるか、

::

    $ wget https://github.com/downloads/libevent/libevent/libevent-2.0.20-stable.tar.gz

展開して設定→インストール

::

    $ tar xvzf libevent-2.0.20-stable.tar.gz
    $ cd libevent-2.0.20-stable
    $ ./configure
    $ make
    $ make install

エコーサーバ
--------------
`ninxit.blog libevent+使い方 <http://www.ninxit.com/blog/2008/02/13/libevent-%E4%BD%BF%E3%81%84%E6%96%B9/>`_ 
や
`in the mythosil Rhythm & Biology. libeventでechoサーバをつくってみた <http://mythosil.hatenablog.com/entry/20110806/1312644585>`_ 
を参考にエコーサーバを書いてみた。

::

    $ cat echoserver.c 
    #include <stdio.h>
    #include <stdlib.h>
    #include <string.h>
    #include <unistd.h>
    #include <netinet/in.h>
    #include <sys/socket.h>
    #include <sys/types.h>

    #include <event.h>

    void receive_handler(int client,short event,void *arg){
    	char buf[128];
    	struct event *ev = (struct event*)arg;
    	ssize_t len;

    	if(event & EV_READ){
    		if((len = recv(client,buf,127,0))<=0){
    			event_del(ev);
    			free(ev);
    			close(client);
    			printf("closed\n");
    		}else{
    			send(client,buf,len,0);
    		}	
    	}
    }

    void accept_handler(int server,short event, void *arg){
    	struct event *ev;
    	struct sockaddr_in addr;
    	socklen_t len = sizeof(addr);

    	if(event & EV_READ){
    		int sock = accept(server,(struct sockaddr*)&addr,&len);
    		ev = (struct event*)malloc(sizeof(struct event));
    		event_set(ev,sock,EV_READ|EV_PERSIST,receive_handler,ev);
    		event_add(ev,NULL);
    		printf("accepted\n");
    	}
    }

    int main(){
    	int sock;
    	int port = 1234;
    	struct sockaddr_in in;
    	struct event ev;
    	/* socket */
    	sock = socket(PF_INET, SOCK_STREAM, 0);
    	if(sock<0){
    		perror("socket");
    		return 1;
    	}
    	/* in setting */
    	memset(&in,0,sizeof(in));
    	in.sin_port = htons(port);
    	in.sin_family = AF_INET;
    	in.sin_addr.s_addr = htonl(INADDR_ANY);
    	/* binding */
    	if(bind(sock, (struct sockaddr *)&in,sizeof(in))<0){
    		perror("bind");
    		return 1;	
    	}
    	/* listen */
    	if(listen(sock,128)<0){
    		perror("listen");
    		return 1;
    	}
    	/* init libevent */
    	event_init();
    	/* libevent settings */
    	event_set(&ev,sock, EV_READ | EV_PERSIST,accept_handler,&ev);
    	event_add(&ev,NULL);
    	/* dispatch */
    	event_dispatch();

    	return 0;
    }

実行例
--------
クライアント側

::

    $ telnet localhost 1234
    Trying 127.0.0.1... Connected to localhost. Escape character is '^]'.
    abc
    abc
    aiueo
    aiueo
    ^]

    telnet> Connection closed.

サーバ側

::

    $ ./a.out 
    accepted
    closed
    ^C

参考文献
----------

 * `本家 <http://libevent.org/>`_ 
 * `ninxit.blog libevent+使い方 <http://www.ninxit.com/blog/2008/02/13/libevent-%E4%BD%BF%E3%81%84%E6%96%B9/>`_ 
 * `in the mythosil Rhythm & Biology. libeventでechoサーバをつくってみた <http://mythosil.hatenablog.com/entry/20110806/1312644585>`_ 
