# 課題1 (2016/5/2)
線形補間 (linear interporation) の関数を返すコードを書いてみる．

目標は[この節](http://quant-econ.net/jl/optgrowth.html#fitted-value-iteration)の図を描く[コード](https://github.com/QuantEcon/QuantEcon.applications/blob/master/optgrowth/linapprox.jl)の[この行](https://github.com/QuantEcon/QuantEcon.applications/blob/master/optgrowth/linapprox.jl#L7)を置き換えられるようなプログラムを書くこと．

* 提出方法：GitHub のリポジトリにファイルを「プッシュ」する．
* 中間締め切り：5月9日 (月)
* 最終締め切り：5月21日 (土)

与えられた1次元配列 `g` ("grid"), `v` ("values") (ともに同じ長さ `n` とする) に対して，
`(g[1], v[1])`, ..., `(g[n], v[n])` を結ぶ折れ線グラフの関数を得たい．
(`g` の要素は小さい順にソートされているとする．)

たとえば，以下のように「関数を返す関数」を書いてみる．

```julia
function my_lin_interp(g, v)
    function func(x)
        ...
    end

    return func
end
```

(ほんとはこれは筋が悪く，type で書いた方がよい．それはまた後で考える．)

最初は `g` の要素が2つ (区間が1つ) だけのケースを想定して書いてみる．
たとえば：

```julia
g = [1, 2]
v = [2, 0]
f = my_lin_interp(g, v)

f(1.25)
# 1.5 が返ってくるはず
```

* [Arrays](http://quant-econ.net/jl/julia_by_example.html#arrays)
* [User-Defined Functions](http://quant-econ.net/jl/julia_by_example.html#user-defined-functions)

一般の n 要素のケースにおいては，`x` がどの区間に入るのかを調べないといけない．
それには
[`searchsortedfirst`](http://docs.julialang.org/en/release-0.4/stdlib/sort/#Base.searchsortedfirst)
あるいは
[`searchsortedlast`](http://docs.julialang.org/en/release-0.4/stdlib/sort/#Base.searchsortedlast)
を使う．
