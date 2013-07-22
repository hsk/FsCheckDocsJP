性質
====

性質は F# の関数定義として表現されます。性質はそのパラメータで以って例外なく定量化されます、だから

.. code-block:: fsharp

  let revRevIsOrig xs = List.rev(List.rev xs) = xs

は等式は全てのリスト xs で成り立つのです。性質は必ず単相型をもってはいけません。上記のような「多層的な」性質は、まるでジェネリックな引数が型オブジェクトであるかのようにテストされます。つまり、様々な単純型(bool, char, string, ...)が生成されることを意味します。それは生成された1つのリストに複数の型が含まれる場合があるかも知れない、例えば {['r', "1a", true]} が上記の性質でチェックされるため使われうるということです。ですが生成された値は型をベースにしているので、xs に異なる推論された型や明示的な型を与えることで振る舞いを単純に変更することもできます:

.. code-block:: fsharp

  let revRevIsOrigInt (xs:list<int>) = List.rev(List.rev xs) = xs

は int のリストについてのみチェックされます。FsCheck は様々な形式の性質をチェックすることができます―これらの形式はテスト可能と呼ばれ、'Testable というジェネリック型によって API において示されます。'Testable は bool 値または unit を返す任意のパラメータを取る関数となるでしょう。後者の場合、もし(例外を)送出しなければテストはパスします。

条件付き性質
------------

性質は ==> という形式をとることができます。例えば、

.. code-block:: fsharp

  let rec private ordered xs = 
     match xs with
     | [] -> true
     | [x] -> true
     | x::y::ys ->  (x <= y) && ordered (y::ys)

  let rec private insert x xs = 
     match xs with
     | [] -> [x]
     | c::cs -> if x <= c then x::xs else c::(insert x cs)

  let Insert (x:int) xs = ordered xs ==> ordered (insert x xs)

条件が成り立つ限り、もし ==> の後の性質が成り立つなら、このような性質は成り立ちます。テストは条件を満たさないテストケースを切り捨てます。条件を満たすケースが100件見つかるまで、あるいはテストケース数の限度に達する(条件が決して成り立たないループを避けるため)まで、テストケースの生成は続けられます。この場合、

.. code-block:: fsharp

  Arguments exhausted after 97 tests.

というようなメッセージが、条件を満たすテストケースが97件見つかり、その97件に対して性質が成り立ったということを示します。この場合、生成された値は int 型に制限されなければならなかったことに気が付きましょう。なぜなら生成された値は比較可能である必要がありましたが、これは型に反映されません。