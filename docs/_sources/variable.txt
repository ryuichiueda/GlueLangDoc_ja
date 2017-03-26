===================================
変数
===================================

　GlueLangでは、変数の型として変数と文字列が定義されています。

（メモ: 全般的に変数の管理の実装が混乱気味なので、リファクタリングが必要。）

ファイル変数
===================================

　GlueLangではファイルを ``/etc/passwd`` のようなパスの文字列の他、変数として扱うことができます。次の例は変数 ``f`` を定義しており、 ``f`` はファイルとして振舞います。

Fig.: file1.glue 

.. code-block:: bash
        :linenos:

	import PATH
	
	file f = head -n 1 '/etc/hosts'   #/etc/hostsから1行読み込んでfに入力
	
	cat f                             #fをcatで出力
	
この例のように、 ``f`` は「ファイル変数」と呼ばれます。ファイル変数は、コマンドに引数として渡すことができます。


文字列変数
===================================

　文字列変数は ``str 名前`` で定義します。次の例からは、上のfile1.glueと同じ出力が得られますが、 ``s`` の内容はGlueLang内のメモリに格納されます。

Fig.: str1.glue 

.. code-block:: bash
	:linenos:

	import PATH
	
	str s = head -n 1 '/etc/hosts'
	
	echo s

where句
===================================

　特定の処理だけに使いたい変数は、where句の中に定義することで作ることができます。
次の例は、ファイルa, bをwhere句の中で定義してdiffで比較している例です。

Fig.: where.glue 

.. code-block:: bash
	:linenos:

	import PATH
	
	# "file a = ..." and "file b" = ... are evaluated before "diff a b"
	diff a b !> echo 'different!!!'
	  where
            file a = seq 10
	    file b = seq 9

実行すると、後ろで定義したaとbがdiffで使えることが分かります。

.. code-block:: bash
	:linenos:

	$ glue ./where.glue 
	10d9
	< 10
	different!!!

　また、bashのコマンド置換の代わりに使うこともできます。
コマンドで作った長い文字列を別のコマンドの引数に渡すときに、すっきりした記述ができます。

Fig.: where_args.glue 

.. code-block:: bash
	:linenos:

	import PATH
	
	#passwdファイルの一番下のユーザの名前でpasswdファイルをgrepする
	grep local.re '/etc/passwd'
	  where
	    str re = tail -n 1 '/etc/passwd' >>= awk '-F:' '{print $1}'

Macで実行した結果を示します。

.. code-block:: bash

	$ glue ./where_args.glue 
	_wwwproxy:*:252:252:WWW Proxy:/var/empty:/usr/bin/false

where句まわりのスコープ
-----------------------------------

　where句の中で定義した変数等は、そのwhere句を持つジョブ内でのみ有効です。
次のコード例では、実行すると ``cat a`` でファイルaが存在しないとエラーが出ます。

* Fig.: where_scope.glue

.. code-block:: bash
	:linenos:

	$ cat where_scope.glue 
	import PATH
	
	diff a b !> echo 'different!!!'
	  where          
	    file a = seq 10
	    file b = seq 9
	
	cat a

.. code-block:: bash
	:linenos:

	$ glue ./where_scope.glue 
	10d9
	< 10
	different!!!
	
	Execution error at line 8, char 5
		line8: cat a
		           ^
	
		Variable a not found
		
		process_level 0
		exit_status 3
		pid 94404

　ジョブの外とwhere句の中に同じ名前の変数があるときは、
where句の中のものが優先されます。
ただし、別の名前をつけることを推奨します。
次の例は、ファイルaが二つ定義されています。

Fig.: where_scope2.glue 

.. code-block:: bash
	:linenos:
	
	import PATH
	
	file a = seq 2
	
	diff a b !> echo 'different!!!'
	  where          
	    file a = seq 10
	    file b = seq 9
	
	cat a

ジョブの中では ``file a = seq 10`` のファイルaが使われ、
外では ``file a = seq 2`` の方が使われます。
	
.. code-block:: bash
	:linenos:

	$ glue ./where_scope2.glue 
	10d9
	< 10
	different!!!
	1
	2
	
