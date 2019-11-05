---
title: "The first post"
date: 2019-10-10T13:03:00+09:00
draft: false
toc: true
---

ブログの開設にあたって、よく利用する記法がブログ側でどのように表現されるかを確認する。特に [Hugo](https://gohugo.io/) の [Jane](https://github.com/xianmin/hugo-theme-jane) テーマで、意図した通りに表示されるかを確認したい。なお、当面は Emacs の [org-mode](https://orgmode.org/ja/) で記事を執筆し、[ox-hugo](https://ox-hugo.scripter.co/) でエクスポートした markdown を [Netlify](https://www.netlify.com/) で公開するというフローでブログを運営していく。


## h1 {#h1}


### h1 {#h1}


#### h2 {#h2}

-    h3

    -    h5


## Table/Text {#table-text}

| name      | org-mode      | result                              |
|-----------|---------------|-------------------------------------|
| bold      | `*hoge*`      | **hoge**                            |
| italic    | `/hoge/`      | _hoge_                              |
| underline | `_hoge_`      | <span class="underline">hoge</span> |
| underline | `<u>hoge</u>` | <u>hoge</u>                         |
| strike    | `+hoge+`      | ~~hoge~~                            |
| code      | `~hoge~`      | `hoge`                              |
| verbatim  | `=hoge=`      | `hoge`                              |


## List {#list}

順序なしリスト

-   hoge
-   fuga
-   piyo

順序ありリスト

1.  hoge
2.  fuga
3.  piyo

チェックボックス

-   [ ] hoge
-   [ ] fuga
-   [ ] piyo


## Quote {#quote}

> Everything should be made as simple as possible,
> but not any simpler ---Albert Einstein


## Formula {#formula}

-   インライン `$y=f(x)$`

\\(y=f(x)\\)

-   ブロック

<!--listend-->

```text
\begin{equation}
\label{eq:1}
y=f(x)
\end{equation}
```

\begin{equation}
\label{eq:1}
y=f(x)
\end{equation}


## Footnote {#footnote}

-   org-mode[^fn:1] `[fn:hoge]`


## Horizontal Rule {#horizontal-rule}

`---`
---


## emacs-lisp {#emacs-lisp}

```emacs-lisp
(emacs-version)
```

```text
GNU Emacs 26.3 (build 2, x86_64-pc-linux-gnu, GTK+ Version 3.22.30)
 of 2019-09-17
```


## R code {#r-code}


### Table {#table}

```R
library(tidyverse)
head(iris)
```

| Sepal.Length | Sepal.Width | Petal.Length | Petal.Width | Species |
|--------------|-------------|--------------|-------------|---------|
| 5.1          | 3.5         | 1.4          | 0.2         | setosa  |
| 4.9          | 3           | 1.4          | 0.2         | setosa  |
| 4.7          | 3.2         | 1.3          | 0.2         | setosa  |
| 4.6          | 3.1         | 1.5          | 0.2         | setosa  |
| 5            | 3.6         | 1.4          | 0.2         | setosa  |
| 5.4          | 3.9         | 1.7          | 0.4         | setosa  |


### Plot {#plot}

```R
ggplot(iris, aes(x = Sepal.Length, y = Sepal.Width)) + geom_point()
```

{{< figure src="/ox-hugo/first-post_iris.png" >}}


### Code output {#code-output}

```R
sessionInfo()
```

```text
R version 3.6.1 (2019-07-05)
Platform: x86_64-pc-linux-gnu (64-bit)
Running under: Ubuntu 18.04.3 LTS

Matrix products: default
BLAS:   /usr/lib/x86_64-linux-gnu/blas/libblas.so.3.7.1
LAPACK: /usr/lib/x86_64-linux-gnu/lapack/liblapack.so.3.7.1

locale:
 [1] LC_CTYPE=en_US.UTF-8       LC_NUMERIC=C
 [3] LC_TIME=en_US.UTF-8        LC_COLLATE=en_US.UTF-8
 [5] LC_MONETARY=en_US.UTF-8    LC_MESSAGES=en_US.UTF-8
 [7] LC_PAPER=en_US.UTF-8       LC_NAME=C
 [9] LC_ADDRESS=C               LC_TELEPHONE=C
[11] LC_MEASUREMENT=en_US.UTF-8 LC_IDENTIFICATION=C

attached base packages:
[1] stats     graphics  grDevices utils     datasets  methods   base

other attached packages:
[1] forcats_0.4.0   stringr_1.4.0   dplyr_0.8.3     purrr_0.3.2
[5] readr_1.3.1     tidyr_1.0.0     tibble_2.1.3    ggplot2_3.2.1
[9] tidyverse_1.2.1

loaded via a namespace (and not attached):
 [1] Rcpp_1.0.2       cellranger_1.1.0 pillar_1.4.2     compiler_3.6.1
 [5] tools_3.6.1      zeallot_0.1.0    jsonlite_1.6     lubridate_1.7.4
 [9] lifecycle_0.1.0  gtable_0.3.0     nlme_3.1-141     lattice_0.20-38
[13] pkgconfig_2.0.3  rlang_0.4.0      cli_1.1.0        rstudioapi_0.10
[17] haven_2.1.1      withr_2.1.2      xml2_1.2.2       httr_1.4.1
[21] generics_0.0.2   vctrs_0.2.0      hms_0.5.1        grid_3.6.1
[25] tidyselect_0.2.5 glue_1.3.1       R6_2.4.0         readxl_1.3.1
[29] modelr_0.1.5     magrittr_1.5     backports_1.1.5  scales_1.0.0
[33] rvest_0.3.4      assertthat_0.2.1 colorspace_1.4-1 labeling_0.3
[37] stringi_1.4.3    lazyeval_0.2.2   munsell_0.5.0    broom_0.5.2
[41] crayon_1.3.4
```


## Reference {#reference}

-   [Jane Theme](https://www.xianmin.org/hugo-theme-jane/)
-   [ox-Hugo Cheat Sheet](https://ladicle.com/post/ox-hugo-cheat/)

[^fn:1]: <https://orgmode.org/ja/>
