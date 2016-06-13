# 課題2 (2016/6/6)
受入保留 (Deferred Acceptance) アルゴリズムを実装してみる．

[Wikipedia の記事](http://ja.wikipedia.org/wiki/安定結婚問題)なり原論文

* D. Gale and L.S. Shapley,
  "[College Admissions and the Stability of Marriage](http://www.jstor.org/stable/2312726),"
  American Mathematical Monthly 69 (1962), 9-15

なりをよく読んでコードを書いてみる．

* 最初は one-to-one (結婚のケース) でよいが，最終的には many-to-one (college admission のケース) を扱う．
  (各 college には受入上限がある．)
* 行く行くは「進学選択シミュレーション」のようなものをやりたい．
* 提出方法：GitHub のリポジトリにファイルを「プッシュ」する．
* 中間締め切り：6月13日 (月)
* 最終締め切り：6月18日 (土)


## 参考文献
* A.E. Roth,
  "[Deferred Acceptance Algorithms: History, Theory, Practice, and Open Questions](http://link.springer.com/article/10.1007/s00182-008-0117-6),"
  International Journal of Game Theory 36 (2008), 537-569


## データ構造

これがベストかわかりませんが，次のようなデータ構造を採用することにしましょう．

* 男性陣: `[1, ..., m]`
* 女性陣: `[1, ..., n]`

### 入力データ

* `m_prefs`: `(n+1)` x `m` の2次元配列  
  第 `i` 列 `m_prefs[:, i]` には男性 `i` が好きな女性の番号が順番に並べてある．
  ただし，`0` は "unmatched" を意味する．

* `f_prefs`: `(m+1)`x `n` の2次元配列  
  第 `j` 列 `f_prefs[j]` には女性 `j` が好きな男性の番号が順番に並べてある．
  ただし，`0` は "unmatched" を意味する．

たとえば，`n=4` として，`m_prefs[1]` が `[2, 3, 0, 4, 1]` だとしたら，
男性 `1` は「女性 `2` が一番好き，その次は女性 `3` が好き，その次は match しないのが好き
(女性 `4, 1` とペアになるくらいなら一人でいる方が好き)」であることを意味する．

### 出力データ

* `m_matched`: 長さ `m` の1次元配列  
  `m_matched[i]` は男性 `i` が match する女性の番号か `0` ("unmatched") が入る．
* `f_matched`: 長さ `n` の1次元配列  
  `f_matched[j]` は女性 `j` が match する男性の番号か `0` ("unmatched") が入る．


## ランダム選好リスト

ランダムに選好リストを生成する関数を書いたので必要があれば使ってみてください．

1. 使い方1  
   [matching_tools.jl](https://raw.githubusercontent.com/oyamad/Matching.jl/0e6d6c1daab949ba2ef4e08369bfce99dd3ce9aa/src/matching_tools.jl)
   を作業フォルダにダウンロードし，`include("matching_tools.jl")` とする．

2. 使い方2  
   パッケージ全体を `Pkg.clone("https://github.com/oyamad/Matching.jl")` でインストールし，`using Matching` とする．

* [使用例](http://nbviewer.jupyter.org/github/oyamad/Matching.jl/blob/2811aed218e1695fffb833554a9d30f449794680/examples/random_prefs.ipynb)

(仕様は今後変わる可能性があります．)


## 単体テスト

テストを作ったので，それが通るようにコードを書きましょう．

[test_deferred_acceptance.jl](https://raw.githubusercontent.com/OyamaZemi/exercises2016/bdfbb7e1992e9667ab59e385b7a723f61ac1f639/ex02/test_deferred_acceptance.jl)
を作業フォルダにダウンロードする．
前提として

* コードが書かれているファイルの名前は `deferred_acceptance.jl`
* 関数の名前は `deferred_acceptance`

になっているので，それと異なる命名をしている人は3, 4行目の

```jl
const file_name = "deferred_acceptance.jl"
const function_name = "deferred_acceptance"
```

の `deferred_acceptance.jl` と `deferred_acceptance` を自分の命名に合わせて書きかえる．

実行するには，Julia ウィンドウあるいは notebook で `include("test_deferred_acceptance.jl")` とする．実行結果が

```
Test Summary:                  | Pass  Total
Testing deferred_acceptance.jl |    4      4
```

のようになれば，テストが通ったということ．


## ゼミ生の成果物

* [Notebook リスト](notebooks.md)
