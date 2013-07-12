クイックスタート
================

簡単な例
--------

プロパティ(性質)の定義の簡単な例は

.. code-block::fsharp

  let revRevIsOrig (xs:list<int>) = List.rev(List.rev xs) = xs

この性質は、リストを逆順にして逆順にしたものは元のリスト自身になるということを主張します。この性質を確かめるために、F# インタラクティブ上にこの定義をロードして、起動してみましょう。

.. code-block::fsharp

  > Check.Quick revRevIsOrig;;
  Ok, passed 100 tests.

性質が失敗すると、FsCheck は反例を表示します。例えば、もし

.. code-block::fsharp

  let revIsOrig (xs:list<int>) = List.rev xs = xs

を定義して、

.. code-block::fsharp

  > Check.Quick revIsOrig;;
  Falsifiable, after 2 tests (2 shrinks) (StdGen (884019159,295727999)):
  [1; 0]

という結果を確認します。

FsCheck は反例のシュリンク [#]_ も行うので、それがテストケースを失敗させる最小の反例となるのです。上の例では反例は確かに最小、つまりリストは少なくとも2つの異なる要素を持たなくてはならないのです。FsCheck はより小さな反例を(何らかの方法で)何回見つけて、よりシュリンクしたかを表示します。