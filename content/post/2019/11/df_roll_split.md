+++
title = "R の 時系列 data.frame を {rsample} を使って 週・月単位で分割する"
date = 2019-11-07
tags = ["r"]
categories = ["programming"]
draft = false
toc = true
+++

[GitHub](https://github.com/five-dots/notes/blob/master/lang/r/general/df%5Froll%5Fsplit/df%5Froll%5Fsplit.org) | [Blog](https://objective-boyd-9b8f29.netlify.app/2019/11/df%5Froll%5Fsplit/) | [Qiita](https://qiita.com/five-dots/items/46fc5c9505b111106e1c)

R で時系列データを交差検証用に分割する際には[ `{rsample}` ](https://github.com/tidymodels/rsample)パッケージを利用すると便利だ。このパッケージの `rolling_origin()` 関数で、訓練データと検証データを時系列にスライドさせながら抽出することができる。

ただし、この関数では訓練・検証のデータ件数を `data.frame` の行数でしか指定できない。これでは、時系列データを、月・週といったカレンダーの単位で分割したいケースに対応しづらい。例えば過去 12 ヶ月のデータでモデルを作成し、その後 3 ヶ月で予測の精度を評価する、といったケースだ。

今回は `{tidyverse}` のパッケージ群と `{lubridate}` を `{rsample}` と組み合わせて、これを実現する方法を紹介する。なお `{rsample}` 全般の利用方法については、[この記事](https://blog.hoxo-m.com/entry/2019/06/08/220307)が詳しいので、一読をおすすめする。


## 更新履歴 {#更新履歴}

-   **[2020/05/07]** 記事の体裁を修正
-   **[2019/11/22]** 週単位のグループ化コードのバグ修正


## 時系列データの交差検証 {#時系列データの交差検証}

本題に入る前に、時系列データを交差検証する場合の分割方法について、簡単に確認してみよう。時系列データはその名前の通り、データの時間的な並びに意味があるデータであるため、単純にデータ全体をランダムに分割することができない。

具体的には、下図のように訓練データと検証データ (予測データ) を時間の経過に沿って、ずらしながら抽出する方法が取られる。さらに、訓練データを固定するか・拡張していくかで大きく 2 つの手法が存在していると考えればよいだろう。

1 訓練データ・検証データともに期間を固定する方法

{{< figure src="https://dl.dropboxusercontent.com/s/hyoffkb4cxjjqlq/roll%5Fsplits.png" >}}

2 訓練データの期間を伸ばしていく方法

{{< figure src="https://dl.dropboxusercontent.com/s/x7cvidzm7mg1ll3/roll%5Fsplits%5Fexpand.png" >}}

`rsampel::rolling_origin()` では、引数で `cumulative = FALSE` (デフォルト) とすれば、1 の方法、 `TRUE` にすれば 2 の方法で抽出することができる。


## ライブラリの読み込み {#ライブラリの読み込み}

それでは、具体的に R での実現方法に移っていこう。まずは、必要なライブラリの読み込みから。

```R
library(tidyverse)
library(lubridate)
library(rsample)
library(tidyquant)
```


## 利用するデータ {#利用するデータ}

データは `{tidyquant}` パッケージに収録されている `FANG` データセットを利用する。Facebook, Amazon, Netflix, Goolge 4 社の株価データだ。今回はこのデータを「 **週単位** 」で利用するという想定でやってみたい。

```R
data(FANG)
FANG2 <- FANG %>%
  ## 例示に必要な列のみ選択する
  ## adjusted = 調整済みの終値
  ## volume = 出来高
  select(symbol, date, adjusted, volume) %>%
  ## 週単位での抽出が理解しやすいように、曜日を列として追加する
  mutate(dayOfWeek = wday(date, label = TRUE))
head(FANG2)
```

| symbol | date       | adjusted  | volume    | dayOfWeek |
|--------|------------|-----------|-----------|-----------|
| FB     | 2013-01-02 | 28        | 69846400  | Wed       |
| FB     | 2013-01-03 | 27.77     | 63140600  | Thu       |
| FB     | 2013-01-04 | 28.76     | 72715400  | Fri       |
| FB     | 2013-01-07 | 29.42     | 83781800  | Mon       |
| FB     | 2013-01-08 | 29.059999 | 45871300  | Tue       |
| FB     | 2013-01-09 | 30.59     | 104787700 | Wed       |


## 週単位でネストされた data.frame を作成する {#週単位でネストされた-data-dot-frame-を作成する}

カレンダーの単位 (週・月や年など) で `rolling_origin()` を利用するため、事前に必要な単位で `data.frame` をネストし、そのネストされた `data.frame` に `rolling_origin()` を適用するというアプローチをとってみる。

```R
FANG_nested <- FANG2 %>%
  ## 年 + 週 でグループ化. 例えば 2019-11-07 であれば、2019 年の第 45 週にグループ化される
  group_by(year = year(date), week = isoweek(date)) %>%
  ## グループのキーとして、週末日を利用する
  mutate(weekend = max(date)) %>%
  ## グループ化を一旦解除
  ungroup() %>%
  ## グループ化のキーとしては、週末日を利用するので、year, week は不要
  select(-year, -week) %>%
  ## nested data.frame を作成
  group_nest(weekend)
FANG_nested
```

**[2019/11/22 追記]**
上記の `group_by(year = year(date), week = isoweek(date))` という書き方は、12 月最終週も week 1 に分類されてしまう可能性があり、意図しない結果を招いていた。素直に `lubridate::ceiling_date()` を利用するほうが、コードが直感的かつ、このようなバグも発生しないので修正した。 `unit` にまとめたい単位 (例 `"week"`, `"month"` など) を指定すれば良い。

```R
FANG_nested <- FANG2 %>%
  group_by(week = ceiling_date(date, unit = "week")) %>%
  ## グループのキーとして、週末日を利用する
  mutate(weekend = max(date)) %>%
  ## グループ化を一旦解除
  ungroup() %>%
  ## グループ化のキーとしては、週末日を利用するので、week は不要
  select(-week) %>%
  ## nested data.frame を作成
  group_nest(weekend)
FANG_nested
```

```R
# A tibble: 209 x 2
   weekend    data
   <date>     <list>
 1 2013-01-04 <tibble [12 × 5]>
 2 2013-01-11 <tibble [20 × 5]>
 3 2013-01-18 <tibble [20 × 5]>
 4 2013-01-25 <tibble [16 × 5]>
 5 2013-02-01 <tibble [20 × 5]>
 6 2013-02-08 <tibble [20 × 5]>
 7 2013-02-15 <tibble [20 × 5]>
 8 2013-02-22 <tibble [16 × 5]>
 9 2013-03-01 <tibble [20 × 5]>
10 2013-03-08 <tibble [20 × 5]>
# … with 199 more rows
```

これで、週単位でネストさせることができた。キーは、グループの最終日 (この例では週末日) に設定したが、この辺りは各自の好みで良いと思う。

念の為、ネストの中を見てみると、きちんと月曜から金曜までのデータが含まれていることが確認できる。

```R
FANG_nested$data[[2]]
## インデックス 1 は、水曜からのデータなので、わかりやすい 2 を表示した
```

| symbol | date       | adjusted   | volume    | dayOfWeek |
|--------|------------|------------|-----------|-----------|
| FB     | 2013-01-07 | 29.42      | 83781800  | Mon       |
| FB     | 2013-01-08 | 29.059999  | 45871300  | Tue       |
| FB     | 2013-01-09 | 30.59      | 104787700 | Wed       |
| FB     | 2013-01-10 | 31.299999  | 95316400  | Thu       |
| FB     | 2013-01-11 | 31.719999  | 89598000  | Fri       |
| AMZN   | 2013-01-07 | 268.459991 | 4910000   | Mon       |
| AMZN   | 2013-01-08 | 266.380005 | 3010700   | Tue       |
| AMZN   | 2013-01-09 | 266.350006 | 2265600   | Wed       |
| AMZN   | 2013-01-10 | 265.339996 | 2863400   | Thu       |
| AMZN   | 2013-01-11 | 267.940002 | 2413300   | Fri       |
| NFLX   | 2013-01-07 | 14.171429  | 45550400  | Mon       |
| NFLX   | 2013-01-08 | 13.88      | 24714900  | Tue       |
| NFLX   | 2013-01-09 | 13.701428  | 20223000  | Wed       |
| NFLX   | 2013-01-10 | 14         | 26117700  | Thu       |
| NFLX   | 2013-01-11 | 14.47      | 29851500  | Fri       |
| GOOG   | 2013-01-07 | 367.008634 | 3323800   | Mon       |
| GOOG   | 2013-01-08 | 366.284329 | 3364700   | Tue       |
| GOOG   | 2013-01-09 | 368.691926 | 4064500   | Wed       |
| GOOG   | 2013-01-10 | 370.370261 | 3685000   | Thu       |
| GOOG   | 2013-01-11 | 369.626004 | 2579900   | Fri       |


## 交差検証用のデータを抽出 {#交差検証用のデータを抽出}

それでは、交差検証用に `rolling_origin()` を適応してみよう。今回は訓練データとして 52 週 (1 年)、検証データとして 13 週 (3 ヶ月) という想定でやってみる。個人的には、元データの行数で考えるよりも、より直感的に指定できるようになったと思う。

```R
FANG_rolled <- rolling_origin(FANG_nested, initial = 52, assess = 13, cumulative = FALSE)
FANG_rolled
```

```R
# Rolling origin forecast resampling
# A tibble: 145 x 2
   splits          id
   <list>          <chr>
 1 <split [52/13]> Slice001
 2 <split [52/13]> Slice002
 3 <split [52/13]> Slice003
 4 <split [52/13]> Slice004
 5 <split [52/13]> Slice005
 6 <split [52/13]> Slice006
 7 <split [52/13]> Slice007
 8 <split [52/13]> Slice008
 9 <split [52/13]> Slice009
10 <split [52/13]> Slice010
# … with 135 more rows
```

実際に、訓練データ・検証データを取り出すには、通常通り `analysis()`, `assessment()` で OK だ。

```R
FANG_analysis1 <- analysis(FANG_rolled$splits[[1]])
FANG_analysis1
```

```R
# A tibble: 52 x 2
   weekend    data
   <date>     <list>
 1 2013-01-04 <tibble [12 × 5]>
 2 2013-01-11 <tibble [20 × 5]>
 3 2013-01-18 <tibble [20 × 5]>
 4 2013-01-25 <tibble [16 × 5]>
 5 2013-02-01 <tibble [20 × 5]>
 6 2013-02-08 <tibble [20 × 5]>
 7 2013-02-15 <tibble [20 × 5]>
 8 2013-02-22 <tibble [16 × 5]>
 9 2013-03-01 <tibble [20 × 5]>
10 2013-03-08 <tibble [20 × 5]>
# … with 42 more rows
```

取り出したデータは、週単位でネストされてしまっているので、分析に利用するためには `dplyr::bind_rows()` でフラットな `data.frame` に再変換する。 `bind_rows()` は **list of data.frame** をそのまま受け取ることができるので、このケースでは非常に使い勝手が良い。

```R
bind_rows(FANG_analysis1$data) %>% head(n = 20)
```

| symbol | date       | adjusted   | volume    | dayOfWeek |
|--------|------------|------------|-----------|-----------|
| FB     | 2013-01-02 | 28         | 69846400  | Wed       |
| FB     | 2013-01-03 | 27.77      | 63140600  | Thu       |
| FB     | 2013-01-04 | 28.76      | 72715400  | Fri       |
| AMZN   | 2013-01-02 | 257.309998 | 3271000   | Wed       |
| AMZN   | 2013-01-03 | 258.480011 | 2750900   | Thu       |
| AMZN   | 2013-01-04 | 259.149994 | 1874200   | Fri       |
| NFLX   | 2013-01-02 | 13.144286  | 19431300  | Wed       |
| NFLX   | 2013-01-03 | 13.798572  | 27912500  | Thu       |
| NFLX   | 2013-01-04 | 13.711429  | 17761100  | Fri       |
| GOOG   | 2013-01-02 | 361.264351 | 5101500   | Wed       |
| GOOG   | 2013-01-03 | 361.474154 | 4653700   | Thu       |
| GOOG   | 2013-01-04 | 368.617014 | 5547600   | Fri       |
| FB     | 2013-01-07 | 29.42      | 83781800  | Mon       |
| FB     | 2013-01-08 | 29.059999  | 45871300  | Tue       |
| FB     | 2013-01-09 | 30.59      | 104787700 | Wed       |
| FB     | 2013-01-10 | 31.299999  | 95316400  | Thu       |
| FB     | 2013-01-11 | 31.719999  | 89598000  | Fri       |
| AMZN   | 2013-01-07 | 268.459991 | 4910000   | Mon       |
| AMZN   | 2013-01-08 | 266.380005 | 3010700   | Tue       |
| AMZN   | 2013-01-09 | 266.350006 | 2265600   | Wed       |

分割毎にモデルを作成したい場合は `purrr::map()` 内で `analysis()` -> `bind_rows()` でデータを取り出した上で、モデル化を行えばよい。

```R
FANG_rolled <- FANG_rolled %>%
  mutate(lm_model = map(splits, ~ {
    d <- bind_rows(analysis(.)$data)
    lm(adjusted ~ volume, data = d)
  }))
FANG_rolled
```

```R
# Rolling origin forecast resampling
# A tibble: 145 x 3
   splits          id       lm_model
 * <list>          <chr>    <list>
 1 <split [52/13]> Slice001 <lm>
 2 <split [52/13]> Slice002 <lm>
 3 <split [52/13]> Slice003 <lm>
 4 <split [52/13]> Slice004 <lm>
 5 <split [52/13]> Slice005 <lm>
 6 <split [52/13]> Slice006 <lm>
 7 <split [52/13]> Slice007 <lm>
 8 <split [52/13]> Slice008 <lm>
 9 <split [52/13]> Slice009 <lm>
10 <split [52/13]> Slice010 <lm>
# … with 135 more rows
```

当然、検証用データも同じ手法で取り出すことが可能だ。

```R
bind_rows(assessment(FANG_rolled$splits[[1]])$data) %>% head(n = 20)
```

| symbol | date       | adjusted   | volume   | dayOfWeek |
|--------|------------|------------|----------|-----------|
| FB     | 2013-12-30 | 53.709999  | 68307000 | Mon       |
| FB     | 2013-12-31 | 54.650002  | 43076200 | Tue       |
| FB     | 2014-01-02 | 54.709999  | 43195500 | Thu       |
| FB     | 2014-01-03 | 54.560001  | 38246200 | Fri       |
| AMZN   | 2013-12-30 | 393.369995 | 2487100  | Mon       |
| AMZN   | 2013-12-31 | 398.790009 | 1996500  | Tue       |
| AMZN   | 2014-01-02 | 397.970001 | 2137800  | Thu       |
| AMZN   | 2014-01-03 | 396.440002 | 2210200  | Fri       |
| NFLX   | 2013-12-30 | 52.427143  | 15075200 | Mon       |
| NFLX   | 2013-12-31 | 52.595715  | 10516800 | Tue       |
| NFLX   | 2014-01-02 | 51.831429  | 12325600 | Thu       |
| NFLX   | 2014-01-03 | 51.871429  | 10817100 | Fri       |
| GOOG   | 2013-12-30 | 554.176782 | 2481300  | Mon       |
| GOOG   | 2013-12-31 | 559.796182 | 2725900  | Tue       |
| GOOG   | 2014-01-02 | 556.004972 | 3656400  | Thu       |
| GOOG   | 2014-01-03 | 551.948999 | 3345800  | Fri       |
| FB     | 2014-01-06 | 57.200001  | 68852600 | Mon       |
| FB     | 2014-01-07 | 57.919998  | 77207400 | Tue       |
| FB     | 2014-01-08 | 58.23      | 56682400 | Wed       |
| FB     | 2014-01-09 | 57.220001  | 92253300 | Thu       |


## まとめ {#まとめ}

事前に必要な単位に `data.frame` をネストすることで `{rsample}` の機能を使いつつ、より直感的に時系列データを分割することができた。また、事前にネストするというテクニックを応用すれば、今回のようなケース以外にも柔軟な交差検証用の分割が実現できるできると思われる。

それでは Happy coding !!
