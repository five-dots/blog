+++
title = "1 つの org ファイルから Hugo と Qiita の Markdown を生成する"
date = 2019-11-05
tags = ["org-mode", "markdown", "hugo"]
categories = ["emacs"]
draft = false
toc = true
+++

普段、技術的なメモは Emacs の [org-mode](https://orgmode.org/ja/) で書いたものを [自身の github リポジトリ](https://github.com/five-dots/notes) に公開している。ただ、これはあくまで個人的なメモなので、それほど他人が見ることを意識したものではない。より見やすい記事として公開するために、個人のブログを、[Hugo](https://gohugo.io/) + [Netlify](https://www.netlify.com/) で開始したが、同時に力を入れて書いた技術記事は Qiita にも公開して、広く読んでもらえるようにしたい。

そこで、これまで通り、org ファイルは 1 つのリポジトリで管理しつつ、個人ブログ向けと Qiita 向けの Markdown をエクスポートできないか試してみた。

1 つの [org-mode](https://orgmode.org/ja/) ファイルから、個人ブログ向けには [ox-hugo](https://ox-hugo.scripter.co/) で、Qiita 向けには [ox-gfm](https://github.com/larstvei/ox-gfm) で Markdown ファイルを出力できないか試してみた。


## 生成されるファイル {#生成されるファイル}

1.  [元となる org ファイル@github](https://github.com/five-dots/notes/blob/master/lang/org-mode/org-for-hugo-qiita/org-for-hugo-qiita.org)
2.  [個人ブログの記事](https://objective-boyd-9b8f29.netlify.com/2019/11/org-for-hugo-qiita/) (Markdown ファイル@github)
3.  Qiita

なお、Hugo テーマは [even](https://github.com/olOwOlo/hugo-theme-even) を利用している。その他のテーマでは、この記事と異なる結果になる可能性もあるので、その点は注意が必要だ。

-   ox-md
    -   table が HTML で出力されるので、文字修飾が有効にならない
    -   Footnotes が有効にならない
    -   code block が `` ```lang``` `` の形式で出力されない
-   ox-gfm
    -   Footnotes が有効にならない
-   ox-qmd
    -
-   ox-pandoc


## 動作確認バージョン {#動作確認バージョン}

| ソフトウェア | バージョン      |
|--------|------------|
| OS       | Ubuntu 18.04    |
| Emacs    | 26.3            |
| org-mode | 9.2.6           |
| Hugo     | 0.59.1/extended |


## 見出し {#見出し}


## Hugo=h2 / Qiita=h1 {#hugo-h2-qiita-h1}


### Hugo=h3 / Qiita=h2 {#hugo-h3-qiita-h2}


#### Hugo=h4 / Qiita=h3 {#hugo-h4-qiita-h3}


##### Hugo=h5 / Qiita=h4 {#hugo-h5-qiita-h4}


###### Hugo=h6 / Qiita=h5 {#hugo-h6-qiita-h5}


### Memo {#memo}

-   Hugo では `*` がタイトルとして扱われるため、 `**` (h2) から見出しとして機能する
-   org-mode のデフォルトでは、h4 までしか出力されないため、 `#+OPTIONS: H:6` のように追加で設定が必要
-   [Even](https://github.com/olOwOlo/hugo-theme-even) theme では h6 が最小


## 表・文字修飾 {#表-文字修飾}

| 項目   | org-mode          | 結果                                     |
|------|-------------------|----------------------------------------|
| 太字   | `*bold*`          | **bold**                                 |
| イタリック | `/italic/`        | _italic_                                 |
| 下線   | `_underline_`     | <span class="underline">underline</span> |
| 取り消し線 | `+strikethrough+` | ~~strikethrough~~                        |
| コード | `~code~`          | <kbd>code</kbd>                          |
| 逐語   | `=verbatim=`      | `verbatim`                               |
| 上付き | `hoge^{super}`    | hoge<sup>super</sup>                     |
| 下付き | `hoge_{sub}`      | hoge<sub>sub</sub>                       |
| ギリシャ文字 | `\alpha`          | &alpha;                                  |


### Memo {#memo}

-   下線は Hugo で `span.underline {text-decoration: underline;}` を `static/css/custom.css` に追加した場合の表示結果
-   コードは Hugo で `(setq org-hugo-use-code-for-kbd t)` にした場合の表示結果


## リスト {#リスト}


### 順序なしリスト {#順序なしリスト}

-   hoge
    -   hoge
    -   fuga
    -   piyo
-   fuga
-   piyo


### チェックボックス {#チェックボックス}

-   [ ] hoge
-   fuga
-   [ ] piyo


### 順序ありリスト {#順序ありリスト}

1.  hoge
    1.  hoge
    2.  fuga
    3.  piyo
2.  fuga
3.  piyo


### 定義リスト {#定義リスト}

リンゴ
: 赤いフルーツ

オレンジ
: 橙色フルーツ

**Memo**

-   チェックボックスは、チェックをつけてしまうと表示されなくなってしまう


## 引用 {#引用}

> Everything should be made as simple as possible,
> but not any simpler ---Albert Einstein

**Memo**

-   問題なく表示されている


## 数式 {#数式}

-   インライン `$y=f(x)$`

\\(y=f(x)\\)

\\(y=f(x)\\)

-   ブロック `$$y=f(x)$$`

\\[ y=f(x) \\]
\\[y=f(x)\\]

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


## 脚注 {#脚注}

-   org-mode[^fn:1] `[fn:name]`

**Memo**

-   Qiita では有効でない


## 水平線 {#水平線}

`-----`

---

**Memo**

-   5 つの `-` で Markdown 側では `---` に変換される


## コードブロック {#コードブロック}


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

| Sepal.Length | Sepal.Width | Petal.Length | Petal.Width | Species |
|--------------|-------------|--------------|-------------|---------|
| 5.1          | 3.5         | 1.4          | 0.2         | setosa  |
| 4.9          | 3           | 1.4          | 0.2         | setosa  |
| 4.7          | 3.2         | 1.3          | 0.2         | setosa  |
| 4.6          | 3.1         | 1.5          | 0.2         | setosa  |
| 5            | 3.6         | 1.4          | 0.2         | setosa  |
| 5.4          | 3.9         | 1.7          | 0.4         | setosa  |


#### Plot {#plot}

```R
library(ggplot2)
ggplot(iris, aes(x = Sepal.Length, y = Sepal.Width)) + geom_point()
```

{{< figure src="https://dl.dropboxusercontent.com/s/4j5jstkg1fsvdiw/iris.png" >}}

**Memo**

-   org-babel からプロット画像を出力する先を Dropbox フォルダに設定し共有リンク機能を利用して画像を公開する
-   Hugo 向けには ox-hugo が自動で画像ファイルを `static/ox-hugo/` へ移動してくれるので、本来は Dropbox を利用する必要はないが、Qiita と記事を共用するためには必要
-   `:exports code` に設定することで、babel からのリンク出力を停止しつつ、ローカルではプロットをインライン画像で確認できる
-   [ここ](http://ijmp320.hatenablog.jp/entry/2015/01/18/171807)の記事を参考に、Dropbox の直リンクに変換し、以下のように HTML として出力する
-   将来的には Dropbox API + Emacs Lisp で自動化したい

<!--listend-->

```text
#+attr_html:
[[https://dl.dropboxusercontent.com/s/4j5jstkg1fsvdiw/iris.png]]
```

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


## 参考 {#参考}

-   [Org-mode で記事を書いて Hugo 向け markdown を ox-hugo で自動生成する話](https://sfus.net/blog/2018/12/org-mode-with-ox-hugo/)
-   [ox-Hugo Cheat Sheet](https://ladicle.com/post/ox-hugo-cheat/)
-   [【備忘録】Dropbox の画像の URL（直リンク）の取得](http://ijmp320.hatenablog.jp/entry/2015/01/18/171807)
-   [emacs の org-mode で書いた記事を qiita に投稿する org-qiita.el](https://qiita.com/dwarfJP/items/594a8d4b0ac6d248d1e4)
-   [Qiita の数式チートシート](https://qiita.com/PlanetMeron/items/63ac58898541cbe81ada)
-   [Org latex exports $ … $ math as \\( … \\) : can this be avoided?](https://emacs.stackexchange.com/questions/47733/org-latex-exports-math-as-can-this-be-avoided)
-   [12.17 Advanced Export Configuration](https://orgmode.org/manual/Advanced-Export-Configuration.html)


## TODOs {#todos}


### org-qiita.el for upload {#org-qiita-dot-el-for-upload}


### Definition list {#definition-list}


### github/markup, wallyqs/org-ruby {#github-markup-wallyqs-org-ruby}


### 他のリンク精査 {#他のリンク精査}


### Qiita + table {#qiita-plus-table}

[^fn:1]: <https://orgmode.org/ja/>
