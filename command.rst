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
~~~~~~~~~~~~~~~~~~~~~~~~~~

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
``Unknown token`` とあるように、 ``cat`` が認識されていません。

.. code-block:: bash
	:linenos:

	$ glue command2
	glue: script read error
	uedamb:examples ueda$ glue command2_ng.glue 
	
	Parse error at line 1, char 1
		line1: cat '/etc/resolv.conf'
		       ^
	
		Unknown token
		
		process_level 0
		exit_status 1
		pid 82246
	
	
