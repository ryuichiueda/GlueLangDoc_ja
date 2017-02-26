=================================
ジョブの管理
=================================

　GlueLangでは、ジョブに名前をつけることができます。
次の例は、3,4,5行目のジョブにそれぞれa,b,cと名前をつけて実行し、内部コマンド ``in.wait`` で待ってから8行目の処理をするというものです。

Fig.: job.glue

.. code-block:: bash
        :linenos:
	
	import PATH
	
	sleep 3 >> echo 'a' >>= sed 's/./&&/' & a
	sleep 2 >> echo 'b' >>= sed 's/./&&/' & b
	sleep 1 >> echo 'c' >>= sed 's/./&&/' & c
	
	in.wait a b c
	echo 'd'


実行してみましょう。 ``sleep`` の秒数に対応して、c,b,aの順に処理が終了し、``in.wait`` でジョブ待ちを行なった後、最後に8行目の ``echo 'd'`` が実行されます。

.. code-block:: bash

	$ glue jobs.glue 
	cc
	bb
	aa
	d
	
