+++
title = "R の知っておくと便利な Infix Operator 達"
date = 2020-05-11
tags = ["r"]
categories = ["programming"]
draft = false
toc = true
+++

[GitHub](https://github.com/five-dots/notes/blob/master/lang/r/general/infix%5Foperator/infix%5Foperator.org) | [Blog](https://objective-boyd-9b8f29.netlify.app/2020/05/infix%5Foperator/) | [Qiita](https://qiita.com/five-dots/items/616c5f07d7a68ec70f62)

みんな大好き `%>%` でおなじみの **Infix Operator**. 調べてみると `{magrittr}` 以外にも、この形式の関数を収録しているパッケージは多数存在する。その中でも、知っておくと便利そう、というものを紹介していきたい。
<br />


## Infix Operator とは？ {#infix-operator-とは}

「神」こと Hadley Wichham の Advanced R [6.8 Function forms](https://adv-r.hadley.nz/functions.html#function-forms) によると、R の関数呼び出しの方法には、全部で 4 つの形態がある。

1.  **Prefix form**: 関数名の後に引数が続く、最も一般的な方法. `hoge(a, b)` の形態。
2.  **Infix form**: 関数名が 2 つの引数の間に置かれる呼び出し方法. `a %hoge% b` の形態。
3.  **Replacement form**: 値を変更するための関数. `hoge(x) <- c(a, b)` の形態。
4.  **Special form**: `for`, `while` や `[`, `[[` のような特別な形態。

2 つ目の形態が今回のトピックだ。2つの値の位置を明確にするために `LHS(左辺) %hoge% RHS(右辺)` という書き方がされることもある。

Infix とは、あまり聞き慣れない言葉だが、Prefix (接頭)・ Suffix (接尾) という単語の仲間で「接中」という意味だと考えれば、理解しやすいのではないかと思う。

自身で定義する場合も簡単で、この 3 つの条件を守ればよい。

1.  引数は 2 つ
2.  関数名は `%` ではじまり `%` で終わる
3.  R の関数命名規則から外れるので、さらに `` ` `` で囲む

<!--listend-->

```R
`%add%` <- function(lhs, rhs) {
  lhs + rhs
}

1 %add% 2
```

```R
[1] 3
```

前述の Advanced R の記事でも述べられている通り、R の全ての関数呼び出しは、Prefix form で書き直すことが可能なため **Infix form でないと書くことができない処理は存在しない**. それでも、2 つの値を比較したりする処理は、Infix Operator を使って書くと、コードがより簡潔に記述できる、という点がメリットではないかと思う。
<br />


## 紹介する演算子まとめ {#紹介する演算子まとめ}

今回紹介する関数を一覧にまとめるとこのようになる。(`{base}` に含まれるものは除外している。) 今後も有用なものが見つかり次第、更新していきたい。

| Operator       | Package       | Description               | Example                               |
|----------------|---------------|---------------------------|---------------------------------------|
| `%>%`          | `{magrittr}`  | 左辺を右辺の第 1 引数へ渡す | `mtcars %>% head()`                   |
| `%<>%`         | `{magrittr}`  | 右辺の結果を左辺に代入する | `mtcars$mpg %<>% log()`               |
| `%T>%`         | `{magrittr}`  | 右辺ではなく、左辺の結果を次に渡す | `mtcars$mpg %T>% plot() %>% mean()`   |
| `%$%`          | `{magrittr}`  | 左辺のオブジェクトに名前でアクセスする | `mtcars %$% cor(mpg, disp)`           |
| `%<-%`         | `{zeallot}`   | `vector` や `list` を分解して代入 | `c(x, y) %<-% c(0, 1)`                |
| `%@%`          | `{rlang}`     | 属性を抽出                | `mtcars %@% class`                    |
| `%¦¦%` (\*)    | `{rlang}`     | `NULL` のデフォルト値     | `NULL %¦¦% "default"`                 |
| `%¦%` (\*)     | `{rlang}`     | `NA` のデフォルト値       | `c("hoge", NA, "fuga") %¦% "default"` |
| `%--%`         | `{lubridate}` | 時間の引き算              | `arrival %--% leave`                  |
| `%m-%`, `%m+%` | `{lubridate}` | 月末日の違いやうるう年を考慮して月を加減 | `ymd("2020-01-31") %m+% months(1)`    |
| `%within%`     | `{lubridate}` | 日付・日時が `Interval` に含まれるか | `today() %winthn% interval`           |
| `%<d-%`        | `{pryr}`      | 遅延評価される変数を作成  | 下記参照                              |
| `%<a-%`        | `{pryr}`      | 活性束縛を作成            | `x %<a-% runif(1)`                    |

※ `¦` は `|` に読み替えていただきたい。(org-mode の Table レイアウトがくずれてしまうため)
<br />


## ライブラリの読み込み {#ライブラリの読み込み}

この記事で利用するパッケージを読み込む。

```R
library(tidyverse)
library(magrittr)
library(zeallot)
library(rlang)
library(lubridate)
library(pryr)
```

<br />


## 個別の紹介 {#個別の紹介}


### `%>%` パイプ演算子 {#パイプ演算子}

-   おなじみのパイプ演算子
-   左辺を右辺の第 1 引数として渡す (`.` を利用すれば、第 1 引数以外にも渡すことが可能)

<!--listend-->

```R
mtcars %>% head(2)
```

```R
              mpg cyl disp  hp drat    wt  qsec vs am gear carb
Mazda RX4      21   6  160 110  3.9 2.620 16.46  0  1    4    4
Mazda RX4 Wag  21   6  160 110  3.9 2.875 17.02  0  1    4    4
```

<br />


### `%<>%` 代入演算子 {#代入演算子}

-   右辺の処理結果を元の左辺のオブジェクトに代入する
    -   パイプの先頭で利用する

例えば、以下のように、処理の結果を同じ変数名で保持したいケースがある。

```R
mtcars <- mtcars %>%
  mutate(mpg = log(mpg))
```

<br />

代入演算子を使って、以下のように簡潔に書き換えることができる。

```R
mtcars$mpg %<>% log()
```

<br />


### `%T>%` Tee 演算子 {#t-tee-演算子}

-   右辺ではなく、左辺の結果をそのままスルーする
-   返り値が無い、副作用を目的とした処理を挟んでも、処理を止めないために利用する
-   基本形: `(オブジェクト) %T>% (副作用を目的とした処理) %>% (本来の処理に戻る)`

<!--listend-->

```R
mtcars$mpg %T>% # 次の plot() は返り値がないため、%T>% を使ってスルーさせる
  plot() %>%
  mean()
```

```R
[1] 0.0719204
```

<br />


### `%$%` Exposition 演算子 {#exposition-演算子}

-   左辺のオブジェクトの名前を右辺で参照できる
-   data 引数を持たない関数に名前を渡すのに便利

<!--listend-->

```R
mtcars %$% cor(mpg, disp)
```

```R
[1] -0.8475514
```

<br />


### `%<-%` 演算子 {#演算子}

-   `vector` や `list` を分解して代入してくれる
    -   Python のアンパックに相当する機能を提供
    -   `data.frame` であれば、列単位に分解してくれる

<!--listend-->

```R
c(x, y) %<-% c(0, 1)
x
y
```

```R
[1] 0
[1] 1
```

<br />

-   「以降全て」を `...rest` で表現できる

<!--listend-->

```R
c(first, ...rest) %<-% list("a", "b", "c", "d")
rest
```

```R
[[1]]
[1] "b"

[[2]]
[1] "c"

[[3]]
[1] "d"
```

<br />


### `%@%` 演算子 {#演算子}

-   左辺の属性を抽出できる

<!--listend-->

```R
# attr(mtcars, "class") と同じ
mtcars %@% class
```

```R
[1] "data.frame"
```

<br />


### `%||%` 演算子 {#演算子}

-   左辺が NULL の場合、右辺に指定した値を返す
    -   他の言語での NULL 合体演算子に相当

<!--listend-->

```R
1 %||% "default"
NULL %||% "default"
```

```R
[1] 1
[1] "default"
```

<br />


### `%|%` 演算子 {#演算子}

-   `%||%` の `NA` 版
    -   右辺で設定したデフォルト値で `NA` を置き換えてくれる

<!--listend-->

```R
c("hoge", NA_character_, "fuga") %|% "default"
```

```R
[1] "hoge"    "default" "fuga"
```

<br />


### `%--%`  演算子 {#演算子}

-   左辺から右辺を引いた時間を lubridate の `Interval` class で返す

<!--listend-->

```R
arrival <- ymd_hms("2011-06-04 12:00:00", tz = "Asia/Tokyo")
leave <- ymd_hms("2011-08-20 14:00:00", tz = "Asia/Tokyo")
arrival %--% leave
```

```R
[1] 2011-06-04 12:00:00 JST--2011-08-20 14:00:00 JST
```

<br />


### `%m-%`, `%m+%` 演算子 {#m-m-plus-演算子}

-   月を安全に加算・減算する
-   月末日やうるう年を考慮

通常、以下の例だと、2/31, 4/31 は存在しないので `NA` になってしまう。

```R
jan <- ymd("2020-01-31")
jan + months(1:3)
```

```R
[1] NA           "2020-03-31" NA
```

<br />

-   `%m+%`, `%m-%` であれば、月末日のズレを考慮して加算・減算してくれる

<!--listend-->

```R
jan %m+% months(1:3)
```

```R
[1] "2020-02-29" "2020-03-31" "2020-04-30"
```

<br />

-   うるう年も考慮してくれる

<!--listend-->

```R
leap <- ymd("2020-02-29")
leap %m+% years(1)
leap %m-% years(1)
```

```R
[1] "2021-02-28"
[1] "2019-02-28"
```

<br />


### `%within%` 演算子 {#within-演算子}

-   日付/日時が `Interval` に含まれているかどうか

<!--listend-->

```R
int1 <- interval(ymd("2001-01-01"), ymd("2002-01-01"))
int2 <- interval(ymd("2001-06-01"), ymd("2002-01-01"))

ymd("2001-05-03") %within% int1
int2 %within% int1
ymd("1999-01-01") %within% int1
```

```R
[1] TRUE
[1] TRUE
[1] FALSE
```

<br />

```R
ttime <- ymd_hms("2019-03-31 12:31:12")
rth <- interval(make_datetime(year(ttime), month(ttime), day(ttime), 9, 30, 0),
                make_datetime(year(ttime), month(ttime), day(ttime), 16, 0, 0))
ttime %within% rth
```

```R
[1] TRUE
```

<br />


### `%<d-%` 演算子 {#d-演算子}

-   Delayed binding (遅延評価, `promise`) を作成する
-   `base::delayedAssign()` と同等

<!--listend-->

```R
system.time(b %<d-% {
Sys.sleep(1)
1
})
system.time(b) # ここを実行した時点で、%<d-% のブロックが実行される
```

```R

user  system elapsed
    0       0       0

user  system elapsed
0.000   0.000   1.002
```

<br />


### `%<a-%` 演算子 {#a-演算子}

-   Active binding (活性束縛) の変数を作成する。(アクセスされる毎に再計算される変数
-   `base::makeActiveBinding()` と同等

<!--listend-->

```R
x %<a-% runif(1)
x
x
```

```R
[1] 0.1833575
[1] 0.05578229
```

<br />


## セッション情報 {#セッション情報}

```R
sessionInfo()
```

```R
R version 3.6.3 (2020-02-29)
Platform: x86_64-pc-linux-gnu (64-bit)
Running under: Ubuntu 18.04.4 LTS

Matrix products: default
BLAS:   /usr/lib/x86_64-linux-gnu/blas/libblas.so.3.7.1
LAPACK: /usr/lib/x86_64-linux-gnu/lapack/liblapack.so.3.7.1

locale:
 [1] LC_CTYPE=en_US.UTF-8       LC_NUMERIC=C
 [3] LC_TIME=en_US.UTF-8        LC_COLLATE=C
 [5] LC_MONETARY=en_US.UTF-8    LC_MESSAGES=C
 [7] LC_PAPER=en_US.UTF-8       LC_NAME=C
 [9] LC_ADDRESS=C               LC_TELEPHONE=C
[11] LC_MEASUREMENT=en_US.UTF-8 LC_IDENTIFICATION=C

attached base packages:
[1] stats     graphics  grDevices utils     datasets  methods   base

other attached packages:
 [1] pryr_0.1.4      lubridate_1.7.8 rlang_0.4.6     zeallot_0.1.0
 [5] magrittr_1.5    forcats_0.5.0   stringr_1.4.0   dplyr_0.8.5
 [9] purrr_0.3.4     readr_1.3.1     tidyr_1.0.3     tibble_3.0.1
[13] ggplot2_3.3.0   tidyverse_1.3.0

loaded via a namespace (and not attached):
 [1] Rcpp_1.0.4.6     cellranger_1.1.0 pillar_1.4.4     compiler_3.6.3
 [5] dbplyr_1.4.3     tools_3.6.3      jsonlite_1.6.1   lifecycle_0.2.0
 [9] nlme_3.1-147     gtable_0.3.0     lattice_0.20-41  pkgconfig_2.0.3
[13] reprex_0.3.0     cli_2.0.2        rstudioapi_0.11  DBI_1.1.0
[17] haven_2.2.0      withr_2.2.0      xml2_1.3.2       httr_1.4.1
[21] fs_1.4.1         generics_0.0.2   vctrs_0.2.4      hms_0.5.3
[25] grid_3.6.3       tidyselect_1.0.0 glue_1.4.0       R6_2.4.1
[29] fansi_0.4.1      readxl_1.3.1     pacman_0.5.1     modelr_0.1.7
[33] codetools_0.2-16 backports_1.1.6  scales_1.1.0     ellipsis_0.3.0
[37] rvest_0.3.5      assertthat_0.2.1 colorspace_1.4-1 stringi_1.4.6
[41] munsell_0.5.0    broom_0.5.6      crayon_1.3.4
```
