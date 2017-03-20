==============================
エラー処理
==============================

　GlueLangでは、従来のシェルよりもエラーの表示を重視しています。
また、エラーが起こるとその場で処理が止まります。
ただし、条件文の中やループの終了判断時にはその限りでなく、
少しルールがあります。

エラーの表示
------------------------------

　エラーが起こると、エラーの箇所と付随情報が表示されます。
次の例はdiffを単体で呼び出した時のエラー表示です。

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

付随情報としては、図のように、エラーの原因（Command error）、
エラーの起きたシェルのレベル（この場合は最上位の0）、
コマンドの終了ステータス（2）が表示されています。

終了ステータス
------------------------------

　GlueLangはsh系のシェルと異なり、
コマンドの終了ステータスをそのまま返すことはしません。
例えば、次の例では先ほどのスクリプトを実行した後に終了ステータスを確認していますが、
diffの終了ステータスは2、glueの終了ステータスは1になっています。

.. code-block:: bash

        $ cat simple_error.glue 
        import PATH

        diff
        $ glue ./simple_error.glue 
        （略。diffのエラー）
                exit_status 2
        （略）
        $ echo $?
        1
	

　このようにする理由は、多くのシェルのように最後のコマンドの終了ステータスをそのまま返すと、
glue自体のエラーの終了ステータスと切り分けられないという問題を解決するためです。
また、こうしておくと、
loopブロックの中のスクリプトのバグでスクリプト全体を止めることができるようになります。

　次のサンプルは、loopの中でコマンドがエラーを起こす場合と、
glueがコマンドを探せないという原因でエラーを起こす場合の
loopブロックの挙動の違いを確認するためのものです。

* 図: loop_stop.glue 

.. code-block:: bash

	import PATH
	
	loop
	  false  #これはコマンドのエラーなので処理続行
	
	echo 'a' #これは実行される
	
	loop
	  falce  #存在しないコマンドの呼び出しはglueのエラー
	
	echo 'b' #これは実行されない
	

実行すると、 ``echo 'b'`` が実行されないことが分かります。
最終行に「b」が出てきません。

.. code-block:: bash

	$ glue ./loop_stop.glue 
	a
	
	Parse error at line 1, char 1
		line1: falce  #存在しないコマンドの呼び出しはglueのエラー
		       ^
	
		Command falce not exist
		
		process_level 1
		exit_status 2
		pid 14395
	
		glue exit_status: 2
	
	Execution error at line 8, char 1
		line8: loop
		       ^
		line9:   falce  #存在しないコマンドの呼び出しはglueのエラー
		line10: 
	
		Command error
		
		process_level 0
		exit_status 1
		pid 14389
