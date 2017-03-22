==============================
エラー処理
==============================

　GlueLangでは、従来のシェルよりもエラーの表示を重視しています。
また、エラーが起こるとその場で処理が止まります。
ただし、条件文の中やループの終了判断時にはその限りでなく、
少しルールがあります。

終了ステータス（まだ実装は不十分）
==========================================================

　GlueLangの特徴の一つは、終了ステータスが、
呼び出したコマンドの終了ステータスとは独立していることです。
通常、Bシェル系の終了ステータスは、最後に呼び出したコマンドの返した値となりますが、
これでは最後のコマンドだけが特別扱いで、また、0以外の終了ステータスをシェルが返したときに、
スクリプトの書き方がまずかったのか、コマンドがエラーを返したのか、
外から区別がつきません。また、これは条件分岐のときにも文法エラーとしてスクリプトを止めるのか、
コマンドの終了ステータスに応じて分岐させるのかという判断も困難にします。
GlueLangでは、この問題を解決します。

　次は、終了ステータスの一覧です。

.. list-table:: 終了ステータス
   :widths: 40 20
   :header-rows: 1

   * - 状況
     - 終了ステータス
   * - コマンドが0以外の終了ステータスを返した
     - 1
   * - コマンドが見つからない
     - 2
   * - 変数などが未定義
     - 3
   * - forkエラー
     - 4（確認する方法募集中。forkbomb?）
   * - カレントディレクトリが取得できない
     - 5（同上）
   * - シグナルによる終了
     - 128+シグナル番号

エラーの表示
==============================

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
=============================

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
	  false  #これはコマンドのエラーなのでループを抜けるだけ
	
	echo 'a' #これは実行される
	
	loop
	  falce  #存在しないコマンドの呼び出しはglueのエラーなので処理が止まる
	
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
