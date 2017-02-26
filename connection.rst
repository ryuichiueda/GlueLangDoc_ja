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

　 ``?>`` はthenを表します。then記号の右側にあるコマンドが成功すると、それより右側にANDやORでコマンドがつながっていても実行されません。失敗するとスクリプトが止まります。

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
