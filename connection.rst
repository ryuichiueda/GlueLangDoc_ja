==========================================
コマンドの接続
==========================================

パイプライン
==========================================

　パイプの記号は ``>>=`` です。
``|`` と比較すると打つのが面倒ですが、
一方でスクリプトの中で目立つので、
よく見落とされる ``|`` よりは分かりやすいだろうという意図です。
また、厳密な裏付けはありませんが、Haskellの ``>>=`` と
パイプの挙動は似ているので、この記号を使用しています。

　パイプは次のように使います。

.. code-block:: bash
        
        /usr/bin/seq '1' '5' >>= /usr/bin/tac
        #Macの場合: /usr/bin/seq '1' '5' >>= /usr/bin/tail '-r'

出力は、bashで ``seq 1 5 | tac`` したときと同じです。
上のスクリプトを ``pipeline.glue`` というファイルに保存して実行すると次のような出力が得られます。

.. code-block:: bash

	$ glue pipeline.glue 
	5
	4
	3
	2
	1


　また、次のように改行することもできます。

.. code-block:: bash

	/usr/bin/seq '1' '5'
	>>= /usr/bin/tail -r
	
        #これでもOK
	/usr/bin/seq '1' '5' >>=
	/usr/bin/tail -r


AND記号
==========================================

　Bシェル系の ``&&`` に相当する記号は ``>>`` です。
これもHaskellに由来します。次のように使います。

.. code-block:: bash

        /bin/echo 'a' >> /bin/echo 'b' >> /bin/echo 'c'

このコードを ``and.glue`` というファイルに保存して実行した時の出力を示します。

.. code-block:: bash

        a
        b
        c

OR記号
==========================================

　一方OR記号については ``!>`` という表記を採用しました。

* Fig.: or.glue

.. code-block:: bash
        :linenos:

        import PATH

        false !> echo 'Echo is executed.'
        true !> echo 'Echo is not executed.'

実行するとこうなります。

.. code-block:: bash

        $ glue or.glue 
        Echo is executed.

　ANDやORは、最後に実行されたコマンドで条件分岐します。
次の例の場合、 ``false`` が失敗するので、 ``>>``
の右側の ``echo 'if'`` は実行されず、その次の ``echo 'else'``
が実行されます。

* Fig.: or2.glue

.. code-block:: bash

        import PATH
        false >> echo 'if' !> echo 'else'

動作は次のようになります。

.. code-block:: bash

        $ glue ./or2.glue 
        else

　一方、次の例ではifの方が出力されて、
この時の ``echo`` が成功するのでelseは出力されません。

* Fig.: or3.glue

.. code-block:: bash

        import PATH
        true >> echo 'if' !> echo 'else'


バグ
------------------------------------------

　次のように二つ ``!>`` をつなげた時の挙動が環境によって異なるバグが発生しており、取りきれていません。

Fig.: or_bug.glue 

.. code-block:: bash

        import PATH
        false !> echo 'b' !> echo 'c'

.. code-block:: bash

        #Travis 上
        $ glue ./or_bug.glue 
        b
        c
        #Ubuntu 16.04 Server, macOS Sierra
        $ glue ./or_bug.glue 
        b

Then記号（未実装）
==========================================

　 ``?>`` はthenを表します。
then記号の左側にあるコマンドが成功すると、右側のコマンドが実行されて、
さらにそれ以後にANDやORでコマンドがつながっていても実行されません。
失敗するとスクリプトが止まります。
つまり、次のようにコマンドを並べると、if文を作ることができます。

.. code-block:: bash

        ###if A then B else if C then D else E###
        A ?> B
        !> C ?> D
        !> E

メモ
------------------------------------------

　Haskellだとこうなるのでもっとスッキリのですが、A、Cに相当する部分が長くなることがあり、シェルには向いていないかもしれません。

.. code-block:: bash

        ###if A then B else if C then D else E###
        A        ?> B
        C        ?> D
        othewise ?> E


複合コマンド（doブロック）
==========================================

　コマンドを束ねたいときはdoという命令の後にインデントして複数のコマンドを記述します。
doと、doに結びつけられたコマンド全体を「doブロック」と呼びます。
doブロックは、シェルにおける複合コマンドとほぼ等価です。

単純なブロック化
-----------------------------------------

　次の例は二つのコマンドを1つに束ねる例です。


* 図: do_block.glue 

.. code-block:: bash

	import PATH
	
	do
	  echo 'a'  
	  echo 'b' 

この例の場合、実行した結果は束ねないときと特に同じです。

.. code-block:: bash

	$ glue do_block.glue 
	a
	b

他の機能との併用
-----------------------------------------

　doの前には文字列やファイルへの格納や、手続き（6章）の宣言を置くことができます。

* 図: do_block_plus.glue

.. code-block:: bash

	import PATH
	
	#手続きの定義・宣言
	proc fn = do
	  echo 'c' 
	
	#ファイルへの格納
	file f = do
	  echo 'a'  
	
	#文字列への格納
	str s = do
	  echo 'b'
	
	cat f >> echo s >> fn

実行結果は次のようになります。


.. code-block:: bash

	$ glue ./do_block_plus.glue 
	a
	b
	c

　また、ANDやORとの併用も可能です。
次の例はORでdoブロックを接続する例です。

図: do_if_then.glue 

.. code-block:: bash

	import PATH
	
	do
	  false       #これで次の!>に飛ぶ
	  echo 'a'    #実行されない
	!> do
	  echo 'b'    #これが実行される
	!> do
	  echo 'c'    #これは実行されない

実行すると、次のようにecho 'b'だけ実行されます。

.. code-block:: bash

	$ glue ./do_if_then.glue 
	b

	
スコープ
-----------------------------------------

　doブロックの中で定義した変数はdoブロックの中でのみ有効です。
次の例はdoブロックの中で定義したファイル変数fを、
doブロックの外で使った例です。

* 図: do_block_scope.glue 

.. code-block:: bash

	import PATH
	
	do
	  file f = echo 'a'  
	
	cat f    #エラーが起きる

実行すると次のようにエラーが出ます。

.. code-block:: bash

	$ glue ./do_block_scope.glue 
	
	Execution error at line 6, char 5
		line6: cat f    #エラーが起きる
		           ^
	
		variable f not found
		
		process_level 0
		exit_status 1
		pid 3128


  回避するときの一例としては、次のようにシステムのファイルを使う方法があります。
実行結果は省略します。

.. code-block:: bash

	import PATH
	
	do
	  file f = echo 'a'  
	  mv f '/tmp/hoge'
	
	cat '/tmp/hoge'
	
　逆に外側で作られた文字列やファイルはdoブロックの中で使うことができます。
実行例は省略しますが、次の例はエラーが起きず、「アイウエオ」と出力されます。

.. code-block:: bash
	
	import PATH
	
	str s = 'アイウエオ'
	
	do
	  echo s
