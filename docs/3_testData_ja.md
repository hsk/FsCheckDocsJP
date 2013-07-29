## Test data: generators, shrinkers and Arbitrary instances

## テストデータ：ジェネレータ、シュリンカーと任意インスタンス

Test data is produced by test data generators. FsCheck defines default generators for some often used types, but you can use your own, and will need to define your own generators for any new types you introduce.

テストデータは、テストデータジェネレータによって生成される。 FsCheckは、いくつかのよく使われるタイプのデフォルトジェネレータが定義されていますが、定義した新しいタイプのために独自のジェネレータを定義する事で、独自のジェネレータを使う事が出来ます。

Generators have types of the form Gen<'a>; this is a generator for values of type a. For manipulating values of type Gen, a computation expression called gen is provided by FsCheck, and all the functions in the Gen module are at your disposal.

ジェネレータはGen<'a>形式の型を持っていて、これは型aの値のためのジェネレータです。
Gen型の値を操作するための,genというコンピュテーション式はFsCheckによって提供され、Genモジュールのすべての関数を自由に利用できます。

Shrinkers have types of the for 'a -> seq<'a>; given a value, a shrinker produces a sequence of values that are in some way smaller than the given value. If FsCheck finds a set of values that falsify a given property, it will try to make that value smaller than the original (random) value by getting the shrinks for the value and trying each one in turn to check that the property is still false. If it is, the smaller value becomes the new counter example and the shrinking process continues with that value.

シュリンカーは 'a -> seq<'a> のための型を持っていて、与えられた値が、シュリンカーが特定の値より小さいいくつかの方法である値のシーケンスを生成します。 FsCheckが指定されたプロパティを改ざん値のセットを見つけた場合、その値を縮小を取得し、プロパティがまだ偽であることを確認するために順番にそれぞれを試みることによって、元の（ランダム）値よりもその値を小さくしようとします。そうである場合、値が小さいほど新しい反例となり、縮小処理は、その値を続行します。

Shrinkers have no special support from FsCheck - this is not needed, since you have all that you need in seq computation expressions and the Seq module.
Finally, an Arbitrary<'a> instance packages these two types to be used in properties. FsCheck also allows you to register Arbitrary instances in a Type to Arbitrary dictionary. This dictionary is used to find an arbitrary instance for properties that have arguments, based on the argument's type.
Arbitrary instances have some helper functions in the Arb module.

ShrinkersはFsCheckからの特別なサポートを持っていない - あなたは Seq のコンピュテーション式およびSeqモジュールで必要なすべてを持っているので、これは必要ありません
最後に、Arbitrary<'a>インスタンスパッケージプロパティで使用されるこれらの2つのタイプ。 FsCheckはまた、Arbitrary DictionaryタイプでArbitraryインスタンスを登録することができます。このDictionaryは、引数の型に基づいて引数を持つプロパティ、に任意のインスタンスを検索するために使用されます。
Arbitraryインスタンスは、Arbモジュール内にいくつかのヘルパー関数を持っています。

#### Generators

#### ジェネレータ

Generators are built up from the function

ジェネレータは、関数から構築されている

```fsharp
val choose : (int * int -> Gen<int>)
```

which makes a random choice of a value from an interval, with a uniform distribution. For example, to make a random choice between the elements of a list, use

これは、一様分布と、間隔の値のランダムな選択肢となります。例えば、リストの要素の間のランダムな選択を行うために

```fsharp
let chooseFromList xs = 
    gen { let! i = Gen.choose (0, List.length xs-1) 
          return (List.nth xs i) }
```

を使用します

#### Choosing between alternatives

#### 代替案のどちらを選択するか

A generator may take the form
Gen.oneof <sequence of generators>
which chooses among the generators in the list with equal probability. For example,

ジェネレータはGen.oneof<sequence generators>の形を取ることができ、
それは等しい確率で、リスト内の発電機の間に選択します。たとえば、以下のプログラムは

```fsharp
Gen.oneof [ gen { return true }; gen { return false } ]
```

generates a random boolean which is true with probability one half.
We can control the distribution of results using the function

確率半分との真のランダムなブール値を生成します。
関数を使用することで結果の分布を制御することができます。

```fsharp
val frequency: seq<int * Gen<'a>> -> Gen<'a>
```

instead. Frequency chooses a generator from the list randomly, but weighs the probability of choosing each alternative by the factor given. For example,

代わりに。周波数がランダムにリストからジェネレータを選択しますが、与えられた係数で各選択肢を選択する確率を量る。例えば、


```fsharp
Gen.frequency [ (2, gen { return true }); (1, gen { return false })]
```    

generates true two thirds of the time.

本来の数の三分の二を生成します。

#### The size of test data

#### テストデータのサイズ

Test data generators have an implicit size parameter; FsCheck begins by generating small test cases, and gradually increases the size as testing progresses. Different test data generators interpret the size parameter in different ways: some ignore it, while the list generator, for example, interprets it as an upper bound on the length of generated lists. You are free to use it as you wish to control your own test data generators. 
You can obtain the value of the size parameter using

テストデータジェネレータは、暗黙のサイズパラメータを持っている; FsCheckは、小さなテストケースを生成することで始まり、徐々にテストが進むにつれてサイズが大きくなります。別のテストデータジェネレータは、異なる方法でサイズパラメータを解釈：リストジェネレータは、例えば、生成されたリストの長さの上限としてそれを解釈しながらいくつかは、それを無視する。あなたは、あなた自身のテストデータ生成を制御するために望むようにそれを自由に使用できます。
あなたが使用してサイズパラメータの値を得ることができ、

```fsharp
val sized : ((int -> Gen<'a>) -> Gen<'a>)
```    

sized g calls g, passing it the current size as a parameter. For example, to generate natural numbers in the range 0 to size, use

サイズgはそれをパラメータとして現在のサイズを渡して、gの呼び出します。大きさ0の範囲の自然数を生成する例は以下のようになります。

```fsharp
Gen.sized <| fun s -> Gen.choose (0,s)
```

The purpose of size control is to ensure that test cases are large enough to reveal errors, while remaining small enough to test fast. Sometimes the default size control does not achieve this. For example, towards the end of a test run arbitrary lists may have up to 50 elements, so arbitrary lists of lists may have up to 2500, which is too large for efficient testing. In such cases it can be useful to modify the size parameter explicitly. You can do so using

サイズ制御の目的は、高速テストするために十分に小さいままテスト·ケースが、エラーを明らかにするのに十分な大きさであることを確認することである。時には、デフォルトのサイズ制御は、これを達成することはありません。例えば、テスト実行任意のリストの終わりに向かっては、効率的なテストのために大きすぎである、50の要素を、リストのように任意のリストは2500年までがあり、最大可能性があります。このような場合、明示的サイズパラメータを変更するために有用であり得る。あなたが使用して行うことができます

```fsharp
val resize : (int -> Gen<'a> -> Gen<'a>)
```

resize n g invokes generator g with size parameter n. The size parameter should never be negative. For example, to generate a random matrix it might be appropriate to take the square root of the original size:

リサイズngのは、sizeパラメータnの生成元gを呼び出します。 sizeパラメータは、マイナスになることはありません。例えば、それは元のサイズの平方根を取るために適切かもしれないランダム行列を生成する：

```fsharp
let matrix gen = Gen.sized <| fun s -> Gen.resize (s|>float|>sqrt|>int) gen
```

#### Generating recursive data types

#### 再帰的なデータタイプを生成する

Generators for recursive data types are easy to express using oneof or frequency to choose between constructors, and F#'s standard computation expression syntax to form a generator for each case. There are also map functions for arity up to 6 to lift constructors and functions into the Gen type. For example, if the type of trees is defined by

再帰的なデータ型の発電機は、各ケースの発電機を形成するためにコンストラクタの間で選択するoneofまたは周波数を用いて、F＃の標準的な計算式の構文表現することが容易である。世代型にコンストラクタや関数を持ち上げるために、最大6〜アリティのマップ機能もあります。たとえば、木の種類が定義されている場合で

```fsharp
type Tree = Leaf of int | Branch of Tree * Tree
```

then a generator for trees might be defined by

Treeのためのジェネレータは以下のように定義されるかもしれません

```fsharp
let rec unsafeTree() = 
    Gen.oneof [ Gen.map Leaf Arb.generate<int> 
                Gen.map2 (fun x y -> Branch (x,y)) (unsafeTree()) (unsafeTree())]
```

However, a recursive generator like this may fail to terminate with a StackOverflowException, or produce very large results. To avoid this, recursive generators should always use the size control mechanism. For example,

しかし、このような再帰的なジェネレータでStackOverflowExceptionで終了に失敗したり、非常に大きな結果を生む可能性があります。これを回避するには、再帰ジェネレータは常にサイズ制御機構を使用してください。例えば、

```fsharp
let tree =
    let rec tree' s = 
        match s with
        | 0 -> Gen.map Leaf Arb.generate<int>
        | n when n>0 -> 
            let subtree = tree' (n/2)
            Gen.oneof [ Gen.map Leaf Arb.generate<int> 
                        Gen.map2 (fun x y -> Branch (x,y)) subtree subtree]
        | _ -> invalidArg "s" "Only positive arguments are allowed"
    Gen.sized tree'
```

Note that 
* We guarantee termination by forcing the result to be a leaf when the size is zero. 
* We halve the size at each recursion, so that the size gives an upper bound on the number of nodes in the tree. We are free to interpret the size as we will. 
* The fact that we share the subtree generator between the two branches of a Branch does not mean that we generate the same tree in each case.

以下の点に注意してください

- 私たちは、サイズがゼロの場合、結果が葉であることが強制的に終了を保証する。
- 私たちは、サイズは、ツリー内のノードの数に上限を与えているので、それぞれの再帰で大きさを半分に。我々は意志としてサイズを解釈するために自由です。
我々は、支店の2支店間のサブツリージェネレータを共有する
- 事実は、我々は、それぞれの場合に同じツリーを生成することを意味するものではありません。

#### Useful Generator Combinators

If g is a generator for type t, then 

gがt型用のジェネレータである場合は、

- `two g` generates a pair of t's, 
- `three g` generates a triple of t's, 
- `four g` generates a quadruple of t's, 
- If xs is a list, then `elements xs` generates an arbitrary element of xs.
- `listOfLength n g` generates a list of exactly n t's. 
- `listOf g` generates a list of t's whose length is determined by the size parameter
- `nonEmptyListOf g` generates a non-empty list of t's whose length is determined by the size parameter.
- `constant v` generates the value v.
- `suchThat p g` generates t's that satisfy the predicate p. Make sure there is a high chance that the predicate is satisfied.
- `suchThatOption p g` generates Some t's that satisfy the predicate p, and None if none are found. (After 'trying hard')

All the generator combinators are functions on the Gen module.

- `two g` Tののペアを生成
- `tree g` tの年代のトリプルを生成
- `four g`、Tの4倍を生成
- xsがリストであれば、`elements xs`はxsの任意の要素を生成します。
- `listOfLength n g`正確にTのn個のリストを生成します。
- `listOf g`その長さがsizeパラメータによって決定されTののリストを生成する
- `nonEmptyListOf g` tのの長さがsizeパラメータによって決定されるの非空のリストを生成します。
- `constant v`値を生成対
- `suchThat p g` tのの述語pを満たすことを生成します。述語が満たされていることに高い可能性があることを確認してください。
- `suchThatOption p g`何も検出されない場合、いくつかのTのそれは述語pを満たす、[なし]が生成されます。 （後に'が頑張って'）

すべてのジェネレータコンビネータはGenモジュールの関数です。

#### Default Generators and Shrinkers based on type

#### デフォルトのジェネレータとShrinkers基づいたタイプ

FsCheck defines default test data generators  and shrinkers for some often used types: unit, bool, byte, int, float, char, string, DateTime, lists, array 1D and 2D, Set, Map, objects and functions from and to any of the above. Furthermore, by using reflection, FsCheck can derive default implementations of record types, discriminated unions, tuples and enums in terms of any primitive types that are defined (either in FsCheck or by you).

unit、bool、byte、int、float、char、string、DateTime、リスト、1次元および2次元配列、Set(集合)、Map、オブジェクトと関数からのいずれかに：FsCheckは、上記したいくつかのよく使われるタイプのデフォルトのテストデータジェネレータとshrinkersを定義しています。さらに、リフレクションを使用して、FsCheckは、デフォルトのレコード·タイプの実装は、判別共用体、タプルと（FsCheck中またはいずれかによって）定義されている任意のプリミティブ型の観点から列挙型を派生させることができます。

You do not need to define these explicity for every property: FsCheck can provide a property with appropriate generators and shrinkers for all of the property's arguments, if it knows them or can derive them. Usually you can let type inference do the job of finding out these types based on your properties. However if you want to coerce FsCheck to use a particular generator and shrinker, you can do so by providing the appropriate type annotations.

あなたは、すべてのプロパティのためにこれらの明示的に定義する必要はありません。FsCheckは、プロパティのすべての引数のための適切な発電機やshrinkers持つプロパティを提供することができ、それが彼らを知っているか、それらを引き出すことができる場合。通常は、型推論はあなたのプロパティに基づいて、これらのタイプを見つけるの仕事をさせることができます。特定のジェネレータとシュリンカーを使用するFsCheckを強制したい場合は、適切な型の注釈を提供することによって、そうすることができます。

As mentioned in the introduction, FsCheck packages a generator and shrinker for a particular type in an Arbitrary type. You can provide FsCheck with an Arbitrary instance for your own types, by defining static members of a class, each of which should return an instance of a subclass of the class Arbitrary<'a>:

冒頭で述べたように、FsCheckは任意の型の特定のタイプのジェネレータとシュリンカーをパッケージ。あなたはクラスの任意のサブクラスのインスタンスを返さなければなりません、それぞれのクラスの静的メンバを定義することによって、あなた自身のタイプのための任意のインスタンスとFsCheckを提供することができます<'a>を：

```fsharp
type MyGenerators =
    static member Tree() =
        {new Arbitrary<Tree>() with
            override x.Generator = tree
            override x.Shrinker t = Seq.empty }
```

Replace the 'a by the particular type you are defiing an Arbitary instance for. Only the Generator method needs to be defined; Shrinker by default returns the empty sequence (i.e. no shrinking will occur for this type).
Now, to register all Arbitrary instances in this class:

あなた用Arbitaryインスタンスをdefiingしている特定のタイプによって'交換してください。唯一のジェネレータ方法を定義する必要があり、デフォルトでは、シュリンカー（つまり全く収縮は、このタイプの発生しません）空のシーケンスを返します。
さて、このクラスのすべての任意のインスタンスを登録します

    Arb.register<MyGenerators>()

FsCheck now knows about Tree types, and can not only generate Tree values, but also e.g. lists, tuples and option values containing Trees:

FsCheckは現在、Treeの型を知っているし、Treeの値を生成することができますだけでなく、例えばリスト、タプルと木を含むオプション値：

```fsharp
let RevRevTree (xs:list<Tree>) = 
List.rev(List.rev xs) = xs

> Check.Quick RevRevTree;;

Ok, passed 100 tests.
```    

To generate types with a generic type argument, e.g.:

ジェネリック型引数を持つ型を生成するには、例えば：

```fsharp
type Box<'a> = Whitebox of 'a | Blackbox of 'a
```

you can use the same principle. So the class MyGenerators can be writtten as follows:

あなたは、同じ原理を使用することができます。だからクラスMyGeneratorsは、次のように書く事ができます：

```fsharp
let boxGen<'a> : Gen<Box<'a>> = 
    gen { let! a = Arb.generate<'a>
          return! Gen.elements [ Whitebox a; Blackbox a] }

type MyGenerators =
    static member Tree() =
        {new Arbitrary<Tree>() with
            override x.Generator = tree
            override x.Shrinker t = Seq.empty }
    static member Box() = Arb.fromGen boxGen
```

Notice that we use the function 'val generate<'a> : Gen<'a>' from the Arb module to get the generator for the type argument of Box. This allows you to define generators recursively. Similarly, there is a function shrink<'a>. Look at the FsCheck source for examples of default Arbitrary implementations to get a feeling of how to write such Arbitrary instances. The Arb module should help you with this task as well.
Now, the following property can be checked:

我々は、機能を使用することに注意してください'valの生成を<' a>を：ジェン<'a>の'箱のタイプ引数にジェネレータを取得する任意波形モジュールから。これは、再帰的にジェネレータを定義することができます。同様に、関数がシュリンクがある<'a>を。このような任意のインスタンスを作成する方法の感覚を得るために、デフォルトの任意の実装の例についてFsCheckソースを見てください。任意波形モジュールは同様に、この作業のお手伝いをする必要があります。
さて、次のプロパティをチェックすることができます：

```fsharp
let RevRevBox (xs:list<Box<int>>) = 
    List.rev(List.rev xs) = xs
    |> Prop.collect xs

> Check.Quick RevRevBox;;
Ok, passed 100 tests.
11% [].
2% [Blackbox 0].
1% [Whitebox 9; Blackbox 3; Whitebox -2; Blackbox -8; Whitebox -13; Blackbox -19;
(etc)
```    

Note that the class needs not be tagged with attributes in any way. FsCheck determines the type of the generator by the return type of each static member.
Also note that in this case we actually didn't need to write a generator or shrinker: FsCheck can derive suitable generators using reflection for discriminated unions, record types and enums.

クラスは、任意の方法で属性でタグ付けされていない必要があることに注意してください。 FsCheck各静的メンバの戻り値の型によってジェネレータのタイプを決定します。
この場合にも、我々は実際にジェネレータやシュリンカーを記述する必要はなかったことに注意してください：FsCheckは判別共用体、レコードタイプと列挙用リフレクションを使用して、適切なジェネレータを導き出すことができます。

#### Some useful methods on the Arb module

#### Arbモジュール上のいくつかの便利なメソッド

- `Arb.from<'a>` returns the registered Arbitrary instance for the given type 'a
- `Arb.fromGen` makes a new Arbitrary instance from just a given generator - the shrinker return the empty sequence
- `Arb.fromGenShrink` make a new Arbitrary instance from a given generator and shrinker. This is equivalent to implementing Arbitrary yourself, but may be shorter.
- `Arb.generate<'a>` returns the generator of the registered Arbitrary instance for the given type 'a
- `Arb.shrink` return the immediate shrinks of the registered Arbitrary instance for the given value
- `Arb.convert` given conversion functions to ('a ->'b) and from ('b ->'a), converts an Arbitrary<'a> instance to an Arbitrary<'b>
- `Arb.filter` filters the generator and shrinker for a given Arbitrary instance to contain only those values that match with the given filter function
- `Arb.mapFilter` maps the generator and filter the shrinkers for a given Arbitrary instance. Mapping the generator is sometimes faster, e.g. for a PositiveInt it is faster to take the absolute value than to filter the negative values.
- `Arb.Default` is a type that contains all the default Arbitrary instances as they are shipped and registerd by FsCheck by default. This is useful when you override a default generator - typically this is because you want to filter certain values from it, and then you need to be able to refer to the default generator in your overriding generator.

- `Arb.from<'a>の`は、指定されたタイプ用登録されている任意のインスタンスを返します。'
- `Arb.fromGenは`だけ与えられた発電機から新しい任意のインスタンスを作る - シュリンカーは、空のシーケンスを返す
- `Arb.fromGenShrink`は与えられた発電機とシュリンカーからの新しい任意のインスタンスを作る。これは、自分で任意の実装と同じですが、短くなることがあります。
- `Arb.generate<'a>の`は、指定された型用登録される任意のインスタンスの生成元を返す'
- `Arb.shrink`は与えられた値用登録されている任意のインスタンスの即時縮小を返す
- `Arb.convert`に与えられた変換関数（ 'A - >' b）およびから（ 'B - >'）、任意の<に変換'へa>のインスタンスを任意<' b>の
- `Arb.filter`は与えられたフィルタ機能と一致する値のみが含まれるように指定された任意のインスタンス用ジェネレータとシュリンカーをフィルタリング
- `Arb.mapFilterは`ジェネレータをマッピングし、与えられた任意のインスタンスに対してshrinkersをフィルタリング。ジェネレータのマッピング、時には速く例えばですPositiveIntにとっては負の値をフィルタリングすることよりも、絶対値を取るために高速です。
- `Arb.Default`はそれらが出荷され、デフォルトでFsCheckによってブックマークへ登録されているように、すべてのデフォルトの任意のインスタンスを含むタイプです。デフォルトのジェネレータを上書きする場合に便利です - 一般的に、あなたはそれから特定の値をフィルタしたいのでこれは、その後、あなたのオーバーライドジェネレータのデフォルトのジェネレータを参照できるようにする必要があります。
