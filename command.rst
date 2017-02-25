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
	
　この挙動は、GlueLangが暗黙理にコマンドのパスを設定しないことを示しています。
この仕様の意図は呼び出すコマンドを厳密に指定するためです。


.. code-block:: bash
	:linenos:
	
	import /usr/bin/ as ub

	ub.seq '1' '5' >>= ub.tac


	
.. [#internal_echo] echoはGlueLangの内部コマンドに存在するので、echoだけでも使えます。ただし、仕様上、in.echoと内部コマンドを示す接頭語をつけるべきでないかと検討中です。
