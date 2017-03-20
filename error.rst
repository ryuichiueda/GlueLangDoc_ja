==============================
エラー処理
==============================

終了ステータス
------------------------------

　GlueLangはsh系のシェルと異なり、
コマンドの終了ステータスをそのまま返すことはしません。
例えば、次の例ではdiffを単体で呼び出していますが、
diffの終了ステータスは2、glueの終了ステータスは1となります。

* 図: simple_error.glue 

.. code-block:: bash

	$ cat simple_error.glue 
	import PATH

	diff
	$ glue ./simple_error.glue 
	（略。diffのエラー）
	
	Execution error at line 3, char 1
		line3: diff
		       ^
	
		Command error
		
		process_level 0
		exit_status 2
		pid 14247
	
	$ echo $?
	1
	
