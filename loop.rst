===============================
ループ
===============================

　GlueLangの繰り返しの機能は、現在のところとても貧弱です。
内部コマンドとして ``repeat`` と ``while`` が実装されています。

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
	
