+++
title = "[R] {purrr} の Adverbs 系の関数が便利そう"
date = 2020-05-13
tags = ["r"]
categories = ["programming"]
draft = false
toc = true
+++

`{purrr}` は個人的に、かなり利用頻度の高いパッケージのひとつで、apply 系の関数は使い方を忘れてしまうくらい map 系の関数をよく使っている。それで `{purrr}` は使いこなしているつもりになっていたが **Adverbs** という、これまで全く使ってこなかったカテゴリーの関数群の存在を知って、これがなかなかおもしろそうだったので、紹介してみたい。

といっても、一通りマニュアルにある例を流してみた、という程度のため、何か間違った記載があるかもしれない。今後、さらに要点をつかめてきたら再度更新したい。


## Adverbs とは？ {#adverbs-とは}

そもそも、Adverbs とは、「関数を引数にとり、関数を返す」関数のこと。元の関数にひと味くわえる・修飾するという意味で、Adverbs (副詞) という言葉が、使われているようだ。 (関数型言語の世界では、よく使われる用語なのだろうか、、よくわからない。)

関数をラップして、より便利に利用できる関数を定義する、というのはよくあることだが、頻出するケースを `{purrr}` 側で用意してくれている、と考えればよいと思う。
<br />


## Adverbs 一覧 {#adverbs-一覧}

最新の `{purrr}` `0.3.4` に収録されている関数を一覧にまとめると、このようになる。 `safely()` や `insistently()` 辺りを使うと、普段書いているような、エラー処理を含む関数をシンプルに書くことができて良さそうだ. `lift()` 系の関数や `compose()` などは、ぱっと見で利用シーンが浮かばないが、この辺りを使いこなすと **関数型を使いこなしていて、なんかすごそう** な感じがする。

| 関数名                | 返ってくる関数                                                |
|--------------------|--------------------------------------------------------|
| `safely()`            | エラー時に止まらずに `list(result, error)` を返す関数         |
| `possibly()`          | エラー時に止まらずに、設定したデフォルト値を返す関数          |
| `auto_browse()`       | エラー時に自動的に `browse()` する関数                        |
| `quietly()`           | メッセージを出力せずに、 `list(result, output, warnings, messages)` を返す関数 |
| `insistently()`       | エラー時に指定した条件(待ち時間・試行回数)でリトライする関数  |
| `slowly()`            | 連続実行の際に、指定時間待ってから試行する関数                |
| `lift()`, `lift_**()` | 引数の形式が変更された関数 (例: `vector` -> `list` など)      |
| `compose()`           | 複数の関数を結合した関数                                      |
| `partial()`           | 一部の引数を固定した関数                                      |
| `negate()`            | 否定形の関数                                                  |

<br />


## 個別の紹介 {#個別の紹介}


### `safely()` {#safely}

-   エラーを発生し得る関数をラップして `list(result, error)` を返す
    -   エラーがなければ `error = NULL`, エラーがあれば `result = NULL`
    -   条件 (データやパラメタ) によって、エラーを吐くアルゴリズムなどを安全に処理できる

<!--listend-->

```R
library(purrr)
f <- function() {
  stop("Error!!", call. = FALSE)
}

safe_f <- safely(f)
safe_f()
```

```R
$result
NULL

$error
<simpleError: Error!!>
```

<br />

`` zeallot::`%<-%` `` (Python のアンパックに相当) を利用するとより簡潔に以降の処理を書くことができる。

```R
library(zeallot)
c(res, err) %<-% safe_f()
res
err
```

```R
NULL

<simpleError: Error!!>
```

<br />


### `possibly()` {#possibly}

-   エラー時には、エラーではなく、事前に設定したデフォルト値 (`otherwise` 引数) を返す

<!--listend-->

```R
possibly_f <- possibly(f, otherwise = "default")
possibly_f()
```

```R
[1] "default"
```

<br />


### `auto_browse()` {#auto-browse}

-   エラー時に自動的に `browse()` を起動してデバッグを支援してくれる
    -   実際の出力はブログ記事では説明しにくいので、割愛

<!--listend-->

```R
auto_browse_f <- auto_browse(f)
auto_browse_f()
```

<br />


### `quietly()` {#quietly}

-   メッセージを出力せずに `list(result, output, warnings, messages)` で返す
    -   `output` が `cat()` や `print()` からの出力
    -   スクリプト実行で余計なメッセージを抑制したい場合に便利

<!--listend-->

```R
verbose_f <- function() {
  cat("output!\n")
  message("messages!")
  warning("warnings!")
  "result!"
}

quiet_f <- quietly(verbose_f)
quiet_f()
```

```R
$result
[1] "result!"

$output
[1] "output!"

$warnings
[1] "warnings!"

$messages
[1] "messages!\n"
```

<br />


### `insistently()` {#insistently}

-   エラーが発生した場合、成功するまでリトライしてくれる
    -   `rate` で待ち時間や、最大試行回数を設定する

<!--listend-->

```R
risky_runif <- function(lo = 0, hi = 1) {
  y <- runif(1, lo, hi)
  if (y < 0.5) stop(y, " is too small")
  y
}
rate <- rate_backoff(pause_base = 0.1, max_times = 10)
insist_runif <- insistently(risky_runif, rate = rate, quiet = FALSE)
insist_runif()
```

```R
Error: 0.402555017499253 is too small
Retrying in 1 seconds.
Error: 0.0129680023528636 is too small
Retrying in 1 seconds.
Error: 0.285843262448907 is too small
Retrying in 1 seconds.
[1] 0.5532546
```

<br />


### `slowly()` {#slowly}

-   連続実行する際に Delay を設定できる
    -   アクセス制限のある API を利用する場合に便利

<!--listend-->

```R
rate <- rate_delay(0.1)
slow_runif <- slowly(~ runif(1), rate = rate, quiet = FALSE)
map_dbl(1:5, slow_runif)
```

```R
Retrying in 0.1 seconds.
Retrying in 0.1 seconds.
Retrying in 0.1 seconds.
Retrying in 0.1 seconds.
[1] 0.5201321 0.0438438 0.3846940 0.3723179 0.9737134
```


### `lift()`, `lift_**()` {#lift-lift}

-   引数の形式が変換された関数を返す
    -   `lift_**()` の形式の亜種が 6 つ
    -   `vector` から `list` への変換であれば `list_vl()`
    -   `list` から `...` への変換であれば `list_ld().`

たとえば `mean()` は `vector` の引数 `x` を受け取るので `sum()` のように `...` を受け取ると勘違いすると予期せぬ結果になる。

```R
# mean(x, ...)
mean(1, 2, 3)
```

```R
[1] 1
```

`mean()` を `...` を受けるバージョンに変更する場合は `lift_vd()` を使えば良い。

```R
lifted_mean <- lift_vd(mean)
lifted_mean(1, 2, 3)
```

```R
[1] 2
```


### `compose()` {#compose}

-   複数の関数を結合した関数を作成する。
    -   デフォルトでは、左から右に結果を受け渡していく。
    -   パイプで複数の処理を繋げていくような処理をたくさん書いていたら、これにまとめるとスッキリするかもしれない。

<!--listend-->

```R
add1 <- function(x) x + 1
compose(add1, add1)(8)
```

```R
[1] 10
```


### `partial()` {#partial}

-   一部の引数を固定した関数を返す
    -   ラッパー関数を作成する動機としては、一番シンプルでよくありそう。

<!--listend-->

```R
my_mean <- partial(mean, na.rm = TRUE)
my_mean(c(1, NA_real_, 2, 3))
```

```R
[1] 2
```


### `negate()` {#negate}

-   Predicate (真偽を返す関数) の否定形を返す関数

<!--listend-->

```R
is.not.null <- negate(is.null)
is.not.null(NULL)
```

```R
[1] FALSE
```


## 参考 {#参考}

-   [Function reference • purrr](https://purrr.tidyverse.org/reference/index.html)
-   [Best practices for exporting adverb-wrapped functions](https://purrr.tidyverse.org/reference/faq-adverbs-export.html)
-   [purrr 0.1.0 を使ってみる(4) lift - Technically, technophobic.](https://notchained.hatenablog.com/entry/2015/10/04/223555)
