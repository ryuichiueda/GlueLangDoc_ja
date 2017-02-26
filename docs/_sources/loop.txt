===============================
ループ
===============================

　GlueLangの繰り返しの機能は、現在のところとても貧弱です。
繰り返し実行される処理の中で変数が使いまわせない等、まだまだ実用的な段階ではありません。
現在のところ、内部コマンドとして ``repeat`` と ``while`` が実装されています。

repeat
===============================

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
===============================

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

