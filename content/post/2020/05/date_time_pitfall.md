+++
title = "R の Date/POSIXct 型ではまったこと"
date = 2020-05-09
tags = ["r"]
categories = ["programming"]
draft = false
toc = true
+++

R の Date 型、POSIXct 型を利用していて過去にはまったポイントを備忘録として整理しておく。


## for loop 内で `Date` が `numeric` になってしまう問題 {#for-loop-内で-date-が-numeric-になってしまう問題}

-   `for` loop 内で class attribute が欠落してしまうことが原因
    -   [For loops in R can lose class information@R-bloggers](https://www.r-bloggers.com/for-loops-in-r-can-lose-class-information/)
    -   `Date` は `numeric` に class attribute を追加したものであるため

<!--listend-->

```R
dates <- c(as.Date("2020-05-01"), as.Date("2020-05-02"))

for (date in dates) {
  print(date)
}
```

```R
[1] 18383
[1] 18384
```

<br />

-   対策 1: <kbd>list</kbd> に変換してからループする

<!--listend-->

```R
for (date in as.list(dates)) {
  print(date)
}
```

```R
[1] "2020-05-01"
[1] "2020-05-02"
```

<br />

-   対策 2: インデックスでアクセスする

<!--listend-->

```R
for (i in seq_along(dates)) {
  print(dates[i])
}
```

```R
[1] "2020-05-01"
[1] "2020-05-02"
```

<br />


## `POSIXct` から `Date` への変換で日付がずれる問題 {#posixct-から-date-への変換で日付がずれる問題}

-   参考: [R: POSIXct -> Date で日付がズレる@Qiita](https://qiita.com/kota9/items/657c8c0ac5092e3ec1ff)

<!--listend-->

```R
td <- as.POSIXct("2020-05-01")
as.Date(td)
```

```R
[1] "2020-05-01"
```

<br />

-   これは `as.Date()` は元の `POSIXct` のタイムゾーンを意識せず、デフォルトで UTC へ変換してしまうことが原因
    -   `as.POSIXct()` で作成した場合、デフォルトでシステムのタイムゾーンを利用する (この場合は、JST)
    -   そのため、JST から 9 時間分の差が発生する
-   以下の例を見れば、違いが良くわかる

<!--listend-->

```R
as.Date(as.POSIXct("2020-05-01 8:00:00")) # 2020-04-30 23:00 へ変換されてから、時間情報が削除されている
as.Date(as.POSIXct("2020-05-01 9:00:00")) # 2020-05-01 00:00 へ変換されてから、時間情報が削除されている
```

```R
[1] "2020-04-30"
[1] "2020-05-01"
```

<br />

-   対策 1： <kbd>tz</kbd> を指定すれば問題ない
    -   変換前と変換後のタイムゾーンを揃えることを意識しておけば良い

<!--listend-->

```R
# UTC に統一して変換
td <- as.POSIXct("2020-05-01", tz = "UTC")
as.Date(td)

# もしくは、JST に統一して変換
## td <- as.POSIXct("2020-05-01")
## as.Date(td, tz = "Asia/Tokyo")
```

```R
[1] "2020-05-01"
```

<br />

-   対策 2: `lubridate::as_date()` を利用する
    -   `lubridate::as_Date()` は、元の `POSIXct` のタイムゾーンを保持して変換してくれる

<!--listend-->

```R
td <- as.POSIXct("2020-05-01")
lubridate::as_date(td)
```

```R
[1] "2020-05-01"
```

<br />


## ミリ秒の丸め問題 {#ミリ秒の丸め問題}

-   文字列から `POSIXct` を作成する際に、ミリ秒がずれる (切り捨てられる)
    -   [R issue with rounding milliseconds@Stackoverflow](https://stackoverflow.com/questions/10931972/r-issue-with-rounding-milliseconds)

<!--listend-->

```R
options(digits.secs = 3)
ms_dt <- as.POSIXct("2019-06-28 12:34:01.123", format = "%Y-%m-%d %H:%M:%OS")
ms_dt
```

```R
[1] "2019-06-28 12:34:01.122 JST"
```

<br />

-   対策 1: <kbd>lubridate::ymd_hms()</kbd> ならずれない

<!--listend-->

```R
options(digits.secs = 3)
lubridate::ymd_hms("2019-06-28 12:34:01.123", tz = "Asia/Tokyo")
```

```R
[1] "2019-06-28 12:34:01.123 JST"
```

<br />

-   ミリ秒単位の経過時間を POSIXct に変換する
    -   株価のティックデータなどで必要になる手法
    -   [R How to convert milliseconds from origin to date and keep the milliseconds@Stackoverflow](https://stackoverflow.com/questions/49828433/r-how-to-convert-milliseconds-from-origin-to-date-and-keep-the-milliseconds)
    -   1000 で割って秒数に換算する (+0.0005 を足すことで丸め誤差を消すことができる)

<!--listend-->

```R
msec <- 1506378448123
dt <- as.POSIXct(msec/1000, origin = "1970-01-01", tz = "America/Chicago")
format(dt + 0.0005, "%Y-%m-%d %H:%M:%OS3")
```

```R
[1] "2017-09-25 17:27:28.123"
```

<br />

-   <kbd>lubridate::as_datetime()</kbd> でも同じようにずれる

<!--listend-->

```R
lubridate::as_datetime(msec/1000 + 0.0005)
```

```R
[1] "2017-09-25 22:27:28.123 UTC"
```

<br />
