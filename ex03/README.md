# 課題3 (2016/6/20)

次は，(新たな関数を書くのではなく) [課題2](../ex02)で作った関数に many-to-one マッチング
(college admission のケース) を扱う機能を加えましょう．

とりあえずは[昨年のページ](https://github.com/OyamaZemi/exercises2015/tree/master/ex03)を参照してください．

今回は Julia なので `resp_matches[indptr[j]:indptr[j+1]-1]` で大学 `j` が受け入れる学生のリストが返されるようになります．
`indptr` は次のようにして作ることができます：

```jl
indptr = Array(Int, n+1)
indptr[1] = 1
for i in 1:n
    indptr[i+1] = indptr[i] + caps[i]
end
```


## 一対一 vs. 多対一

たとえば，次のように書くようにしてみましょう．

```jl
# 多対一のケース
function deferred_acceptance(prop_prefs::Matrix{Int},
                             resp_prefs::Matrix{Int},
                             caps::Vector{Int})

    # 多対一のコードを書く

    return prop_matches, resp_matches, indptr
end

# 一対一のケース
function deferred_acceptance(prop_prefs::Matrix{Int}, resp_prefs::Matrix{Int})
    caps = ones(Int, size(resp_prefs, 2))
    prop_matches, resp_matches, indptr =
        deferred_acceptance(prop_prefs, resp_prefs, caps)
    return prop_matches, resp_matches
end
```

ここで，複数スロットを持つ方 (つまり大学) が respondants であることを想定しています．
(このケースで書けたら，逆のケースも書いてみる．あるいは最初から多対多で書いておいてもよい．)


## 単体テスト

Many-to-one 用にもテストを作ったので，それが通るようにコードを書きましょう．

[test_deferred_acceptance.jl](https://raw.githubusercontent.com/OyamaZemi/exercises2016/d8a41a8929d67110109cb08d0557de86767fcd05/ex03/test_deferred_acceptance.jl)
を作業フォルダにダウンロードする．
課題2と同じフォルダで作業しているのなら，前回のテストファイルに上書きする．

前回と同様

```jl
const file_name = "deferred_acceptance.jl"
const function_name = "deferred_acceptance"
```

を自分の命名に合わせて書きかえる．

Julia ウィンドウあるいは notebook で `include("test_deferred_acceptance.jl");` として，

```
Test Summary:                  | Pass  Total
Testing deferred_acceptance.jl |   10     10
```

のようになれば，テストが通ったということ．


## ランダム選好リスト

`caps` も返すように拡張しました．

1. 使い方1  
   [新しい `matching_tools.jl`](https://raw.githubusercontent.com/oyamad/Matching.jl/c5f86f101eeb75f5c93d80b0726be2253a7fc765/src/matching_tools.jl)
   を作業フォルダにダウンロードし，`include("matching_tools.jl")` とする．

2. 使い方2  
   パッケージ全体を `Pkg.clone("https://github.com/oyamad/Matching.jl")` でインストールし，`using Matching` とする．
   前回 `Pkg.clone` している人は「すでにある」と言われるだろうから，その場合は `Pkg.update()` でアップデートする．

* [使用例](http://nbviewer.jupyter.org/github/oyamad/Matching.jl/blob/0df7172f47a62a63c48cee28d70c9f7d1b38de46/examples/random_prefs.ipynb)


## ゼミ生の成果物

* [Notebook リスト](notebooks.md)


## パフォーマンス比較の結果 (2016/7/4)

* [結果](http://nbviewer.jupyter.org/github/OyamaZemi/exercises2016/blob/84cc1244ae41cd8f3fdc7a58e0a1f4a349e5a513/ex03/competition_many_to_one.ipynb)


## 既存研究のシミュレーションの再現

0. I. Ashlagi, Y. Kanoria, and J. D. Leshno (2015).
   "[Unbalanced Random Matching Markets: The Stark Effect of Competition](http://web.stanford.edu/~iashlagi/papers/UnbalancedMatchingAKL.pdf)."


## その他の拡張

* 以下の論文の Section 4 "Common quotas: nested set systems" のアルゴリズムを実装してみる：
  
  * P. Biró, T. Fleiner, R. W. Irving, and D. F. Manlove (2010).
    "[The College Admissions problem with lower and common quotas](http://www.sciencedirect.com/science/article/pii/S0304397510002860),"
    Theoretical Computer Science 411, 3136-3153.
