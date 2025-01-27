+++
title = "R でトレード戦略開発に使えるデータセット"
date = 2019-11-13
tags = ["r", "stock"]
categories = ["finance"]
draft = false
toc = true
+++

R でトレード戦略を開発する場合、 `quantmod::getSymbols()` を利用して、Yahoo Finance から最新のデータをダウンロードしたり、データベンダーから有料データを購入したりして、データを用意することが一般的だと思われる。ただ、ブログ内で R パッケージの利用例を示すときなどは、その都度どこかからダウンロードさせるよりも、R のデータセットが利用できると、読者に負担を掛けずに済むため、読みやすい記事になるだろう。

この記事では、主に米国株式のデータを中心に、有用と思われる データセットをリストアップしていく。良いものが見つかり次第追記していきたい。


## `sp500ret` from `{rugarch}` {#sp500ret-from-rugarch}

-   S&P 500 指数の日足終値をベースにした対数収益率
-   データソースは、Yahoo Finance
-   1987-03-10 から 2009-01-30 まで
-   `data.frame` 形式
-   rowname が日付になっている

<!--listend-->

```R
library(rugarch)
data(sp500ret)
tail(sp500ret)
```

```text
               SP500RET
2009-01-23  0.005363236
2009-01-26  0.005537856
2009-01-27  0.010866312
2009-01-28  0.033006834
2009-01-29 -0.033681051
2009-01-30 -0.023052810
```


## `sp500` from `{depmixS4}` {#sp500-from-depmixs4}

-   S&P 500 指数の月足四本値と終値ベースの対数収益率
-   1950-02-28 から 2012-01-31 まで
-   `data.frame` 形式
-   rowname が日付になっている

<!--listend-->

```R
library(depmixS4)
data(sp500)
tail(sp500)
```

```text
              Open    High     Low   Close     Volume       logret
2011-08-31 1213.00 1230.71 1209.35 1218.89 5267840000 -0.058467492
2011-09-30 1159.93 1159.93 1131.34 1131.42 4416790000 -0.074467128
2011-10-31 1284.96 1284.96 1253.16 1253.30 4310210000  0.102306592
2011-11-30 1196.72 1247.11 1196.72 1246.96 5801910000 -0.005071483
2011-12-30 1262.82 1264.12 1257.46 1257.60 2271850000  0.008496553
2012-01-31 1313.53 1321.41 1306.69 1312.41 4235550000  0.042659999
```


## `FANG` from `{tidyquant}` {#fang-from-tidyquant}

-   FANG (Facebook, Amazon, Netflix, Goolge) の日足四本値
-   `tibble` 形式
-   2013-01-02 から 2016-12-30 まで

<!--listend-->

```R
library(tidyquant)
data(FANG)
tail(FANG)
```

```text
# A tibble: 6 x 8
  symbol date        open  high   low close  volume adjusted
  <chr>  <date>     <dbl> <dbl> <dbl> <dbl>   <dbl>    <dbl>
1 GOOG   2016-12-22  792.  793.  789.  791.  969100     791.
2 GOOG   2016-12-23  791.  793.  787.  790.  623400     790.
3 GOOG   2016-12-27  791.  798.  788.  792.  789100     792.
4 GOOG   2016-12-28  794.  794.  783.  785. 1132700     785.
5 GOOG   2016-12-29  783.  786.  779.  783.  742200     783.
6 GOOG   2016-12-30  783.  783.  770.  772. 1760200     772.
```


## `edhec` from `{PerformanceAnalytics}` {#edhec-from-performanceanalytics}

-   EDHEC composite hedge fund style index returns
-   13 のスタイル別ヘッジファンドの月次リターンデータ
-   `xts` 形式
-   1997-01-31 から 2009-08-31 まで

<!--listend-->

```R
library(PerformanceAnalytics)
data(edhec)
tail(edhec[, 1:4]) # 最初の 4 列のみ抽出
```

```text
           Convertible Arbitrage CTA Global Distressed Securities Emerging Markets
2009-03-31                0.0235    -0.0180                0.0022           0.0350
2009-04-30                0.0500    -0.0140                0.0387           0.0663
2009-05-31                0.0578     0.0213                0.0504           0.0884
2009-06-30                0.0241    -0.0147                0.0198           0.0013
2009-07-31                0.0611    -0.0012                0.0311           0.0451
2009-08-31                0.0315     0.0054                0.0244           0.0166
```
