# elmi-optimize-memo

## そもそもの問題とやりたいこと

でかいプロジェクトのコンパイルが遅い。プロファイルを取ってみたら時間の9割がelmi(モジュールをコンパイルした結果として吐かれる型情報ファイル)のパースに取られていた。
実装を見てみると全てをベタ書きするようなエンコードをしていて最適化の余地がたくさんあるので魔改造していきたい。

まずカイシャプロジェクトで最も問題になっているレコードのtype aliasから。

### type aliasの問題

```elm
type alias Three a = {p: a, q: a, r: a}

hoge: Three Int
...
```

このhogeの型は

```hs
      (TAlias 
        (Canonical {_package = Name {_author = author, _project = project}, _module = Main})
        Three 
        [(a,TType (Canonical {_package = Name {_author = elm, _project = core}, _module = Basics}) Int [])]
        (Filled (TRecord (fromList [(p,FieldType 0 (TType (Canonical {_package = Name {_author = elm, _project = core}, _module = Basics}) Int [])),(q,FieldType 0 (TType (Canonical {_package = Name {_author = elm, _project = core}, _module = Basics}) Int [])),(r,FieldType 0 (TType (Canonical {_package = Name {_author = elm, _project = core}, _module = Basics}) Int []))]) Nothing))
      ) 
```
こうなっている

読みやすくすると

```hs
TAlias (author.project.Main.Three)
  [(a, elm.core.Basics.Int[])]
  (Record [
    (p, elm.core.Basics.Int[])
  , (q, elm.core.Basics.Int[])
  , (r, elm.core.Basics.Int[])
  ])
```
的な感じ。aliasに型引数を渡す度に中身が(使用回数+1)回ベタ書きされるので大きなプロジェクトだと大変なことになる。


#### 改善案1
左辺に出てきた型を右辺で再度書かないようにする。

```hs
TAlias (author.project.Main.Three)
  [(a, elm.core.Basics.Int[])] 
  (Record [(p, a), (q, a), (r,a)])
```
のような感じか

右辺に型引数名の `a` も残してくれていると楽だったのだが `elm.core.Basics.Int` しか残ってないので思ったよりちょっとだるそう

バイナリエンコード部分で部分木の一致を見つけるのは文脈持ち回りながらエンコードをするのは計算量が(型式のサイズ^2)になりそう、CanonicalizeでTAliasを作っている部分から弄るほうがいいか？


#### 改善案1の問題点
社プロジェクトだと

```elm

type alias Shared = {
      companies : Dict Id Company
     ,user : Dict Id User
     , ...
}
```

のようにDict型がたくさん並んだレコードが最も大きくなっている。
改善案1でもDictの型のサイズは半分に減り、このShared型もほぼ半分になるのだが、似たような構造をさらにくくり出せるのでは？みたいな気持ちはある


### function
同様に、

```elm
huga : Three -> Three -> Three
```
のような型もThreeが3つ展開される。
これも引数と返り値に出てくる型を全て最初に列挙してやれば倍々ゲームは避けられるはず
