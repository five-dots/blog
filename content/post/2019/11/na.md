+++
title = "R のモダンな NA 処理まとめ"
date = 2019-11-06
tags = ["r"]
categories = ["programming"]
draft = false
toc = true
+++

データの欠損値を表す `NA` 。その `NA` をモダンなパッケージを用いて処理する方法についてまとめる。特に `vector` と `data.frame` に対して `NA` の削除や置換方法を中心に記載していきたい。

※ここで「モダン」と言っているのは、特に明確な定義があるわけではなく、最近開発されたパッケージという程度の意味である。


## 更新履歴 {#更新履歴}

-   2020/5/18 文書の体裁を修正。
-   2020/5/3 `{rlang}` の `%|%` 演算子を追加。
-   2020/5/3 文書の体裁を修正。


## 方針 {#方針}

この記事では `{dplyr}` や `{tidyr}` などのパッケージを積極的に使って `NA` 処理をする方法を紹介する方針だ。もちろん `{base}` の機能でも基本的な `NA` 処理は可能だ。

例えば `vector` から `NA` を削除する場合には、

```R
x <- c(1, 2, 3, NA, 5)
x
x[!is.na(x)]
```

```text

[1]  1  2  3 NA  5

[1] 1 2 3 5
```

`NA` を特定の値、例えば 0 に設定したい場合には、

```R
x[is.na(x)] <- 0
x
```

```text

[1] 1 2 3 0 5
```

と書くのが一般的だろう。添字に `logical vector` やインデックスを渡すこのやり方は、いかにも R っぽいコードであるし、汎用的でもある。

ただ、個人的にはこの記述法は好ましくない考えている。プログラミングは **「どのようにやるか (How)」ではなく、「何をやるか (What)」** という視点で書くべきだからだ。このくらいシンプルな例であれば問題ないかもしれないが、インデックスに複雑な計算が入っていたり `for` がネストされていて `i, j, k...` などと登場してくるととても読む気がなくなってしまうし、まさに How にフォーカスした書き方と言えるだろう。

後者の例であれば `{tidyr}` を使って、こう書いた方が **「何をしたいか (What)」** が明確でよりわかりやすい。

```R
tidyr::replace_na(x, 0)
```

```text
[1] 1 2 3 0 5
```

いつでも使いたいパッケージを使うことができる、という環境にない場合もあるだろうし、何だかんだで `{base}` での書き方も押さえておく必要がある。どちらも学ぶ必要があって大変、というのは 「R あるある」かもしれないが、将来コードを見え返す自分のためにも、少しでもわかりやすいコードを心がけるのは有益だと思う。


## 紹介する関数まとめ {#紹介する関数まとめ}

代表的な `NA` 処理毎に `{base}` の機能のみを使った一般的な書き方と、今回紹介する関数を `vector`, `data.frame` 毎に一覧にまとめるとこのようになる。「 `NA` **を** 置換」は `NA` を `0` に置換する場合、「 `NA` **に** 置換」は `1` を `NA` に置換すると想定した場合の例である。


### vector {#vector}

|        | 一般的なコード例       | 紹介するコード例                                |
|--------|----------------|-----------------------------------------|
| NA を削除 | `x[!is.na(x)]`         | `na.omit(x)`                                    |
| NA の有無 | `stopifnot(!anyNA(x))` | `assert_that(noNA(x))`                          |
| NA を置換 | `x[is.na(x)] <- 0`     | `replace_na(x, 0)`, `coalesce(x, 0)`, `x %¦% 0` |
| NA に置換 | `x[x==1] <- NA`        | `na_if(x, 1)`                                   |

※ `%¦%` は `%|%` に読み替えていただきたい。(Org-mode の Table レイアウトが崩れてしまうため)


### data.frame {#data-dot-frame}

|        | 一般的なコード例           | 紹介するコード例                   |
|--------|--------------------|----------------------------|
| NA を削除 | `df[complete.cases(df), ]` | `drop_na(df, everything())`        |
| NA の有無 | `stopifnot(!anyNA(df))`    | `assert(df, not_na, everything())` |
| NA を置換 | `df[is.na(df)] <- 0`       | `replace_na(df, list(y  0))`       |
| NA に置換 | `df[df==1] <- NA`          | `mutate(df, na_if(x, 1))`          |

`NA` を削除は `NA` を含む行をまるごと削除する例である。

また `data.frame` 向けの特殊な例として `tidyr::fill()` と `recipes::step_meanimpute()` などの `step_*impute()` の関数も一部紹介する。


## ライブラリの読み込み {#ライブラリの読み込み}

まずは、利用するパッケージの読み込みからスタート。

```R
library(dplyr)
library(tidyr)
library(rlang)
library(recipes)
library(assertr)
library(assertthat)
```


## NA には型がある {#na-には型がある}

具体的な内容に入る前に `NA` の型について確認しておきたい。自分自身、よく理解せずに過去にハマった経験があるからだ。

`NA` には型があるのだが、単に `NA` とした場合には `logical` 型である。例えば `character` 型の `NA` が欲しい場合には `NA_character_` とする必要がある。この辺りのことは `?NA` を見るか、日本語では [こちら](https://qiita.com/fujit33/items/5950889b983f93250998) の記事が詳しい。

`raw` 型を除く 5 つの `vector types` でこれらの型付きの `NA` が用意されている。

```R
purrr::map_lgl(c(NA, NA_integer_, NA_real_, NA_character_, NA_complex_), is.na)
```

```text
[1] TRUE TRUE TRUE TRUE TRUE
```

当然、こうしたチェックはすべて `TRUE` になる。

```R
is.logical(NA)
is.numeric(NA_real_)
is.integer(NA_integer_)
is.character(NA_character_)
is.complex(NA_complex_)
```

```text
[1] TRUE

[1] TRUE

[1] TRUE

[1] TRUE

[1] TRUE
```

この「 `NA` の型」が問題になる例として、 `dplyr::if_else()` や `dplyr::case_when()` など **関数の返り値の型が同じかどうかを厳密にチェックするタイプの関数** を利用する場合がある。

例えば、この例は `base::ifelse()` では意図した通りの結果になるが `dplyr::if_else()` ではエラーになる。

`base::ifelse()` の場合、

```R
x <- c(3, 2, 1, 0, -1, -2, -3)
ifelse(x > 0, "positive", NA)
```

```text

[1] "positive" "positive" "positive" NA         NA         NA         NA
```

`dplyr::if_else()` の場合、

```R
dplyr::if_else(x > 0, "positive", NA)
```

```text
Error: `false` must be a character vector, not a logical vector
Run `rlang::last_error()` to see where the error occurred.
```

これは `dplyr::if_else()` が `TRUE/FALSE` の評価結果として、同じ型であることを求めるからだ。この場合には、 `NA_character_` を使って明示的に `character` 型の欠損値であることを示す必要がある。

```R
dplyr::if_else(x > 0, "positive", NA_character_)
```

```text
[1] "positive" "positive" "positive" NA         NA         NA         NA
```

自らがコードの中で `NA` を設定する場合には、必ず型を明示したほうがより安全になるだろう。(そのお陰で `base::ifelse()` よりも `dplyr::if_else()` のほうが若干高速らしい)


## 利用するデータ {#利用するデータ}

ここからは `vector`, `data.frame` ともにできるだけシンプルなデータをつかって、具体的な `NA` 処理を見ていく。


### vector {#vector}

```R
x <- c(1, 2, 3, NA, 5)
x
```

```text

[1]  1  2  3 NA  5
```


### data.frame {#data-dot-frame}

```R
df <- data.frame(
  x = c(1, 2, 3),
  y = c(1, NA, 3),
  z = c(1, NA, NA)
)
df
```

| x | y   | z   |
|---|-----|-----|
| 1 | 1   | 1   |
| 2 | nil | nil |
| 3 | 3   | nil |

※この記事は、emacs の org-mode を使って執筆しているが、org-mode では `NA` が `nil` と記載されてしまうので、適宜読み替えていただきたい。


## NA を削除する {#na-を削除する}


### vector {#vector}

-   `stats::na.omit(object, ...)` を使う
    -   モダンなパッケージと言っておきながら `{stats}` からの関数だが、十分にシンプルかつ明確
    -   取り除かれたインデックスを `attribute` として保持してくれる

<!--listend-->

```R
x <- c(1, 2, 3, NA, 5)
na.omit(x)
```

```text

  x  y  z
1 1  1  1
2 2 NA NA
3 3  3 NA

[1] 1 2 3 5
attr(,"na.action")
[1] 4
attr(,"class")
[1] "omit"
```


### data.frame {#data-dot-frame}

-   `tidyr::drop_na(data, ...)` を使う
    -   特定の列の `NA` を省いた `data.frame` を返してくれる
    -   列選択には `dplyr::select()` 同様の方法が利用できる

<!--listend-->

```R
df %>%
  drop_na(y) # y 列の NA を含む行を削除
```

| x | y | z   |
|---|---|-----|
| 1 | 1 | 1   |
| 3 | 3 | nil |

-   全ての列から `NA` を含む行を削除したい場合は `tidyselect::everything()` を使う
    -   `filter(df, complete.cases(df))` と同じだが、個人的にはより意図が明確になると思う

<!--listend-->

```R
df %>%
  drop_na(everything())
```

| x | y | z |
|---|---|---|
| 1 | 1 | 1 |


## NA の有無を確認する {#na-の有無を確認する}

`NA` が (ひとつでも) 含まれていないか確認したいケースというのは `NA` が含まれていた場合を不正として扱いたい場合が多いだろう。そうした観点で、ここでは関数の入力値のチェックや、一連のデータ処理の間でアサーションを行う場合の例を紹介する。


### vector {#vector}

-   `assertthat::noNA(x)` を使う
    -   [ `{assertthat}` ](https://github.com/hadley/assertthat)は `base::stopifnot()` よりもエラー時により直感的なわかりやすいメッセージを出してくれる
    -   `noNA()` は、ひとつでも `NA` が含まれていた場合 `FALSE` を返す

<!--listend-->

```R
x <- c(1, 2, NA, 4)
assert_that(noNA(x))
```

```text

  x y  z
1 1 1  1
3 3 3 NA

  x y z
1 1 1 1

Error: x contains 1 missing values
```

-   `{base}` のみだと以下のように書くことができるが `{assertthat}` の方がエラーが明確でわかりやすい。

<!--listend-->

```R
stopifnot(!anyNA(x))
```

```text
Error: !anyNA(x) is not TRUE
```


### data.frame {#data-dot-frame}

-   `assertr::assert()` と `assertr::not_na()` を組み合わせる
    -   [ `{assertr}` ](https://github.com/ropensci/assertr) は `data.frame` をパイプ内でアサーションするためのパッケージ
    -   エラーの場合に、違反箇所を明示してくれる

<!--listend-->

```R
df %>%
 # dplyr 等のなんらかの処理 %>%
 assert(not_na, y) # 結果が意図通りかを確認するためのアサーションをパイプで挟む
```

```text

Column 'y' violates assertion 'not_na' 1 time
    verb redux_fn predicate column index value
1 assert       NA    not_na      y     2    NA

Error: assertr stopped execution
```

-   列選択には `{tidyselect}` の関数が利用できるので、全ての列に対して NA チェックをしたい場合は `everything()` とすれば良い

<!--listend-->

```R
df %>% assert(not_na, everything())
```

```text
Column 'y' violates assertion 'not_na' 1 time
    verb redux_fn predicate column index value
1 assert       NA    not_na      y     2    NA

Column 'z' violates assertion 'not_na' 2 times
    verb redux_fn predicate column index value
1 assert       NA    not_na      z     2    NA
2 assert       NA    not_na      z     3    NA

Error: assertr stopped execution
```


## NA を置換する {#na-を置換する}


### vector {#vector}

-   `tidyr::replace_na(data, replace)` を使う

<!--listend-->

```R
replace_na(x, 0)
```

```text
[1] 1 2 0 4
```

-   置換後の値が 1 つでない場合、 `dplyr::coalesce(...)` を使う
    -   複数のベクトルから、最初の `NA` でない値を返してくれる
    -   複数のベクトルの指定した順に `NA` でない値で合体してくれるイメージ
    -   全ての引数は、長さ 1 もしくは、第 1 引数と同じ長さである必要がある

<!--listend-->

```R
y <- c(1, 2, 3, 4)
coalesce(x, y)
```

```text

[1] 1 2 3 4
```

-   `{rlang}` の `%|%` を使う
    -   他の言語でいう NULL 合体演算子のようなイメージで、コードが簡潔になる
    -   左辺の `NA` を右辺の値で置き換えてくれる

<!--listend-->

```R
x %|% 0
```

```text
[1] 1 2 0 4
```


### data.frame {#data-dot-frame}

-   `data.frame` の場合も `tidyr::replace_na()` を使う
    -   ただし、置換後の値を列ごとに `list` で指定する

<!--listend-->

```R
replace_na(df, replace = list(y = 0, z = 2))
```

| x | y | z |
|---|---|---|
| 1 | 1 | 1 |
| 2 | 0 | 2 |
| 3 | 3 | 2 |

-   直前の `NA` でない値で置換したい場合 `tidyr::fill()` を使う
    -   時系列データの `NA` 置換でよく利用する (当日が `NA` なら前日の値で埋める等)
    -   `.direction = "down"/"up"` で下方向に置換するか、上方向に置換するかを選ぶことができる

<!--listend-->

```R
fill(df, y, .direction = "down")
```

| x | y | z   |
|---|---|-----|
| 1 | 1 | 1   |
| 2 | 1 | nil |
| 3 | 3 | nil |

-   特定の値ではなく、より柔軟に `NA` を置換したい場合は [ `{recipes}` ](https://github.com/tidymodels/recipes)パッケージの `step_*impute()` 関数群を使う
    -   例えば、平均値で置換したい場合は `step_meanimpute()`
    -   `{recipes}` や `{tidymodels}` パッケージ群の使い方は、[こちら](https://dropout009.hatenablog.com/entry/2019/01/06/124932)の記事がわかりやすい

<!--listend-->

```R
df %>%
  recipe() %>%
  step_meanimpute(y, z) %>% # step_*() で前処理をパイプで繋いでいく
  prep() %>%                # 実際に前処理を実行
  juice()                   # 前処理結果を data.frame として取り出す
```

| x | y | z |
|---|---|---|
| 1 | 1 | 1 |
| 2 | 2 | 1 |
| 3 | 3 | 1 |

-   `step_*impute()` 系は現状 7 つの関数が用意されている
    -   機能は名前からなんとなく想像はできると思うが、詳細はマニュアル参照

<!--listend-->

```R
pacman::p_funs(recipes) %>%
  stringr::str_subset("^step_.*impute$")
```

```text

  x y z
1 1 1 1
2 2 0 2
3 3 3 2

  x y  z
1 1 1  1
2 2 1 NA
3 3 3 NA

# A tibble: 3 x 3
      x     y     z
  <
<
<dbl>
1     1     1     1
2     2     2     1
3     3     3     1

[1] "step_bagimpute"    "step_knnimpute"    "step_lowerimpute"
[4] "step_meanimpute"   "step_medianimpute" "step_modeimpute"
[7] "step_rollimpute"
```


## NA に置換する {#na-に置換する}


### vector {#vector}

-   `dplyr::na_if(x, y)` を使う
    -   特定の値を `NA` に置き換える
    -   不正な値を `NA` にして、除外する際に使う
    -   `x`: 対象となるベクトル
    -   `y`: `NA` に置換するベクトル

<!--listend-->

```R
na_if(x, 1)
```

```text
[1] NA  2 NA  4
```


### data.frame {#data-dot-frame}

-   `data.frame` の場合も `dplyr::na_if(x, y)` を `mutate()` 内で使う

<!--listend-->

```R
df %>%
  mutate(b = na_if(y, 1))
```

| x | y   | z   | b   |
|---|-----|-----|-----|
| 1 | 1   | 1   | nil |
| 2 | nil | nil | nil |
| 3 | 3   | nil | 3   |

-   複数列に適応したい場合は `mutate_at()` + `{tidyselect}` を使う

<!--listend-->

```R
df %>%
  mutate_at(vars(everything()), na_if, y = 1) # ここでの y は、na_if() の引数名
```

| x   | y   | z   |
|-----|-----|-----|
| nil | nil | nil |
| 2   | nil | nil |
| 3   | 3   | nil |
