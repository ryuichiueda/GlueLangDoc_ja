===============================
ループ
===============================

loopブロック
===============================

　loopブロックは、コマンドが0以外の終了ステータスを返すまで、
無限にブロック内の制御を繰り返します。

シンプルな使い方
------------------------------------------

　この例は、dateにUnix時刻を吐かせて、5で割り切れる数になったらループを抜ける処理です。
testコマンドが0でない終了ステータスを返したのを受けてループが終わって最終行のechoが実行されます。


Fig.: simple_loop.glue

.. code-block:: bash
        :linenos:

	import PATH
	 
	loop
	  str t = date '+%s' >>= awk '{print $1%5}'
	  echo t
	  test t -ne 0
	  sleep 1
	 
	echo 'end'

.. code-block:: bash
        :linenos:

	$ glue ./simple_loop.glue 
	2
	3
	4
	0
	end

sh系のシェルのwhileのような使い方
------------------------------------------

　shやbashのように、 while <コマンド> ; do <処理> ; done と書く場合、つまりコマンドの終了ステータスで処理を実行するかしないかを決めるときは、1段インデントが深くなりますが、次のように記述できます。
1段目のインデントが条件、そのあとのdoブロックが実行したい処理になります。

Fig.: loop_as_while.glue

.. code-block:: bash
        :linenos:

	import PATH
	 
	loop
	  str t = date '+%s' >>= awk '{print $1%5}'
	  test t -ne 0 >> do
	    echo t
	    sleep 1
	 
	echo 'end'

実行すると、Unix時刻が5で割り切れる時にdoブロックの中身がスキップされて実行されません。

.. code-block:: bash
        :linenos:

	$ glue ./loop_as_while.glue 
	3
	4
	end

　ネストが2段になりますが、shやbashのwhileよりも長く条件となるコマンドが書けます。だいたい、while hogehoge ; do …という書き方が、hogehogeという部分をコマンドだと気づかない初心者を量産しており悪い影響を与えているので、踏襲はしていません。

内部コマンドによる繰り返し（廃止予定）
==============================================================

　内部コマンドとして ``repeat`` と ``while`` が実装されています。
内部コマンドなので、どちらも ``in.`` を頭につけて利用します。

repeat
-------------------------------

　repeatは回数を指定して手続きを実行します。今のところ手続きだけを処理対象の引数としてとります。
次の例は手続きfを4回繰り返す処理です。


Fig.: internal_repeat.glue 

.. code-block:: bash
        :linenos:

	proc f = do
	  echo 'aaa'
	  echo 'bbb'
	
	in.repeat 3 this.f

実行すると次のようになります。

.. code-block:: bash

	$ glue internal_repeat.glue 
	aaa
	bbb
	aaa
	bbb
	aaa
	bbb
	
　もちろん結果をファイルに書き出すこともできます。

.. code-block:: bash
	:linenos:

	$ cat internal_repeat_file.glue 
	import PATH
	
	proc f = do
	  echo 'aaa'
	  echo 'bbb'
	
	file x = in.repeat 3 this.f    #xというファイルに書き出す
	head -n 3 x                    #頭3行だけ出力

	###実行###
	$ glue internal_repeat_file.glue 
	aaa
	bbb
	aaa

while
-------------------------------

　whileは手続きが失敗するまでその手続きを実行します。
今のところ手続きだけを処理対象の引数としてとります。
次の例は、 ``date(1)`` でUNIX時刻を出力して、
3で割った余りが0になれば ``test(1)`` が1を返して失敗するという
手続きをwhileで実行したものです。
whileは終了ステータスが非ゼロになるのを前提で使うものなので、
手続きが0以外を返したときも処理が続行されます。
repeatの場合は即止まります。

.. code-block:: bash
	:linenos:

	$ cat internal_while.glue 
	import PATH
	
	proc f = do
	  sleep 1
	  str tmp = date '+%s' >>= awk '{print $1%3}' 
	  echo tmp
	  test tmp -ne 0
	  
	in.while this.f
	echo 'OK'          #これは実行される

	###実行（0が出たら止まる）###
	$ glue internal_while.glue 
	2
	0
	OK


制御演算子の案
===============================

　演算子として繰り返しを実装する方法について書いておきます。
``xargs(1)`` 等を使いこなせばいらないような気もしますが、それは言っちゃいけないような気がします。

while文
-------------------------------

　 ``<?>`` はHaskellの記号と紛らわしいので ``<>`` でもよいかもしれない。

.. code-block:: bash

        ###Aが終了ステータス0の間、Bを実行###
        A <?> B
        ###Aが終了ステータス0の間実行###
        ###いや、これは左側（上）にコマンドがあると紛らわしいかも###
        <?> A

for文
-------------------------------

　やりすぎ？

.. code-block:: bash

        ###Aをn回繰り返す###
        <n> A
        ###文字列a,b,cをAの引数にしつつ実行###
        <['a' 'b' 'c']> A
