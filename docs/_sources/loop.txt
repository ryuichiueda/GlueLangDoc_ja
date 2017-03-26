===============================
ループ
===============================

whileブロック
===============================

　whileブロックは、コマンドが0以外の終了ステータスを返すまで、
無限にブロック内の制御を繰り返します。

シンプルな使い方
------------------------------------------

　この例は、dateにUnix時刻を吐かせて、5で割り切れる数になったらループを抜ける処理です。
testコマンドが0でない終了ステータスを返したのを受けてループが終わって最終行のechoが実行されます。


Fig.: simple_while.glue

.. code-block:: bash
        :linenos:

	import PATH
	 
        while	
	  str t = date '+%s' >>= awk '{print $1%5}'
	  echo t
	  test t -ne 0
	  sleep 1
	 
	echo 'end'

.. code-block:: bash
        :linenos:

	$ glue ./simple_while.glue 
	2
	3
	4
	0
	end

sh系のシェルのwhileのような使い方
------------------------------------------

　shやbashのように、 while <コマンド> ; do <処理> ; done と書く場合、つまりコマンドの終了ステータスで処理を実行するかしないかを決めるときは、1段インデントが深くなりますが、次のように記述できます。
1段目のインデントが条件、そのあとのdoブロックが実行したい処理になります。

Fig.: while_do.glue

.. code-block:: bash
        :linenos:

	import PATH
	 
        while	
	  str t = date '+%s' >>= awk '{print $1%5}'
	  test t -ne 0 ?> do
	    echo t
	    sleep 1
	 
	echo 'end'

実行すると、Unix時刻が5で割り切れる時にdoブロックの中身がスキップされて実行されません。

.. code-block:: bash
        :linenos:

	$ glue ./while_do.glue 
	3
	4
	end

　ネストが2段になりますが、shやbashのwhileよりも長く条件となるコマンドが書けます。
while hogehoge ; do …という書き方が、
hogehogeという部分をコマンドだと気づかない初心者を量産しており悪い影響を与えているので、
踏襲はしていません。

　また、 ``?>`` でエラーが起きたときは、スクリプトが即時に止まります。
終了ステータスは（この例では表示されていませんが）8です。


Fig.: while_do.glueのthenの後にfalseを挟んで実行した時の様子

.. code-block:: bash
	:linenos:

	$ glue ./while_do_error.glue 
	
	Execution error at line 3, char 1
		line3: while
		       ^
		line4:   str t = date '+%s' >>= awk '{print $1%5}'
		line5:   test t -ne 0 ?> do
		line6:     false
		line7:     echo t
		line8:     sleep 1
		line9:  
	
		Command error
		
		process_level 0
		exit_status 8
		pid 5016


foreachブロック
===============================

　bashのwhileは標準入力から字を受け付けますが、
GlueLangではforeachブロックがこの機能を持っています。
readと組み合わせると次のように使えます。

.. code-block:: bash
        :linenos:

	###こんなスクリプト###
	$ cat hoge.bash
	#!/bin/bash
	 
	seq 1 3 |
	while read a ; do
	    echo "@" $a
	done
	###こんな出力###
	$ ./hoge.bash
	@ 1
	@ 2
	@ 3

　GlueLangでは、foreachブロックを使うことで、
同様の処理が実装できます。
次の例のように、argv配列に読み込んだ文字列が格納されます。

Fig.: foreach_simple.glue 

.. code-block:: bash
        :linenos:

	import PATH 
	
	#一つずつ数字をforeachに入力
	seq 1 3 >>= foreach
	  echo '@' argv[1]
	
	#2列でforeachに入力
	seq 1 4 >>= xargs -n 2 >>= foreach
	  str f1 = echo argv[1]
	  str f2 = echo argv[2]
	  echo f1 f2

