# 課題1 (2016/5/2)
線形補間 (linear interpolation) の関数を返すコードを書いてみる．

目標は[この節](http://quant-econ.net/jl/optgrowth.html#fitted-value-iteration)の図を描く[コード](https://github.com/QuantEcon/QuantEcon.applications/blob/master/optgrowth/linapprox.jl)の[この行](https://github.com/QuantEcon/QuantEcon.applications/blob/master/optgrowth/linapprox.jl#L7)を置き換えられるようなプログラムを書くこと．

* 提出方法：GitHub のリポジトリにファイルを「プッシュ」する．
* 中間締め切り：5月9日 (月)
* 最終締め切り：5月21日 (土)

与えられた1次元配列 `grid`, `vals` (ともに同じ長さ `n` とする) に対して，
`(grid[1], vals[1])`, ..., `(grid[n], vals[n])` を結ぶ折れ線グラフの関数を得たい．
(`grid` の要素は小さい順にソートされているとする．)

たとえば，以下のように「関数を返す関数」を書いてみる．

```julia
function my_lin_interp(grid, vals)
    function func(x)
        ...
    end

    return func
end
```

(ほんとはこれは筋が悪く，type で書いた方がよい．それはまた後で考える．)

最初は `grid` の要素が2つ (区間が1つ) だけのケースを想定して書いてみる．
たとえば：

```julia
grid = [1, 2]
vals = [2, 0]
f = my_lin_interp(grid, vals)

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

(`grid` の範囲外の値が `x` として入力されたときの動作は自分で適宜決める．)


## 拡張・改良

次は

* 引数 `x` が AbstractVector (1次元配列とそのたぐい) のときへの対応 (universal function 化)
* type としての実装

をやってみる．

### AbstractVector 入力への対応

[Types, Methods and Performance](http://quant-econ.net/jl/types_methods.html) の章の

* [User Defined Methods](http://quant-econ.net/jl/types_methods.html#user-defined-methods)
* [Exercise 2](http://quant-econ.net/jl/types_methods.html#exercise-2)

を参考にする．

```julia
julia> f([1.25, 1.5])
2-element Array{Float64,1}:
 1.5
 1.0
```

とか

```julia
julia> f(1:0.25:1.5)
3-element Array{Float64,1}:
 2.0
 1.5
 1.0
```

のように `x` が Vector や Range として与えられたら要素それぞれに対する `y` の値を Vector で返すようにしたい．

たとえば次のように書いてみる (`func` の中身は適当)：

```julia
function my_lin_interp(grid, vals)
    function func(x)
        return x * 2
    end

    function func(x::AbstractVector)
        return x + 10
    end

    return func
end

grid = [1, 2]
vals = [2, 0]
f = my_lin_interp(grid, vals)
```

引数 `x` が `AbstractVector` のときは `func(x::AbstractVector)` に書いてある命令が，それ以外のときは `func(x)` に書いてある命令が実行される：

```julia
julia> f(1)
2

julia> f([1, 2])
2-element Array{Int64,1}:
 11
 12
```

このままだと，`x` がたとえば文字列でも `func(x)` が実行される．
`Int64` や `Float64` だけ受けつけたいときは `Real` という
[abstract type](http://quant-econ.net/jl/types_methods.html#abstract-types)
の annotation をつけて，`func(x)` の代わりに `func(x::Real)` と書く．

同様に，`func(x::AbstractVector)` についても `Real` の subtype からなる `AbstractVector` だけ受けつけたいときは，`func{T<:Real}(x::AbstractVector{T})` と書く．

* [Parametric Composite Types](http://docs.julialang.org/en/release-0.4/manual/types/#parametric-composite-types)

スカラーのケースがちゃんと書けたら，ベクトルに対応するのは実は簡単で，

```julia
function my_lin_interp(grid, vals)
    function func(x::Real)
        ...
    end

    function func{T<:Real}(x::AbstractVector{T})
        n = length(x)
        out = Array(Float64, n)
        for i in 1:n
            out[i] = func(x[i])
        end
        return out
    end

    return func
end
```

のように書けばよい．
`func(x[i])` のところでは，`x[i]` がスカラー (`x[i]` のタイプが `Real` のサブタイプ) であることから
`func(x::Real)` が呼ばれている．

### Type として実装

[Immutable Composite Type](http://docs.julialang.org/en/release-0.4/manual/types/#immutable-composite-types)
を使ってみる．

```julia
immutable MyLinInterp
    grid
    vals
end
```

`MyLinInterp` タイプに対して `call` メソッドを定義することで，

```julia
grid = [1, 2]
vals = [2, 0]
f = MyLinInterp(grid, vals)

f(1.25)
```

のように使えるようになる．

`call` メソッドを定義するには以下のように書く：

```julia
function Base.call(f::MyLinInterp, x)
    ...
end
```

この場合も，`x` がスカラーのケースとベクトルのケースについて定義しておけば，それぞれ入力に対して動作を変えることができる．

* [Call overloading and function-like objects](http://docs.julialang.org/en/release-0.4/manual/methods/#call-overloading-and-function-like-objects)


## ファイル構成

最初は作業は Jupyter notebook で行えばよいですが，最終的には

1. 関数を定義する `.jl` ファイル (たとえば `lin_interp.jl`)
2. そのデモの notebook (たとえば `lin_interp_demo.ipynb`)

を作ってください．

`lin_interp_demo.ipynb` では

* タイトルと名前を書く ([ここ](http://quant-econ.net/jl/getting_started.html#other-content) も参照)
* 必要であればパッケージを読み込む
* `lin_interp.jl` の中身を表示する

    ```jl
  ;cat lin_interp.jl
  ```

* `lin_interp.jl` を読み込む

  ```jl
  include("lin_interp.jl")
  ```

* `lin_interp.jl` で定義した関数を使ってみる  
  このページにある簡単な例を実行したり，[この節](http://quant-econ.net/jl/optgrowth.html#fitted-value-iteration)の図を書いてみたり，いろいろデモを行ってみる

ようにしてください．

* [実行例](http://nbviewer.jupyter.org/github/OyamaZemi/exercises2016/blob/master/ex01/lin_interp_demo.ipynb)


## ゼミ生の成果物

* [Notebook リスト](notebooks.md)
