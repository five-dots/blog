---
title: "org-mode で Hugo と Qiita の記事を一元管理する"
date: 2019-11-05
draft: false
toc: true
---

個人のブログを、[Hugo](https://gohugo.io/) + [Netlify](https://www.netlify.com/) で開始したが、それなりに力を入れて書いた技術記事は Qiita にも公開することで、広く読んでもらえるようにしたい。そこで、1 つの [org-mode](https://orgmode.org/ja/) ファイルから、個人ブログ向けには [ox-hugo](https://ox-hugo.scripter.co/) で、Qiita 向けには [ox-gfm](https://github.com/larstvei/ox-gfm) で Markdown ファイルを出力できないか試してみた。

-   Hugo blog
-   Qiita
-   [元となる org ファイル@github](https://github.com/five-dots/notes/blob/master/lang/org-mode/org-for-hugo-qiita/org-for-hugo-qiita.org) (Markdown に変換されてしまうので、ソースは Raw で見る)

なお、Hugo テーマは [even](https://github.com/olOwOlo/hugo-theme-even) を利用している。その他のテーマでは、この記事と異なる結果になる可能性もあるので、その点は注意が必要だ。


## Software Version {#software-version}

| Software | Version         |
|----------|-----------------|
| OS       | Ubuntu 18.04    |
| Emacs    | 26.3            |
| org-mode | 9.2.6           |
| Hugo     | 0.59.1/extended |


## Heading {#heading}


## h2 {#h2}


### h3 {#h3}


#### h4 {#h4}


##### h5 {#h5}


###### h6 {#h6}

**Memo**

-   `*` がタイトルとして扱われるため、 `**` (h2) から Heading として扱われる
-   デフォルトでは、h4 までしか出力されないため、 `#+OPTIONS: H:6` のように追加で設定が必要
-   [even](https://github.com/olOwOlo/hugo-theme-even) theme では h6 が最小


## Table/Text {#table-text}

| name      | org-mode | result                              |
|-----------|----------|-------------------------------------|
| bold      | `*hoge*` | **hoge**                            |
| italic    | `/hoge/` | _hoge_                              |
| underline | `_hoge_` | <span class="underline">hoge</span> |
| strike    | `+hoge+` | ~~hoge~~                            |
| code      | `~hoge~` | `hoge`                              |
| verbatim  | `=hoge=` | `hoge`                              |
| greek     | `\alpha` | &alpha;                             |

**Memo**

-   org-mode 記法でのアンダーラインは反映されていない


## List {#list}

順序なしリスト

-   hoge
-   fuga
-   piyo

チェックボックス

-   [ ] hoge
-   fuga
-   [ ] piyo

順序ありリスト

1.  hoge
2.  fuga
3.  piyo

**Memo**

-   チェックボックスは、チェックをつけてしまうと表示されなくなってしまう


## Quote {#quote}

> Everything should be made as simple as possible,
> but not any simpler ---Albert Einstein

**Memo**

-   問題なく表示されている


## Formula {#formula}

-   インライン `$y=f(x)$`

\\(y=f(x)\\)

-   ブロック `$$y=f(x)$$`

\\[
y=f(x)
\\]

-   ブロック

<!--listend-->

```text
\begin{equation}
y=f(x)
\end{equation}
```

\begin{equation}
y=f(x)
\end{equation}

**Memo**

-   `\begin{equation} ... \end{equation}` ブロックは Qiita では有効でない


## Footnote {#footnote}

-   org-mode[^fn:1] `[fn:name]`

**Memo**

-   Qiita では有効でない


## Horizontal Rule {#horizontal-rule}

`-----`

---

**Memo**

-   5 つの `-` で Markdown 側では `---` に変換される


## Code Block {#code-block}


### Emacs Lisp {#emacs-lisp}

```emacs-lisp
(emacs-version)
```

```text
GNU Emacs 26.3 (build 2, x86_64-pc-linux-gnu, GTK+ Version 3.22.30)
 of 2019-09-17
```


### R {#r}


#### Code output {#code-output}

```R
R.version
```

```text
               _
platform       x86_64-pc-linux-gnu
arch           x86_64
os             linux-gnu
system         x86_64, linux-gnu
status
major          3
minor          6.1
year           2019
month          07
day            05
svn rev        76782
language       R
version.string R version 3.6.1 (2019-07-05)
nickname       Action of the Toes
```


#### Table {#table}

```R
library(tidyverse)
head(iris)
```


#### Plot {#plot}

```R
library(ggplot2)
ggplot(iris, aes(x = Sepal.Length, y = Sepal.Width)) + geom_point()
```

{{< figure src="https://dl.dropboxusercontent.com/s/4j5jstkg1fsvdiw/iris.png" >}}

**Memo**

-   org-babel からプロット画像を出力する先を Dropbox フォルダに設定し共有リンク機能を利用して画像を公開する
-   Hugo 向けには `ox-hugo` が自動で画像ファイルを `static/` へ移動してくれるので、本来は Dropbox を利用する必要はないが、Qiita と記事を共用するためには必要
-   `:exports code` に設定することで、babel からのリンク出力を停止しつつ、ローカルではプロットをインライン画像で確認できる
-   [ここ](http://ijmp320.hatenablog.jp/entry/2015/01/18/171807)の記事を参考に、Dropbox の直リンクに変換し、以下のように HTML として出力する
-   将来的には Dropbox API + Emacs Lisp で自動化したい

<!--listend-->

```org
#+attr_html:
[[https://dl.dropboxusercontent.com/s/4j5jstkg1fsvdiw/iris.png]]
```


### Python {#python}

```python
import sys
sys.version
```

```text
3.6.8 (default, Oct  7 2019, 12:59:55)
[GCC 8.3.0]
```


## Reference {#reference}

-   [Org-mode で記事を書いて Hugo 向け markdown を ox-hugo で自動生成する話](https://sfus.net/blog/2018/12/org-mode-with-ox-hugo/)
-   [ox-Hugo Cheat Sheet](https://ladicle.com/post/ox-hugo-cheat/)
-   [【備忘録】Dropbox の画像の URL（直リンク）の取得](http://ijmp320.hatenablog.jp/entry/2015/01/18/171807)

[^fn:1]: <https://orgmode.org/ja/>
