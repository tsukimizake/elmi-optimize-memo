# elmi-optimize-memo

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
的な感じ

理想としては

```hs
TAlias (author.project.Main.Three)
  [(a, elm.core.Basics.Int[])] 
  (Record [(p, a), (q, a), (r,a)])
```
のような感じか

右辺に型引数名の `a` も残してくれていると楽だったのだが `elm.core.Basics.Int` しか残ってないので思ったよりちょっとだるそう

CanonicalizeでTAliasを作っている部分から弄るか？
