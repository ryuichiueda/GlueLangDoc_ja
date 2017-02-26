=================================
手続き
=================================

　手続きはシェルの「関数」に相当するものです。
次のように定義、実行します。
実行するときは、このスクリプトを指す ``this``
という接頭語を付けます。

* Fig.: proc.glue

.. code-block:: bash
	
	#definition
	proc f = /bin/echo 'HELLO'
	
	#execution
	this.f

実行すると手続きの中のechoが呼び出されます。
	
.. code-block:: bash
	
	$ glue proc1.glue 
	HELLO
	
