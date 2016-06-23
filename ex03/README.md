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
