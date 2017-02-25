====================================
コマンドの呼び出し
====================================

単純な呼び出し
====================================

　一番単純な呼び出しは次のようなものです。
ファイル名は ``command.glue`` です。

.. code-block:: bash

        /bin/echo 'abc'

　これを次のように呼び出すと、echoが実行できます。

.. code-block:: bash

        $ glue command.glue 
        abc

パスの扱い
====================================

　デフォルトの状態でのGlueLangは、
シェルよりも書き方が制限されます。
まず、次の例はフルパスでcat(1)を呼び出す例です。

* Fig.: command2.glue

.. code-block:: bash
        /bin/cat '/etc/resolv.conf'

これを実行すると、 ``/etc/resolv.conf`` がcatされます。
実行結果は割愛します。


　一方で、通常のBシェルのように次のように書いたスクリプトはどうでしょうか。

* Fig.: command2.glue

.. code-block:: bash

	cat '/etc/resolv.conf'

実行すると次のようにエラーが出ます。
``Unknown token`` とあるように、 ``cat`` が認識されていません
[#internal_echo]_ 。

.. code-block:: bash
	:linenos:

	$ glue command2
	glue: script read error
	$ glue command2_ng.glue 
	
	Parse error at line 1, char 1
		line1: cat '/etc/resolv.conf'
		       ^
	
		Unknown token
		
		process_level 0
		exit_status 1
		pid 82246
	
　この挙動は、GlueLangが暗黙理にコマンドのパスを設定しないことを示しています。この仕様の意図は呼び出すコマンドを厳密に指定するためです。

　フルパスでコマンドを指定したくないときは、次のようにimport文を使います。ディレクトリ ``/usr/bin/`` を ``ub`` という名前に結びつけて、 ``/usr/bin/`` にあるseqコマンドにubという接頭語をつけて呼び出しています。


.. code-block:: bash
	:linenos:
	
	import /usr/bin/ as ub
	ub.seq '1' '5'

　接頭語は、同じディレクトリにあるコマンドを同じグループに結びつける役割をします。この仕様は、例えばLinuxとMacで異なるディレクトリにコマンドがある際のポータビリティを損ねます。しかし、例えばGlueLang用のポータビリティのあるコマンドを同じディレクトリに置いて使うという使い方を想定すると、逆にポータビリティを確保できます。

　もっと「雑に」コマンドを使いたいときには、別のディレクトリを同じ接頭語に結びつけることもできます。

.. code-block:: bash
	:linenos:
	
	import /bin/ as b
	import /usr/bin/ as b
	
	b.echo 'hoge'
	b.seq 1 10
	
この例では、二つのディレクトリのコマンドを同じ接頭語 ``b`` をつけて使っています。

　もっと横着をする方法もあります。次の例は環境変数PATHを読み込んで
接頭語なしでコマンドを使う方法です。
使い捨てのスクリプトを書くときにはこのように楽に書いた方が良いでしょう。

.. code-block:: bash
	:linenos:

        import PATH
        awk 'BEGIN{a=1;print a,a}'

このコードは次のように動作します。

.. code-block:: bash

        $ glue path_env.glue 
        1 1

	
.. [#internal_echo] echoはGlueLangの内部コマンドに存在するので、echoだけでも使えます。ただし、仕様上、in.echoと内部コマンドを示す接頭語をつけるべきでないかと検討中です。
