# 課題4 (2016/7/4)

Kandori-Mailath-Rob (KMR) の確率進化モデルのシミュレーションをしてみる．

とりあえず[昨年の課題](https://github.com/OyamaZemi/exercises2015/tree/master/ex04)を参照のこと．
「[KMR の確率進化動学の説明](http://nbviewer.jupyter.org/github/OyamaZemi/exercises2015/blob/master/ex04/KMR_notes.ipynb)」をよく読む．


## 対称 2x2 協調ゲームと *p*-支配

次のような協調ゲーム (coordination game) を考える：

```
      1      2
  ---------------
1 | 4, 4 | 0, 3 |
  ---------------
2 | 3, 0 | 2, 2 |
  ---------------
```

純粋戦略 Nash 均衡は (1, 1) と (2, 2) である．

このゲームにおいて，Nash 均衡 (2, 2) は 1/3-支配的 (1/3-dominant) であるという．
ここで，p (0 <= p <= 1) に対して「相手が行動 2 をとる確率が p より小さいならば自分の最適反応は 1，p より大きいならば自分の最適反応は 2」ということが成り立つとき，均衡 (2, 2) を p-支配的という．

今回の分析では，p の値が，与えられた対称 2x2 協調ゲームに関して必要な情報をすべて含んでいる．


## KMR 確率進化動学のマルコフ行列

まずは，与えられた `p` の値 (上で導入した p-支配の値)，プレイヤーの数 `N`，randomness のパラメタ `epsilon` に対して，マルコフ行列を返すような関数を書きましょう．
「同時手番」か「逐次手番」かどちらか決めて (あるいは両方に対して) 関数を書きます．

```jl
# 同時手番の場合
function kmr_markov_matrix_simultaneous(p::Float64, N::Int, epsilon::Float64)
    P = Array(Float64, N+1, N+1)  # 入れ物を定義する
    
    # 動学の定義にしたがって P の中身を埋めていく
    
    return P
end
```

```jl
# 逐次手番の場合
function kmr_markov_matrix_sequential(p::Float64, N::Int, epsilon::Float64)
    P = zeros(N+1, N+1)  # 0 がたくさんあるので `zeros` で入れ物を定義する
    
    # 動学の定義にしたがって P の中身を埋めていく
    
    return P
end
```

これで，たとえば `p = 1/3; N = 10; epsilon = 0.01` に対して `P = kmr_markov_matrix_simultaneous(p, N, epsilon)`
あるいは `P = kmr_markov_matrix_sequential(p, N, epsilon)` として，
(`using QuantEcon` とした上で) `mc = MarkovChain(P)` とすれば，`MarkovChain` インスタンスが作られるが，
state は行動 2 をプレイするプレイヤーの数 0, ..., N なので，`state_values` を `0:N` として
`mc = MarkovChain(P, 0:N)` とするとよい．

`mc` を使って simulation をしたり ([ここ](http://quant-econ.net/jl/finite_markov.html#using-quantecon-s-routines)参照)，定常分布を求めたり ([ここ](http://quant-econ.net/jl/finite_markov.html#calculating-stationary-distributions)参照) する (それらをグラフにプロットしてみる)．


## KMR 動学を表すタイプを作る

たとえば，次のような composite type で表してみましょう：

```jl
type KMR2x2
    p::Float64
    N::Int
    epsilon::Float64
    revision::Symbol  # simultaneous か sequential を指定する
    mc::MarkovChain
end
```

Constructor を作ります．
たとえば：

```jl
function KMR2x2(p::Float64, N::Int, epsilon::Float64;
                revision::Symbol=:simultaneous)  # デフォルトをたとえば simultaneous とする
    if revision == :simultaneous  # 片一方しか作っていない人は，作ってない方は消す
        P = kmr_markov_matrix_simultaneous(p, N, epsilon)
    elseif revision == :sequential
        P = kmr_markov_matrix_sequential(p, N, epsilon)
    else
        throw(ArgumentError)  # エラーを返す
    end
    
    mc = MarkovChain(P, 0:N)
    
    return KMR2x2(p, N, epsilon, revision, mc)  # KMR2x2 インスタンスを返す
end
```

あとは `KMR2x2` に対してメソッドを作ります．
