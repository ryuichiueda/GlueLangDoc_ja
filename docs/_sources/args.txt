================================
引数
================================

外部からの引数の取り込み
================================

　引数は配列argvに入っており、スクリプト内で利用できます。

* Fig.: args1.glue

.. code-block:: bash

        /bin/echo argv[1] argv[2]

実行してみましょう。

.. code-block:: bash

        $ glue ./args1.glue abc アイウエオ
        abc アイウエオ


手続きでの引数
================================

　手続きでも同様に引数が使えます。

Fig.: args2.glue 

.. code-block:: bash

	#!/usr/local/bin/glue
	
	proc hoge = do
	  /bin/echo argv[1] >>= /usr/bin/rev
	  /bin/echo argv[2]
	
	this.hoge 'abc' 'OK'

実行するとこのようになります。

.. code-block:: bash

	$ ./args2.glue 
	cba
	OK

