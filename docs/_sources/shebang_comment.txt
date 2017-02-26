==============================
シバンとコメント
==============================

シバン（shebang）
==============================

　glueのパスを指定することで、
スクリプトをコマンドとして扱えるようになります。
例えば次のようなスクリプトを作ったら、

.. code-block:: bash

        $ cat shebang.glue 
        #!/usr/local/bin/glue
        
        /bin/echo 'shebang'

次のように ``chmod(1)`` で実行権限を与えて実行できます。

.. code-block:: bash

        $ chmod +x shebang.glue        #実行権限をつける
        $ ./shebang.glue               #実行
        shebang


　glueコマンドのインストール場所は、次のように ``which(1)`` で調査できます。

.. code-block:: bash

        $ which glue
        /usr/local/bin/glue
