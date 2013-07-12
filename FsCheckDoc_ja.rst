クイックスタート
================

簡単な例
--------

プロパティ(性質)の定義の簡単な例は

::

  let revRevIsOrig (xs:list<int>) = List.rev(List.rev xs) = xs

この性質は、リストを逆順にして逆順にしたものは元のリスト自身になるということを主張します。この性質を確かめるために、F# インタラクティブ上にこの定義をロードして、起動してみましょう。

::

  > Check.Quick revRevIsOrig;;
  Ok, passed 100 tests.

性質が失敗すると、FsCheck は反例を表示します。例えば、もし

::

  let revIsOrig (xs:list<int>) = List.rev xs = xs

を定義して、

::

  > Check.Quick revIsOrig;;
  Falsifiable, after 2 tests (2 shrinks) (StdGen (884019159,295727999)):
  [1; 0]

という結果を確認します。