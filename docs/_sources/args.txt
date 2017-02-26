================================
引数
================================

　引数は配列argvに入っており、スクリプト内で利用できます。

* Fig.: args1.glue

.. code-block:: bash

        /bin/echo argv[1] argv[2]

実行してみましょう。

.. code-block:: bash

        $ glue ./args1.glue abc アイウエオ
        abc アイウエオ

