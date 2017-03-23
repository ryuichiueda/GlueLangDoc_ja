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
where句の中の変数はlocalという接頭語をつけることで利用できます。
実行結果は省略します。

Fig.: where.glue 

.. code-block:: bash
	:linenos:

	import PATH
	
	# "file a = ..." and "file b" = ... are evaluated before "diff a b"
	diff local.a local.b
	  where
            file a = seq 10
	    file b = seq 10

　where句の機能は、bashのコマンド置換の代わりに使うことを意図して実装したものです。
コマンドで作った長い文字列を別のコマンドの引数に渡すときに、すっきりした記述ができます。
ただ、まだ文字列変数をwhere句で使えないので、次の例は動きません。

Fig.: where_args.glue 

.. code-block:: bash
	:linenos:

	import PATH
	
	#passwdファイルの一番下のユーザの名前でpasswdファイルをgrepする
	grep local.re '/etc/passwd'
	  where
	    str re = tail -n 1 '/etc/passwd' >>= awk '-F:' '{print $1}'
