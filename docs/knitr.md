---
title: "**knitr**: Rによる美麗で柔軟そして高速な動的レポート生成"
author:
  - "著者: Xie, Yihui (谢益辉)"
  - "翻訳者: Katagiri, Satoshi^[twitter@[ill_identified](https://twitter.com/ill_identified)] (片桐 智志)"
site: bookdown::bookdown_site
description : "knitr ドキュメントの非公式日本語訳 Unofficial Japanese translation of Yihui's knitr documentation"
documentclass: bxjsreport
link-citations: true
linkcolor: blue
citecolor: blue
urlcolor: magenta
github-repo: Gedevan-Aleksizde/knitr-doc-ja
monofont: Ricty Discord
jmonofont: Ricty Discord
---

# knitr {-}


<!-- inline で書くと見づらいのでここに移動 -->

---
date: "ver. 1.6 (2022/02/19 00:40:13 JST, 本家最終更新時刻: [2022/01/25 16:17:20 JST](https://github.com/rbind/yihui/tree/master/content/knitr))"
---

:::{.infobox .important data-latex="{important}"}
本稿は [CC BY-NC-SA 4.0 (クリエイティブ・コモンズ 表示 - 非営利 - 継承 4.0 国際)ライセンス](https://creativecommons.org/licenses/by-nc-sa/4.0/deed.ja) で提供されています. Yihui 氏によるオリジナルは https://yihui.org/knitr/ で読むことができます.

This is an unofficial Japanese translation of Yihui's **knitr** documentation, which is licensed under [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/).  The original documentation by Yihui is [here](https://yihui.org/knitr/).
:::

[![](https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png)](https://creativecommons.org/licenses/by-nc-sa/4.0/deed.ja)

:::{.infobox .tip data-latex="{tip}"}
**訳注**: Yihui 氏のサイトの一連のドキュメントはかなり以前から氏のブログ投稿として断続的に更新されてきたものを編纂しているため, 内容の重複した記述がいくつかありますし, RStudio および R Markdown が普及している現在では, やや out-of-date な記述も見られます (現在では Sweave はあまり使いません). そのため, **サイドバーの目次から辿れるページ, およびそれらでリンクされているページ以外は重要度が低いと見なし, 翻訳していません**. 同様の理由から「用例」も一部を除き翻訳していません (**共同翻訳者は常に歓迎します**: https://github.com/Gedevan-Aleksizde/knitr-doc-ja).

しかしながら翻訳時の **knitr** および R Markdown に関する日本語の情報流通を鑑みるに, 非常に有用なドキュメントであると翻訳者は確信しています. それぞれのトピックがどの環境を想定したものなのか, よく確認してご覧ください.
:::

## 概要 {-}

**knitr** パッケージは R を使用した透明な動的なレポート作成エンジンとしてデザインされ, 長年にわたって存在した Sweave の問題を解決するとともに他のアドオンパッケージの機能を統合しています (**knitr**
&asymp; Sweave + cacheSweave + pgfSweave + weaver + `animation::saveLatex` +
`R2HTML::RweaveHTML` + `highlight::HighlightWeaveLatex` + 0.2 * brew + 0.1 *
SweaveListingUtils + その他).

[<img src="https://db.yihui.org/imgur/yYw46aF.jpg" align="right" width=200 style="margin-left: 1em;" />](https://amzn.com/1498716962)

- 透明性とはユーザーが入力と出力に完全にアクセスできることを意味します. 例えば R ターミナル上では  `1 + 2` は `[1] 3` を出力しますが, **knitr**によってユーザーは `1 + 2` を LaTeX の `\begin{verbatim}` と `\end{verbatim}` や, HTML タグ `<div class="rsource">` と `</div>` の間に出力したりできますし, `[1] 3` を `\begin{Routput}` と `\end{Routput}` の間に出力できたりします. 詳細は \@ref(hooks) 章『フック』を参照してください.
- **knitr** は R コードを Rターミナル上で実行した場合でもユーザーの意図する結果と一致することを目指しています. 例えば `qplot(x, y)` でそのままグラフを出力できます (`print()` は不要です) し, デフォルトでコードチャンク内の**全ての**プロットが出力されます.
- **`pgfSweave`** や **`cacheSweave`** といったパッケージは有用な機能を Sweave に提供しました (高品質な tikz のグラフィックとキャッシュ) が, **knitr** はこの実装をより簡単にしました.
- **knitr** のデザインによってあらゆる言語 (例: R, Python, awk) が入力可能となり, あらゆるマークアップ言語の形式で出力が可能となりました (例: LaTeX, HTML, Markdown, AsciiDoc,  reStructuredText).

このパッケージは [GitHub](https://github.com/yihui/knitr) 上で開発されており, インストール手順と[FAQ](#FAQs)もここで参照できます. [パッケージの README](https://github.com/yihui/knitr#readme) も確認してください. このサイトは **knitr** の完全なドキュメントとして提供されており, [メインマニュアル](#manual), [グラフィックス](#graphics), [用例](#examples), [knitr-examples](https://github.com/yihui/knitr-examples) を閲覧することができます. さらに統合的な参考資料としては, [knitr book](https://github.com/yihui/knitr-book) (邦訳未刊行) をご覧ください.

## モチベーション {-}

Sweave の拡張の難しいところは **`utils`** パッケージから大量のコードをコピーする必要があることでした (`SweaveDrivers.R` のコードは700行を超えています). そしてこれは上述の2つのパッケージ両方での作業です. コードをコピーペーストしたら, パッケージ開発者は R の公式バージョンの変更にとても慎重に対応せねばなりません --- うんざりする仕事だとは思いませんか. **knitr** パッケージでは sweave の処理の全行程を小規模な管理しやすい関数群へとモジュール化したため, 喜ばしいことにメンテナンスも機能拡張も容易になりました (例えば簡単に HTML への対応もできるようになりました). その一方で **knitr** は多くのビルトイン機能を持ちますが, パッケージの機能のコア部分をハックする必要のないようになっています. そして, Sweave のマニュアルの FAQ の項目にあったいくつかの問題は,  **knitr** は直接解決することができます.

> 旧態依然とした態度をプログラム構造へと変えましょう. 「我々の主な仕事は何をすべきかコンピュータに教えることである」と考えるのをやめ, むしろ「人間に対してコンピュータにさせたかったことを説明する」ように注力することです
>
> --- ドナルド E. クヌース 『文芸的プログラミング』, 1984

## 特徴 {-}

本パッケージのアイディアは他のパッケージから借用し, いくつか (キャッシュなど) は異なる形で再実装されています. 以下にパッケージの特徴をいくつか挙げます:

- **信頼性の高い**出力: バックエンドで [**`evaluate`**](https://cran.rstudio.com/package=evaluate) を使用することで, **knitr** においてあらゆるコードは, 結果のプリント, グラフのプロットそして警告・メッセージ・エラーでさえ, R ターミナルでするのと同様に書き出されます. (厳密にプログラミングする場合, これらは無視するべきではありません. 特に警告は).
  - **`ggplot2`** や **`lattice`** のようなグリッドベースのグラフィックパッケージでは, ユーザーはしばしば `print()` を書くのを忘れてしまう, これは些細な問題です. 実際に R ターミナルで出力する時に `print()` は不要ですから. **knitr** ではあなたが意図したとおりの結果を得ることができます.
- キャッシュ機能の組み込み: **`cacheSweave`** のようですが,  しかし **knitr** ではキャッシュの保存や遅延読み込みのため base R の関数を直接使い, そして最も顕著な違いはキャッシュされたチャンクは依然として出力もできることです (**`cacheSweave`** ではキャッシュされたチャンクは. `print()` を明示的に書いたとしても何も出力しませんが, **knitr** はキャッシュされても通常通り出力します).
- R コードの整形: R コードの自動整形に [**formatR**](https://yihui.org/formatr/) パッケージが使用されます (長い行の折り返し, スペースとインデントの追加, など). これは `keep.source=FALSE` としてもコメントが犠牲になることはありません.
- 20 を超えるグラフィックデバイスを直接サポートしています. チャンクオプションに `dev='CairoPNG'` と書けば, すぐさま [**Cairo**](https://cran.rstudio.com/package=Cairo) パッケージの `CairoPNG()` に切り替えることができますし,  `dev='tikz'` なら [**tikzDevice**](https://cran.rstudio.com/package=tikzDevice) パッケージの `tikz()` に切り替えられます. これ以上簡単にすることなどできないでしょう? これらの組み込みのデバイス (正確に言うならラッパー) は単位にインチを使用しています. ビットマップでもです (ピクセルは `dpi` オプションに基づいて変換されます. デフォルトは 72 です).
- グラフィックスにおけるさらなる柔軟性:
  - 出力の幅と高さをさらに指定することが出来ます (`fig.width` はグラフィックデバイスのオプションで, `out.width` は文書に出力する際のものです. たとえば LaTeX なら `out.width='.8\\textwidth'` とします.)
  - グラフの位置も再調整できます. グラフが生成された位置に正確に掲載することも, チャンクとまとめて表示することもできます (`fig.show='hold'` オプションを使用してください).
  - 最後のグラフだけ欲しいというのでない限り (`fig.keep='last'` オプションを使用してください), コードチャンクごとに複数のグラフが保存されます.
- R コードはコードチャンクに直接書くだけでなく, 他の R スクリプトファイルから読み込むこともでき, 文書を書きながらコードを実行するのが簡単になりました (特に LyX で有益です).
- パワーユーザーのために, さらなるカスタマイズが可能になっています.
  - Rコードをパースする際に正規表現は拒否されます. つまり, `<<>>=`, `@`, `\Sexpr{}` などを使う必要はありません. もしやりたいのなら任意のパターンを使用できます. 例: `%% begin.rcode`, `%% end.rcode`.
  - フック (hook) は出力の制御を拒否します. 例えばエラーメッセージを赤い太字で表示したいとしたら, ソースコードをイタリック体で表示したいとしたら, フック機能をコードの実行前および実行語に定義することが出来ます. フック機能はこのパッケージの力を無限に拡大する可能性を持ちます. (例えば, アニメーションとか, rgl 3D プロットとか...)

多くの取り組みが, デフォルトでの美しい出力と可読性の向上をもたらしています. 例えばコードチャンクはシンタックスハイライトされて表示され, LaTeX ではさらに ([**framed**](https://www.ctan.org/pkg/framed) パッケージを使うことで) 薄灰色の背景色がつきます. そのため他のテキストよりやや目立って表示されます. 読書体験はきっと `verbatim` や `Verbatim` 環境を使うよりも良いものとなります. プロンプトで表示される先頭の `>` や `+` はデフォルトでは**出力されません** (`prompt=TRUE` で表示することもできます).  このような記号はコードに割り込んでくるので, コードをコピーして自分で実行する際に非常に不便であり, ドキュメントを読む時にまったくもって迷惑していました (訳注: **とてもよくわかる**).

## 謝辞 {-}

Sweave, **`pgfSweave`**, **`cacheSweave`**, **`brew`**, **`decumar`**,
**`R2HTML`**, **tikzDevice**, **`highlight`**, **`digest`**, **`evaluate`**, **`roxygen2`**, そしてもちろん, R の開発者に対し, 多くのひらめきとツールをもたらしてくれたことに感謝します. 多くのベータテスターによる [フィードバック](https://github.com/yihui/knitr/issues) に心から感謝します. 本パッケージは **documar** のデザインに基づいて始まりました.

## FOAS {-}

<img src="https://db.yihui.org/imgur/XmT6L3F.png" style="float:right;margin-left:1em" width=150 />
**knitr** が [Foundation for Open Access Statistics](http://www.foastat.org/) (FOAS) の協力のもとで開発されたことを誇りに思います. FOAS はフリーウェア, オープンアクセス, 統計学における再現可能な研究を推進するという世界規模の課題を持った非営利の公益法人です.

<!--chapter:end:index.Rmd-->

<!---
title: Options
subtitle: Chunk options and package options
date: '2020-06-30'
slug: options
show_toc: true
--->

# オプション {#options}

チャンクオプションとパッケージオプションについて

オリジナルのページ: https://yihui.org/knitr/options/

オリジナルの更新日: 2020-06-30

----

:::{.infobox .tip data-latex="{tip}"}
このドキュメントでは **knitr** 全般の機能を紹介しており, `Rnw` や `Rhtml` の仕様についても言及しています. R Markdown でも **knitr** の機能は使われますが, R Markdown 独自の中間処理によって, 最終的な出力がここで説明されているものと異なる可能性がある点に注意してお読みください.
:::

**knitr** パッケージはソースコード, テキスト, グラフ, チャンクで使用するプログラミング言語といった, コードチャンクのコンポネントのほとんど全部をカスタマイズするための多くのオプションを提供します. `knit` 処理のカスタマイズをパッケージレベルでカスタマイズするオプションもあります. この章では **knitr** で使用できる全てのチャンクオプションとパッケージオプションを解説します. 以下のリスト中で, オプションのデフォルト値になっているものはカッコ内に表記しています.

## チャンクオプション一覧 {#chunk-options}

チャンクオプションはチャンクのヘッダに書きます. チャンクヘッダの構文は文書フォーマットがなんであるかに依存します. 例えば `.Rnw` ファイル (R + LaTeX) であれば, `<< >>=` という記号の中に書きます. `.Rmd` ならば, チャンクヘッダは ```` ```{r} ```` 内に書きます. 以下の例は主に `.Rmd` (R Markdown) の場合ですが, ほとんどのチャンクオプションはどのフォーマットでも使用可能です.

チャンクオプションは以下のように `タグ名=値` という形式で書きます.

````
```{r, my-chunk, echo=FALSE, fig.height=4, dev='jpeg'}
```
````

**チャンクラベル**は特殊なチャンクオプションです (例: 先ほどの例の `my-chunk` がそれにあたります). これは唯一のタグが不要なチャンクオプションです (つまり, 値のみ書くことになります). もし `タグ名=値` の形式で書きたいのならば, チャンクオプション名の `label` を明示的に使うこともできます.

````
```{r label="my-chunk"}
```
````

各チャンクのラベルは文書内において一意であることが前提です. 特にキャッシュとグラフのファイル名はチャンクラベルで紐付けているため重要です. ラベルのないチャンクは `unnamed-chunk-i` という形式でラベル名が割り当てられます. `i` は順に整数が割り当てられます.

文書全体のチャンクオプションのデフォルト値を変更するために `knitr::opts_chunk$set()` を使うことができます. 例えば以下のようなチャンクを文書の冒頭に書きます.

````
```{r, setup, include=FALSE}
knitr::opts_chunk$set(
  comment = '', fig.width = 6, fig.height = 6
)
```
````

チャンクオプションの豆知識をいくつか掲載します.

1. チャンクヘッダは1行で書かねばなりません. 改行してはいけません.
2. チャンクラベルとファイルパスにスペース, ピリオド `.`, アンダースコア `_` を使用するのは避けましょう. セパレータが必要ならば, ハイフン `-` の使用を推奨します. 例えば `setup-options` はラベル名として望ましいですが `setup.options` や `chunk 1` は良くありません. `fig.path = 'figures/mcmc-'` はパス名として良いですが, `fig.path = 'markov chain/monte carlo'` は良くありません.
3. 全てのオプションの値は **R の構文として適切でなければなりません**. チャンクオプションを関数の引数のように考えると良いでしょう.
  - 例えば **character** 型をとるオプションは引用符で囲まなければなりません. 例: `results = 'asis'` や `out.width = '\\textwidth'`. ただしリテラルのバックスラッシュは二重のバックスラッシュが必要なことを忘れないでください.
  - 理論上はチャンクラベルもまた引用符で囲む必要がありますが, 利便性のため書かなくとも自動で引用符が追加されます (例: ```` ```{r, 2a}``` ```` は ```` ```{r, label='2a'}``` ```` として扱われます).
  - R のコードとして有効なものである限り, いくらでも複雑な構文を書くことができます.

チャンクオプションの書き方の別の方法として, コードチャンクの本文内に `#| ` の後に書くことができます. 例えば以下のように.

````
```{r}
#| my-chunk, echo = FALSE, fig.width = 10,
#| fig.cap = "This is a long long
#|   long long caption."
plot(cars)
```
````

チャンクオプションとコードの間を1行開けるかどうかは任意です. この記法はオプションの改行が許容されます. 好きなだけ改行してオプションを書くことができます. 同じオプションが本文とチャンクヘッダ (```` ```{} ```` の内側) の両方で指定された場合, 前者が後者を上書きします. チャンク内では `<タグ>: <値>` のような YAML 式の記法でオプションを書くこともできます. 通常は, 1行毎に1つづつオプションを書かねばなりません. 例えば以下のように.

````
```{r}
#| echo: false
#| fig.width: 10
```
````

YAML 記法を選択した場合, 生の R の式ではなく YAML の値として有効なものを書かねばなりません.


以下では `オプション`: (`デフォルト値`; 値の型) という形式で, **knitr** で使えるチャンクオプションのリストを掲載します.

### コード評価関連 {#evaluate}

-   **`eval`**: (`TRUE`; `logical` または `numeric`).: コードチャンクを評価するかどうか. どの R の評価式を評価するかを選ぶために `numeric` のベクトルを使用することもできます. 例: `eval=c(1, 3, 4)` ならば1つ目, 3つ目, そして4つ目の評価式を評価し, `eval = -(4:5)` は 4, 5つ目の式以外の全てを評価します.

### テキストの出力関連 {#text-output}

-   **`echo`**: (`TRUE`; `logical` または `numeric`).: 出力される文書にソースコードを表示するかどうか. 「表示」「隠す」に対応する `TRUE`/`FALSE` に加え, どの R の評価式を表示するかを選ぶために `numeric` のベクトルを使用することもできます. 例: `echo=2:3` は 2, 3番めの評価式を表示し,  `echo = -4` は4番目だけを非表示にします.
- `results`: (`'markup'`; `character`) 実行結果のテキストの部分をどう表示するかを制御します. このオプションは通常のテキスト出力にのみ影響することに注意してください (警告・メッセージ・エラーは適用範囲外です). 取りうる値は次のとおりです.
    - `markup`: 出力の文書フォーマットに応じて適切な環境でテキスト出力をマークアップします. 例えば R Markdown ならば `"[1] 1 2 3"` が **knitr** によって以下のように加工されます. この場合, `results='markup'` は囲み (```` ``` ````) 付きのコードブロックとして出力されることを意味します.
    
    ````
    ```
    [1] 1 2 3
    ```
    ````
    - `asis`: テキスト出力を「そのまま」書き出します. つまり, 生の結果テキストをマークアップ一切なしでそのまま文書に書き出します.
    
    ````
    ```{r, results='asis'}
    cat("I'm raw **Markdown** content.\n")
    ```
    ````
    - **`hold`**: チャンクと flush の全てのテキスト出力をチャンクの末尾に固定します.
    - **`hide`** (または `FALSE`): テキスト出力を表示しません.
-   `collapse`: (`FALSE`; `logical`) 可能であれば, ソースと出力をつなげて1つのブロックにするかどうかです (デフォルトではソースと出力はそれぞれ独立したブロックです). このオプションは Markdown 文書でのみ有効です.
-   **`warning`**: (`TRUE`; `logical`).: 警告文 (`warning()` で出力されるテキスト) を保存するかどうかです. `FALSE` の場合, 全ての警告文は文書に出力されず, 代わりにコンソールに書き出されます. 警告文の一部を選ぶインデックスとして, `numeric` 型のベクトルを指定することもできます. この場合のインデックスの数値は「何番目の警告文を表示するか」を参照する (例: `3` はこのチャンクから投げられた3番目の警告文を意味します) ものであって, 「何番目の R コードの警告文の出力を許可するか」ではないことに注意してください.
-   **`error`**: (`TRUE`; `logical`).: エラー文 (`stop()` で出力される文です) を保持するかどうかです. デフォルトの `TRUE` では, **コード評価はエラーが出ても停止しません!** エラー時点で停止させたい場合はこのオプションを `FALSE` に指定してください. **R Markdown ではこのデフォルト値は `FALSE` に変更されていることに注意**してください. チャンクオプションに `include=FALSE` がある場合, 起こりうるエラーを見落とさないように,  **knitr** はエラー時点で停止するようになります.
-   **`message`**: (`TRUE`; `logical`).: `message()` が出力するメッセージ文を (`warning` オプションと同様に) 表示するかどうかです.
-   **`include`**: (`TRUE`; `logical`).: 出力する文書にチャンクの出力を含めるかどうかです. `FALSE` ならばなにも書き出されませんが, コードの評価はなされ, チャンク内にプロット命令があるのならグラフのファイルも生成されます. よってこの図をそれ以降で任意に挿入することもできます.
-   **`strip.white`**: (`TRUE`; `logical`).: 出力時にソースコードの冒頭と末尾から空白行を除去するかどうかです.
-   **`class.output`**: (`NULL`; `character`).: テキストの出力ブロックに追加するクラス名のベクトル. このオプションは R Markdown で HTML を出力する際にのみ機能します. 例えば `class.output = c('foo', 'bar')` はテキスト出力が `<pre class="foo bar"></pre>` で囲まれたブロックとして書き出されます.
-   **`class.message`**/**`class.warning`**/**`class.error`**: (`NULL`; `character`)・: `class.output` と同様に, R Markdown においてそれぞれ メッセージ文, 警告文, エラー文のブロックに対してクラス名を与えます. `class.source` もまた同様にソースコードのブロックに対して適用されます. "[コードの装飾](#code-decoration)" のセクションを参照してください.
-   **`attr.output`**/**`attr.message`**/**`attr.warning`**/**`attr.error`**: (`NULL`;
    `character`).: 上記の `class.*` オプション群と同様に, Pandoc に対してコードブロックの属性を指定します. つまり `class.*` は `attr.*` の特殊ケースです. 例: `class.source = 'numberLines'` は `attr.source = '.numberLines'` と等価ですが, `attr.source` は任意の値を取ることができます. 例えば, `attr.source = c('.numberLines', 'startFrom="11"')`.
-   **`render`**: (`knitr::knit_print`; `function(x, options, ...)`).: チャンクで表示される値に対して適用する関数です. 関数の第1引数には (`x`) はチャンクの各評価式が評価された結果が与えられます. このチャンクのチャンクオプションがリストとして第二引数 `opstions` に与えられます. この関数は文字列を返すことを想定しています. 詳細は package vignette (`vignette('knit_print', package = 'knitr')`) を参照してください.
-   **`split`**: (`FALSE`; `logical`).: 出力ブロックを分割したファイルに書き出すかどうか. LaTeX ならば `\input{}` で読み込み, HTML ならば `<iframe></iframe>` タグで読み込まれます. このオプションは `.Rnw`, `.Rtex` そして `.Rhtml` でのみ機能します.

### コードの装飾関連 {#code-decoration}

- `tidy`: (`FALSE`) R コードを整形するかどうかです. 他の有効な値は次のとおりです.
    - `TRUE` (`tidy = 'formatR'` と等価です): 整形のために `formatR::tidy_source()` を呼び出します.
    - `'styler'`: コード整形のために `styler::style_text()` を呼び出します.
    - 整形されたコードを返す, `function(code, ...) {}` という形式の任意の関数.
    - 整形が失敗した場合, 元の R コードは変更されません (警告は表示されます).
- `tidy.opts`: (`NULL`; `list`) `tidy` オプションで指定した関数へのオプション引数のリストです. 例えば `tidy.opts = list(blank = FALSE, width.cutoff = 60)` は `tidy = 'formatR'` に対して空白行を削除し各行が60文字におさまるように改行しようとします.
- `prompt`: (`FALSE`; `logical`) R コードにプロンプト記号 (`>` など) を付けるかどうかです.  `?base::options` ヘルプページの `prompt` と `continue` を参照してください. プロンプト記号の追加は, 読者がコードをコピーするのを難しくさせるため, `prompt=FALSE` のほうが良い選択であることに留意してください. エンジンが `R` 以外の場合, このオプションはうまく機能しないことがあります (issue [\#1274](https://github.com/yihui/knitr/issues/1274)).
-   **`comment`**: (`'##'`; `character`).: テキスト出力の各行の先頭文字です. デフォルトでは, コメントアウトできるよう `##` となっているので, 読者は文書から任意の範囲をそのままコピーしても出力部分は無視されるのでそのまま実行することができます. `comment = ''` を指定することで, デフォルトの `##` は除去されます.
-   **`highlight`**: (`TRUE`; `logical`).: ソースコードをシンタックスハイライトするかどうかです^[訳注: R Markdown ではさらに, YAML フロントマターで適用するハイライトのテーマ名を指定できます].
-   **`class.source`**: (`NULL`; `character`).: 出力された文書のソースコードブロックのクラス名です. 出力ブロックに対して機能する `class.output` をはじめとする `class.*` シリーズと同様です.
-   **`attr.source`**: (`NULL`; `character`).: ソースコードブロックの属性です. `attr.output` などの `attr.*` シリーズと同様です.
- -   `lang`: (`NULL`; `character`) コードチャンクの言語名です. デフォルトでは言語名はエンジン名と同じです. 例: `r`. このオプションは主に Markdown ベースの文書出力でシンタックスハイライトするためのものです.
-   `size`: (`'normalsize'`; `character`) `.Rnw` 使用時のチャンクサイズのフォントサイズです. 指定可能なサイズは  [overleaf のヘルプページ (英語)](https://www.overleaf.com/learn/latex/Font_sizes,_families,_and_styles) を参照してください^[訳注: `\normalsize`, `\Large`, `\LARGE` など LaTeX で指定できるフォントサイズを表すマクロのことを指しています].
-   **`background`**: (`'#F7F7F7'`; `character`).:  `.Rnw` 使用時のチャンクブロックの背景色です^[訳注: R Markdown では背景色は CSS や `class.output` などで設定する必要があります. 詳細は R Markdown Cookbook などを参照してください].
-   **`indent`**: (`character`).: チャンクの出力で各行に追加する文字です. 典型的には空白と同義です. このオプションは読み込み専用を想定しており, 値は **knitr** が文書を読み込む際に設定されます. 例えば以下のチャンクでは, `indent` は空白文字2個です^[訳注: R Markdown の場合は **knitr** 以外の中間処理があるため, 必ずしもこのルールを守りません].
    
    ````
    ```{r indent-example, echo=2}
    set.seed(42)
    rnorm(10)
    ```
    ````
### キャッシュ関連 {#options-cache}

-   **`cache`**: (`FALSE`; `logical`).: コードチャンクのキャッシュを取るかどうかです. 初回の実行またはキャッシュが存在しない場合は通常通り実行され, 結果がデータセットが保存され (`.rdb`, `.rdx` ファイルなど), それ以降でコードチャンクが評価されることがあれば, 以前保存されたこれらのファイルからこのチャンクの結果を読み出します. ファイル名がチャンクラベルと R コードの MD5 ハッシュ値で一致する必要があることに注意してください. つまりチャンクになんらかの変更がある度に異なる MD5 ハッシュ値が生成されるため, キャッシュはその度に無効になります. 詳細は [キャッシュの解説](#cache) を参考にしてください.
-   **`cache.path`**: (`'cache/'`; `character`).: 生成したキャッシュファイルの保存場所を指定します. R Markdown ではデフォルトでは入力ファイルの名前に基づきます. 例えば `INPUT.Rmd` の `FOO` というラベルのチャンクのキャッシュは `INPUT_cache/FOO_*.*` というファイルパスに保存されます.
-   **`cache.vars`**: (`NULL`; `character`).: キャッシュデータベースに保存される変数名のベクトルを指定します. デフォルトではチャンクで作られた全ての変数が識別され保存されますが, 変数名の自動検出はロバストではないかもしれませんし,  保存したい変数を選別したい場合もあるかもしれないので, 保存したい変数を手動選択することもできます.
-   **`cache.globals`**: (`NULL`; `character`).: このチャンクで作成されない変数の名前のベクトルを指定します. このオプションは主に `autodep = TRUE` オプションをより正確に動作させたいときに使います. チャンク `B` で使われているグローバル変数が チャンク `A` のローカル変数として使われているときなど. グローバル変数の自動検出に失敗した際に使う場合, こにオプションを使って手動でグローバル変数の名前を指定してください (具体例として issue [\#1403](https://github.com/yihui/knitr/issues/1403) を参照してください). さらに, `cache.globals = FALSE` は, 変数がグローバルかローカルかにかかわらず, コードチャンク内のすべての変数を検出することを意味します. 
-   **`cache.lazy`**: (`TRUE`; logical).: 遅延読み込み `lazyLoad()` を使うか, 直接 `load()` でオブジェクトを読み込むかを指定します. 非常に大きなオブジェクトに対しては, 遅延読み込みは機能しないかもしれません. よってこの場合は `cache.lazy = FALSE` が望ましいかもしれません (issue [\#572](https://github.com/yihui/knitr/issues/572) を参照してください).
-   **`cache.comments`**: (`NULL`; `logical`).: `FALSE` の場合, R コードチャンク内のコメントを書き換えてもキャッシュが無効になりません.
-   **`cache.rebuild`**: (`FALSE`; `logical`).: `TRUE` の場合, キャッシュが有効であってもチャンクのコードの再評価を行います. このオプションはキャッシュの無効化の条件を指定したいときに有用です. 例えば `cache.rebuild = !file.exists("some-file")` とすれば `some-file` が存在しないときにチャンクが評価されキャッシュが再構成されます (issue [\#238](https://github.com/yihui/knitr/issues/238) を参照).
-   **`dependson`**: (`NULL`; `character` または `numeric`).: このチャンクが依存しているチャンクのラベル名を文字ベクトルで指定します. このオプションはキャッシュされたチャンクでのみ適用されます. キャッシュされたチャンク内のオブジェクトは, 他のキャッシュされたチャンクに依存しているかもしれず, 他のチャンクの変更に合わせてこのチャンクも更新する必要があるかもしれません.
    - `dependson` に `numeric` ベクトルを与えた場合, それはチャンクラベルのインデックスを意味します. 例えば `dependson = 1` ならばこの文書の1番目のチャンクに依存することを意味し, `dependson = c(-1, -2)` は直前の2つのチャンクを意味します (負のインデックスは現在のチャンクからの相対的な位置を表します).
    - `opts_chunk$set()` によってグローバルにチャンクオプションを設定した場合, このオプションは機能しません. ローカルなチャンクオプションとして設定しなければなりません.
-   **`autodep`**: (`FALSE`; `logical`).: グローバル変数を検出することでチャンク間の依存関係を分析するかどうかを指定します (あまり信頼できません). よって, `dependson` を明示的に指定する必要はありません.

### グラフ関連 {#plots}

-   **`fig.path`**: (`'figure/'`; `character`).: 図の保存ファイルパスを生成する際の接尾語. `fig.path` とチャンクラベルを連結したものがフルパスになります. `figure/prefix-` のようにディレクトリ名が含まれて, それが存在しない場合はディレクトリが作成されます.
-   **`fig.keep`**: (`'high'`; `character`).: グラフをどのように保存するかです. 可能な値は次のとおりです.
    -   `high`: 高水準プロットのみ保存 (低水準の変更は全て高水準プロットに統合されます).
    -   `none`: 全て破棄します.
    -   `all`: 全てのプロットを保存します (低水準プロットでの変更は新しいグラフとして保存されます).
    -   `first`: 最初のプロットのみ保存します.
    -   `last`: 最後のプロットのみ保存します.
    -   数値ベクトルを指定した場合, その値は保存する低水準プロットのインデックスとなります.
    低水準プロットとは `lines()` や `points()` などの関数によるグラフ描画のことです. `fig.keep` についてより理解するには次のようなチャンクを考えてください. 通常はこれで2つのグラフを出力します (`fig.keep = 'high'` を指定したので).  `fig.keep = 'none'` としたなら, いかなるグラフも保存されません. `fig.keep = 'all'` ならば, 4 つのグラフとして保存されます. `fig.keep = 'first'` ならば `plot(1)` によって作成されたグラフが保存されます. `fig.keep = 'last'`, なら, 最後の10本の垂線を描画したグラフが保存されます.

    
    ```{.r .numberLines .lineAnchors}
    plot(1) # 高水準プロット
    abline(0, 1) # 低水準の作図
    plot(rnorm(10)) # 高水準プロット
    # ループ内での複数の低水準作図 (R 評価式としては1つ)
    for (i in 1:10) {
      abline(v = i, lty = 2)
    }
    ```

-   **`fig.show`**: (`'asis'`; `character`).: グラフをどのように表示し, 配置するかです. 可能な値は次のとおりです.
    -   `asis`: グラフが生成された場所にそのまま出力します (R ターミナルで実行した場合とおなじように).
    -   `hold`: 全てのグラフをまとめてチャンクの最後に出力します.
    -   `animate`: チャンクに複数のグラフがある場合, 連結して1つのアニメーションにします.
    -   `hide`: グラフをファイルに保存しますが, 出力時は隠します.
-   **`dev`**: (LaTeX の場合は `'pdf'`^[訳注: `pdf` は日本語表示に向いていないため, `cairo_pdf` などを利用することをおすすめします],  HTML/Markdown の場合は `'png'`; `character`).: グラフをファイルに保存する際のグラフィックデバイスです. base R および, **`Cairo`**, **`cairoDevice`**, **`svglite`**, **`ragg`**, **tikzDevice** パッケージの提供するデバイスに対応しています. デバイスの例: `pdf`, `png`, `svg`, `jpeg`, `tiff`,
    `cairo_pdf`, `CairoJPEG`, `CairoPNG`, `Cairo_pdf`, `Cairo_png`, `svglite`,
    `ragg_png`, `tikz`, など. 有効なデバイスの一覧は `names(knitr:::auto_exts)` を参照してください. また, `function(filename, width, height)` という引数を定義した関数名を文字列で与えることでも指定できます. 画像サイズの単位は **常にインチ**です. ビットマップであってもインチで指定したものがピクセルに変換されます.

チャンクオプション `dev`, `fig.ext`, `fig.width`, `fig.height`, `dpi` はベクトルを与えられます (長さが足りない場合は再利用されます). 例えば `dev = c('pdf', 'png')` は1つのグラフに対して 1つづつ `PDF` と `PNG` ファイルを作成します.

-   **`dev.args`**: (`NULL`; `list`).: グラフィックデバイスに与える追加の引数です. 例えば `dev.args = list(bg = 'yellow', pointsize = 10)` を `dev = 'png'` に与えられます. 特定のデバイスに依存するオプション (詳細はそれぞれのデバイスのドキュメントを確認してくだささい). `dev` に複数のデバイスが指定されている場合は `dev.args` を引数のリストをさらにリストでくくることになるでしょう. それぞれの引数リストが対応するデバイスに与えられます. 例: `dev = c('pdf', 'tiff'), dev.args = list(pdf = list(colormodel = 'cmyk', useDingats = TRUE), tiff = list(compression = 'lzw'))`.
-   **`fig.ext`**: (`NULL`; `character`).: 出力するグラフのファイル拡張子です. `NULL` ならばグラフィックデバイスに応じて自動決定されます. 詳細は `knitr:::auto_exts` を確認してください.
-   **`dpi`**: (`72`; `numeric`). ビットマップデバイスに対する DPI (インチ毎ドット, `dpi * inches = pixels`) です.
-   **`fig.width`**, **`fig.height`**: (いずれも `7`; `numeric`).: グラフの幅と高さです. 単位はインチです. グラフィックデバイスに与えられます.
-   **`fig.asp`**: (`NULL`; `numeric`).: グラフのアスペクト比, つまり 高さ/幅 の比です. `fig.asp` が指定された場合, 高さ (`fig.height`) は `fig.width * fig.asp` によって自動設定されます.
-   **`fig.dim`**: (`NULL`; `numeric`).: `fig.width` と `fig.height` を指定する長さ2の数値のベクトルです. 例: `fig.dim = c(5, 7)` は `fig.width = 5, fig.height = 7` の省略形です. `fig.asp` と `fig.dim` が指定された場合, `fig.asp` は無視されます (警告文が出力されます).
-   **`out.width`**, **`out.height`**: (`NULL`; `character`).: 出力時の画像の幅と高さです. 実体としての幅と高さである `fig.width`
    と `fig.height` とは異なります. つまりグラフは文書に表示される際にスケールが調整されます. 出力フォーマットに応じて, これら2つのオプションはそれぞれ特殊な値を取ることができます. 例えば LaTeX ならば `.8\\linewidth`, `3in`, `8cm` などと指定でき, HTML ならば `300px` と指定できます. `.Rnw` ならば `out.width` のデフォルト値は `\\maxwidth` に変更され, その値は [framed のページ](#framed) で書いたように定義されます. 例えば `'40%'` のようにパーセンテージで指定もでき, これは LaTeX では `0.4\linewidth` に置き換えられます.
-   **`out.extra`**: (`NULL`; `character`).: 図の表示に関するその他のオプションです. LaTeX で出力する場合は `\includegraphics[]` に挿入される任意の文字に対応し (例: `out.extra = 'angle=90'` ならば図の90度回転), HTML なら `<img />` に挿入されます (例: `out.extra = 'style="border:5px solid orange;"'`).
-   **`fig.retina`**: (`1`; `numeric`).: このオプションは HTML で出力する際にのみ適用されます. [Retina ディスプレイ](https://ja.wikipedia.org/wiki/Retina%E3%83%87%E3%82%A3%E3%82%B9%E3%83%97%E3%83%AC%E3%82%A4)  に対して画像サイズを調整する比率 (多くの場合は2を指定します) です. チャンクオプションの  `dpi` を `dpi * fig.retina` で, `out.width` を `fig.width * dpi / fig.retina` で計算します. 例えば `fig.retina = 2` なら, 画像の物理サイズが2倍となり, その表示サイズは半分になります.
-   **`resize.width`**, **`resize.height`**: (`NULL`; `character`).: LaTeX で出力する際に `\resizebox{}{}` コマンドで使われます. これら2つのオプションは Tikz グラフィックスをリサイズしたい場合のみ必要になります. それ以外に通常使うことはありません. しかし **tikzDevice** の開発者によれば, 他の箇所のテキストとの一貫性のため, Tikz グラフィックスはリサイズを想定していません. 値の1つでも `NULL` ならば, `!` が使用されます (この意味がわからない方は **graphicx** のドキュメントを読んでください).
-   **`fig.align`**: (`'default'`; `character`).: 出力時の画像の位置揃え (アラインメント) です. 可能な値は `default`, `left`, `right`, `center` です. `default` は位置について特に何も調整しません.
-   **`fig.link`**: (`NULL`; `character`) 画像に与えるリンク.
-   **`fig.env`**: (`'figure'`; `character`).: 画像に使われる LaTeX 環境. 例えば `fig.env = 'marginfigure'` ならば `\begin{marginfigure}` で囲まれます. このオプションの使用は `fig.cap` が指定されいることが条件です.
-   **`fig.cap`**: (`NULL`; `character`).: 図のキャプションです.
-   **`fig.alt`**: (`NULL`; `character`) HTML 出力時の図の `<img>` タグの `alt` 属性に使う代替テキストです. デフォルトでは, 代替テキストが与えられた場合チャンクオプション `fig.cap` には代替テキストが使われます.
-   **`fig.scap`**: (`NULL`; `character`).: 図の短縮キャプションです. 出力が LaTeX の場合のみ意味をなします. 短縮キャプションは `\caption[]` コマンドに挿入され, 大抵の場合は PDF 出力時の「図一覧」で表示される見出しとして使われます.
-   **`fig.lp`**: (`'fig:'`; `character`).; 図の相互参照に使われるラベル^[訳注: チャンクラベルと混同しないでください]の接頭語で, `\label{}` コマンドに挿入されます. 実際のラベルはこの接頭語とチャンクラベルを連結して作られます. 例えば図のラベルが  ```` ```{r, foo-plot} ```` will be ならば, デフォルトでは図のラベルは `fig:foo-plot` になります.
-   **`fig.pos`**: (`''`; `character`).: LaTeX の `\begin{figure}[]` に使われる, 画像の位置調整オプション^[訳注: LaTeX では通常は図の位置は調整されますが, `fig.pos='H'` ならばその位置で固定されます]を指定します. 
-   **`fig.subcap`**: (`NULL`).: subfigures のためのキャプションです. 複数のグラフが1つのチャンクにあり, かつ `fig.subcap` も `fig.cap` is `NULL` である場合, `\subfloat{}` が個別の画像の表示に使われます (この場合はプリアンブルに `\usepackage{subfig}` と書く必要があります). 具体例は [067-graphics-options.Rnw](https://github.com/yihui/knitr-examples/blob/master/067-graphics-options.Rnw) を参照してください.
-   **`fig.ncol`**: (`NULL`; `integer`). subfigure の数です. 例えば[この issue](https://github.com/yihui/knitr/issues/1327#issuecomment-346242532) を見てください (`fig.ncol` も `fig.sep` も LaTeX でのみ機能します).
-   **`fig.sep`**: (`NULL`; `character`).: subfigures どうしの間に挿入されるセパレータを指定する文字ベクトルです. `fig.ncol` が指定された場合, デフォルトでは `fig.sep` に N 個ごとに  `\newline` が挿入されます (`N` は列の数です). 例えば `fig.ncol = 2` ならばデフォルトは `fig.sep = c('', '', '\\newline', '', '', '\\newline', '', ...)` となります.
-   **`fig.process`**: (`NULL`; `function`).: 画像ファイルに対する後処理の関数です. 関数は画像のファイルパスを引数として, 挿入したい新しい画像のファイルを返すものであるべきです. 関数に `options` 引数がある場合, この引数にチャンクオプションのリストが与えられます.
-   **`fig.showtext`**: (`NULL`; `logical`).: `TRUE` ならばグラフの描画前に `showtext::showtext_begin()` が呼ばれます. 詳細は [**`showtext`**](http://cran.rstudio.com/package=showtext) パッケージのドキュメントを参照してください^[訳注: `showtext` は手っ取り早く日本語を表示できますが, いくつかの制約があります. 詳細は『[おまえはもうRのグラフの日本語表示に悩まない (各OS対応)](https://ill-identified.hatenablog.com/entry/2020/10/03/200618)』『[Rでのフォントの扱い](https://oku.edu.mie-u.ac.jp/~okumura/stat/font.html)』などを見てください.].
-   **`external`**: (`TRUE`; `logical`).: tikz グラフィックの処理 (PDF 生成時のコンパイル前の処理) を外部化するかどうかです. **tikzDevice** パッケージの `tikz()` デバイスを使う場合 (つまり `dev='tikz'` を指定したとき) のみ使用され, コンパイル時間を短縮することが可能です.
-   **`sanitize`**: (`FALSE`; `character`). tikz グラフィックでサニタイズ (ここでは, LaTeXで特殊な意味を持つ文字のエスケープ処理) するかどうかです. 詳細は **tikzDevice** パッケージのドキュメントを参照してください.

さらにこの他に, ユーザーが使用することを想定していない隠しオプションが2つあります. `fig.cur` (複数の図表がある場合の, 現在の図番号/インデックス) と `fig.num` (チャンク内の図の合計数) です. これら2つのオプションは **knitr** が複数の図そしてアニメーションを処理するためにあります. 場合によっては手動で保存した画像ファイルを使ってアニメーションを書き出す場合などに役に立つかもしれません (使用例として [graphics manual](https://github.com/yihui/knitr/releases/download/doc/knitr-graphics.pdf) を参照してください).

### アニメーション関連 {#animation}

- **`interval`**: (`1`; `numeric`).: アニメーションの1フレームごとの時間 (単位は秒) です.
- **`animation.hook`**: (`knitr::hook_ffmpeg_html`; `function` または `character`). HTML 出力時のアニメーション作成用のフック関数を指定します. デフォルトでは FFmpeg を使って WebM 動画ファイルに変換します.
    - 別のフック関数として [**gifski**](https://cran.r-project.org/package=gifski) パッケージの `knitr::hook_gifski` 関数はGIFアニメーションを作ることができます.
    - このオプションは `'ffmpeg'` や `'gifski'` といった文字列を指定することもできます. これらは対応するフック関数の省略形です. 例: `animation.hook = 'gifski'` は `animation.hook = knitr::hook_gifski` を意味します.
- **`aniopts`**: (`'controls,loop'`; `character`).: アニメーションに対する追加のオプションです. 詳細は LaTeX の [**animate**
        パッケージのドキュメント](http://ctan.org/pkg/animate)を参照してください.
- **`ffmpeg.bitrate`**: (`1M`; `character`).: WebM 動画の質を制御するための FFmpeg の引数 `-b:v` に対応する値を指定できます.
- **`ffmpeg.format`**: (`webm`; `character`).: FFmpeg の出力する動画フォーマットです. つまり, 動画ファイルの拡張子名です.

### コードチャンク関連 {#code-chunk}

- **`code`**: (`NULL`; `character`).: 指定された場合, そのチャンクのコードを上書きします. この機能によって, プログラミング的にコード挿入が可能になります. 例えば `code = readLines('test.R')` とすれば `test.R` の内容を現在のチャンクで実行します.
`file`: (`NULL`; `character`) これが指定された場合, `code` オプションが, チャンクとして読み込まれた外部ファイルの内容で上書きされます. `file = "test.R"` というチャンクオプションは `code = xfun::read_all("test.R")` を指定しているのと同じことを意味します.
- **`ref.label`**: (`NULL`; `character`).: 現在のチャンクのコードに引き継ぐ, 別のチャンクのラベルの文字列ベクトルを指定します (動作例は  [チャンク参照](#reference) を確認してください).

### 子文書関連 {#child-document}

- **`child`**: (`NULL`; `character`).:  親文書に挿入する子文書のファイルパスを示す文字ベクトルを指定します.

### 言語エンジン関連 {#engine}

- **`engine`**: (`'R'`; `character`).: コードチャンクの言語名です. 指定可能な名前は `names(knitr::knit_engines$get())` で確認できます. 例: `python`, `sql`, `julia`, `bash`, `c`, など. `knitr::knit_engines` で他の言語を使うためのセットアップが可能です.
- **`engine.path`**: (`NULL`; `character`).: 実行可能なエンジンのパスを指定します. あなたのお使いのシステムの別の実行ファイルを使用するためのオプションです. 例えば `python` はデフォルトでは `/usr/bin/python` を参照しますが, 他のバージョンを使うため `engine.path = '~/anaconda/bin/python'` などと指定することもできます^[訳注: R Markdown の場合, Python のバージョンは `reticulate` パッケージでも制御できます. むしろそちらをつかったほうが便利だと思われます.]. `engine.path` もまたパスのリストを与えられます. これによってエンジンごとにそれぞれパスを指定することができます. 以下のコードが例です. リストの名前はエンジン名と一致する必要があります.

    
    ```{.r .numberLines .lineAnchors}
    knitr::opts_chunk$set(engine.path = list(
      python = "~/anaconda/bin/python",
      ruby = "/usr/local/bin/ruby"
    ))
    ```

- **`engine.opts`**: (`NULL`; `character`) 言語エンジンに与える追加引数. チャンクの段階ではオプションを文字列またはリストで指定することができます.
    
    ````
    ```{bash, engine.opts='-l'}
    echo $PATH
    ```
    ````
    
    ````
    ```{cat, engine.opts = list(file = "my_custom.css")}
    h2 {
      color: blue;
    }
    ```
    ````
    
    グローバルレベルでは, 要素名に言語名を与えた文字列のリストが使用できます. `engine.path` と同様に, `knitr::opts_chunk$set()` で引数のテンプレートを作ると便利です.
    
    
    ```{.r .numberLines .lineAnchors}
    knitr::opts_chunk$set(engine.opts = list(
      perl = "-Mstrict -Mwarnings",
      bash = "-o errexit"
    ))
    ```
    
    各エンジンはそれぞれ自身の `engine.opts` を持ち, 固有のオプションを定義します. 言語エンジンのドキュメントを調べるべきでしょう. R Markdown クックブックには [`cat`](https://bookdown.org/yihui/rmarkdown-cookbook/eng-cat.html)^[翻訳版: https://gedevan-aleksizde.github.io/rmarkdown-cookbook/eng-cat.html], [`sass`/`scss`](https://bookdown.org/yihui/rmarkdown-cookbook/eng-sass.html)^[翻訳版: https://gedevan-aleksizde.github.io/rmarkdown-cookbook/eng-sass.html]エンジンの例が掲載されています.

### オプションテンプレート関連

-   **`opts.label`**: (`NULL`; `character`).: `knitr::opts_template` でのオプションのラベルです. オプションセットのラベルは `knitr::opts_template` で設定できます (`?knitr::opts_template` を参照してください). このオプションにより, 頻繁に使うチャンクオプションのタイピング労力を削減できます. 

    **訳注**: 例えば次のように, `echo=F` を設定するテンプレート `noecho` をどこかで作成したとします. すると, 以降のチャンクで `opts.label="noecho"` を設定すると `opts_template` で設定した `noecho` のオプションが全て適用されます. もちろん複数のオプションをまとめることもできるので, 定番の設定を使いまわすのが簡単になります.
    
    
    ```{.r .numberLines .lineAnchors}
    knitr::opts_template$set(noecho = list(echo = F))
    ```

### ソースコードの抽出関連

-   **`purl`**: (`TRUE`; `logical`).: ソースドキュメントから `knitr::purl()` でソースコードを取りだす時, このチャンクを含めるか除外するかどうかです.

### その他のチャンクオプション

-   **`R.options`**: (`NULL`; `list`).: コードチャンク内でのローカルな R オプションを指定します. これらは `options()` によってこのコードチャンクの直前に一時的に設定され, 実行後に戻されます.

## パッケージオプション一覧 {#package-options}

パッケージオプションは [`knitr::opts_knit`](#objects) を使用することで変更できます. **`knitr::opts_chunk` と混同しないでください**. 使用例は以下のとおりです.

    
    ```{.r .numberLines .lineAnchors}
    knitr::opts_knit$set(progress = TRUE, verbose = TRUE)
    ```

別の方法として, R の基本関数である `options()` を使ってパッケージオプションを設定する場合は `?knitr::opts_knit` を参照してください.

可能なオプションは次のとおりです.

- **`aliases`**: (`NULL`; `character`).: チャンクオプションのエイリアスを指定する名前付きベクトルです. 例えば `c(h = 'fig.height', w = 'fig.width')` は **knitr** に `h` は `fig.height` `w` は `fig.width` と同じ意味だと認識させます. このオプションは名前の長いチャンクオプションのタイピング労力を削減できます.
- **`base.dir`**: (`NULL`; `character`).: グラフを生成する際のディレクトリの絶対パスです.
- **`base.url`**: (`NULL`; `character`).: HTML ページに掲載する画像のベースURLです.
- **`concordance`**: (`FALSE`; `logical`).: この機能は  RStudio によって実装されている機能で, `.Rnw` でのみ有効です. 入力ファイルの行番号に対応した行番号を出力ファイルに書き出すかどうかを指定します. これにより, 出力から入力の誘導が可能になり, 特に LaTeX のエラー発生時に役に立ちます.
-   **`eval.after`**: (`c('fig.cap', 'fig.alt'; `character`).: オプション名の文字ベクトルを指定します. このオプションはチャンクが評価された**後で**評価され, 他の全てのオプションはチャンクが評価される前に評価されます. 例えば `eval.after = 'fig.cap'` が指定されているときに `fig.cap = paste('p-value is', t.test(x)$p.value)` とすると, `eval.after` にはチャンクの評価後の `x` の値が使用されます.
- **`global.par`**: (`FALSE`; `logical`).: `TRUE` にすると, それ以前のコードチャンクでの `par()` での設定が引き継がれます (もちろん, この設定は R グラフィックスのみで有効です). デフォルトでは  **knitr** はグラフの記録のために新規のグラフィックデバイスを開き, コードの評価後に閉じるため, `par()` による設定はその都度破棄されます.
- **`header`**: (`NULL`; `character`).: 文書の開始前に挿入するテキストを指定します. (例えば, LaTeX ならば `\documentclass{article}` の直後, HTML ならば `<head>` タグの直後). このオプションは LaTeX プリアンブルや HTML ヘッダでコマンドやスタイルの定義をするのに有用です. ドキュメントの開始地点は `knitr::knit_patterns$get('document.begin')` で知ることができます. このオプションは `.Rnw` と `.Rhtml` 限定の機能です^[訳注: R Markdown ではヘッダの設定は YAML フロントマターで行います].
-   `label.prefix`: (`c(table = 'tab:')`; character) ラベルの接頭語を指定します. 現時点では `kable::kable()` によって生成される表のラベルに対する接頭語のみサポートしています.
-  **`latex.options.color`**, **`latex.options.graphicx`**: (`NULL`).: それぞれ LaTeX パッケージの  **color** と **graphicx** に対するオプションを指定します. これらのオプションは `.Rnw` 限定の機能です^[訳注: R Markdown ではこの機能もやはり YAML フロントマターが担当しています].
- **`latex.tilde`** (`NULL`): .Rnw 文書のシンタックスハイライトされたチャンク出力内でのチルダ文字を表す LaTeX コマンドの文字列です (使用例は issue [#1992](https://github.com/yihui/knitr/issues/1992) を見てください).
-   **`out.format`**: (`NULL`; `character`).: 可能な値は `latex`, `sweave`,
    `html`, `markdown`, `jekyll` です. このオプションは入力ファイル名に応じて自動で決定され, 自動設定されるフック関数に影響します. 例えば `?knitr::render_latex` を参考にしてください. このオプションは `knitr::knit()` が実行される**前に**設定する必要があります (文書内で設定しても機能しません).
-   **`progress`**: (`TRUE`; `logical`).: `knitr::knit()` の実行中にプログレスバーを表示するかどうかを指定します.
-   **`root.dir`**: (`NULL`; `character`).: コードチャンク評価時のルートディレクトリを指定します. `NULL` の場合, 入力ファイルと同じ場所が指定されます.
-   **`self.contained`**: (`TRUE`; `logical`).: 出力する文書が自己完結的であるべきかどうかを指定します (`.tex` ファイルにスタイルを書き出すか, `html` に CSS を書き出すか). このオプションは `.Rnw` と `.Rhtml` でのみ機能します^[訳注: R Markdown では出力フォーマット関数に同様のオプションが用意されていることが多いです].
-   **`unnamed.chunk.label`**: (`unnamed-chunk`; `character`).: ラベルを設定していないチャンクのラベルの接頭語を指定します.
-   **`upload.fun`**: (`identity`; `function`).: ファイルパスを引数にとり, ファイルに対して処理を行い出力フォーマットが HTML または Markdown の場合に文字列を返す関数を指定します. 典型的な使い方として, 画像をアップロードしそのリンクを返す関数を指定します. 例えば `knitr::opts_knit$set(upload.fun = knitr::imgur_upload)` でファイルを http://imgur.com にアップロードできます (`?knitr::imgur_upload` を参照してください).
-   **`verbose`**: (`FALSE`; `logical`).: 情報を冗長に詳細するか (例えば各チャンクで実行されたRコードやメッセージログなど), チャンクラベルとオプションのみ表示するかを指定します.

<!--chapter:end:01-options.Rmd-->

<!---
title: Hooks
subtitle: Customizable functions to run before / after a code chunk, tweak the output,
  and manipulate chunk options
date: '2017-02-03'
slug: hooks
show_toc: true
--->

# フック {#hooks}

コードチャンク実行前後のカスタマイズ関数, 出力の調整, チャンクオプションの操作について

オリジナルのページ: https://yihui.org/knitr/hooks/

オリジナルの更新日: 2017-02-03

----

**knitr** の `knit_hooks` オブジェクトはフック (hook) を設定するのに使います. 基本的な使用法は `knit_hooks$set(param = FUN)` です (詳細は \@ref(objects) 章『オブジェクト』参照)で, ここでの  `param` はチャンクオプション名, `FUN` は関数です. フックには2種類あります. チャンクに対するフックと出力に対するフックです. フック関数はどのように設計するかによって異なる形式をとるでしょう.

## チャンクフック

チャンクフックは対応するチャンクオプションの値が `NULL` ではないとき  コードチャンクの前後に実行されます (つまりオプションになんらかの値を設定している限り, 実行されるということです). この関数は以下のように 3 つの引数が定義されるべきです.


```{.r .numberLines .lineAnchors}
foo_hook <- function(before, options, envir) {
  if (before) {
    ## チャンク実行前の処理
  } else {
    ## チャンク実行後の処理
  }
}
```

**knitr** が文書を処理する時, 各チャンクの直前に `foo_hook(before = TRUE)` が (チャンクがキャッシュされていたり評価しないように設定されていない限り) 呼び出され, チャンク直後に `foo_hook(before = FALSE)` が呼び出されます. `options` 引数は現在のチャンクに設定された**オプション** (\@ref(options) 章) です (例えば `options$label` は現在のチャンクのラベルです). `envir` にはコードチャンクが評価させる環境を指定します. 後の2つの引数はチャンクフックのオプション引数です. 例えば, 次のように `small_mar` オプションにフックを設定します. (訳注: マージン調整だけでは違いがわかりづらい, というか R Markdown はデフォルトでマージン調整するので背景色も変更して違いをわかりやすくしました)


```{.r .numberLines .lineAnchors}
knitr::knit_hooks$set(small_mar = function(before, options, envir) {
  if (before) {
    par(mar = c(4, 4, .1, .1)) # 上と右側のマージンを小さく設定
    par(bg = "blue") # 背景色を青にする
  }
})
```

そしてフックに設定した関数はこのように呼び出されます.

````
```{r, myplot, small_mar=TRUE}
 ## small_mar=TRUE は必須ではない: NULL でさえなければフックは適用される
hist(rnorm(100), main = '')
```
````


\begin{center}\includegraphics[width=1\linewidth,height=1\textheight,keepaspectratio]{knitr_files/figure-latex/myplot-1} \end{center}

**knitr** のフックは出力にテキストを挿入することにも使えます. そのため, このタイプのフック関数は文字列を返す必要があります. この機能はフックの能力を大いに広げます. **`rgl`** パッケージを例に取りましょう. **rgl** によって生成された 3D グラフを Markdown または HTML 文書に挿入したい時, このタイプのフック関数を考える事になるでしょう (この例よりも洗練された `hook_rgl()` 関数が **`rgl`** パッケージにあるので参照してください). 


```{.r .numberLines .lineAnchors}
knit_hooks$set(rgl = function(before, options, envir) {
  if (!before) {
    ## チャンクコードが評価された後の処理
    if (rgl.cur() == 0) {
      return()
    } # アクティブなデバイスがないかどうか
    name <- paste0(options$fig.path, options$label, ".png")
    rgl.snapshot(name, fmt = "png")
    return(paste0("![rgl plot](", name, ")\n"))
  }
})
```

そしてコードチャンクはこのようになります.

````
```{r, fancy-rgl, rgl=TRUE}
library(rgl)  # 用例は ?plot3d から
open3d()
x = sort(rnorm(1000)); y = rnorm(1000); z = rnorm(1000) + atan2(x,y)
plot3d(x, y, z, col = rainbow(1000))
```
````

Markdown の場合 `![rgl plot](fancy-rgl.png)` と出力されているでしょう.

要約すると:

1. フックは `knit_hooks` で, `knit_hooks$set(foo = FUN)` という構文で設定されます
2. あるチャンクで `foo` というチャンクオプションが `NULL` 以外の値をとる場合, このフック関数 `FUN` が実行されます.
3. フックはチャンクの直前と直後に実行できます
4. フックによって返される文字列は修正が加えられることなく出力ブロックに書き出されます.

さらなる用例は [045-chunk-hook.md](https://github.com/yihui/knitr-examples/blob/master/045-chunk-hook.md) ([source](https://github.com/yihui/knitr-examples/blob/master/045-chunk-hook.Rmd)) を参照してください.

## 出力フック

出力フックはチャンクからの**生の**出力をカスタマイズし洗練するために使います. 様々な種類の出力に対処するための 8つの出力フック関数が存在します.

- **`source`**: ソースコード
- **`output`**: 通常の R の出力で, 警告文, メッセージ文, エラー文を除くもの (つまり, 通常の R ターミナルで出力されていたものです)
- **`warning`**: `warning()` による警告文
- **`message`**: `message()` によるメッセージ文
- **`error`**: `stop()` によるエラー文 (コードチャンクとインライン R コードの両方に適用されます)
- **`plot`**: 出力されるグラフ
- **`inline`**: インライン R コードの出力
- **`chunk`**: チャンクの全ての出力 (つまりその前のフックにも生み出されたもの)
- **`document`**: 文書全体の出力 (デフォルトでは `base::identity` が適用されます)

これらのフックは全て `function(x, options)` という形式をとります (例外として, `inline` と `document` のフックのみ引数は `x` の1つです), `x` が出力の文字列で, `options` が現在のチャンクのオプションのリストです. 出力フックに関する情報と用例をさらに詳しく知りたい場合は [_R Markdown Cookbook_](https://bookdown.org/yihui/rmarkdown-cookbook/output-hooks.html) を参考にしてください.

以下は `error` フックの用例になります. [R Markdown](https://rmarkdown.rstudio.com) 上で, エラー文に対して追加で整形処理をするフックです.


```{.r .numberLines .lineAnchors}
knitr::knit_hooks$set(error = function(x, options) {
  paste(c(
    '\n\n:::{style="color:Crimson; background-color: SeaShell;"}',
    gsub("^## Error", "**Error**", x),
    ":::"
  ), collapse = "\n")
})
```

このようなチャンクでフックの動作をテストします

````
```{r, error=TRUE}
1 + "a"
```
````


:::{style="color:Crimson; background-color: SeaShell;"}
**Error** in 1 + "a":  二項演算子の引数が数値ではありません

:::

デフォルトではチャンクフックは空ですが, 出力フックはデフォルト設定があり, 以下のようにしてリセットできます.


```{.r .numberLines .lineAnchors}
knitr::knit_hooks$restore()
```

:::{.infobox .caution data-latex="{caution}"}

**訳注**

R Markdown の場合, 基本的な出力フォーマットにもデフォルトでフックが定義されており, 処理内容によっては予期せぬ結果になることがあるため, 単純な上書きや `$restore()` は意図しない動作につながることがあります. 詳細は "R Markdown Cookboox" [Ch. 12](https://bookdown.org/yihui/rmarkdown-cookbook/output-hooks.html)^[翻訳版: https://gedevan-aleksizde.github.io/rmarkdown-cookbook/output-hooks.html] を確認ください.
:::

本パッケージは出力を異なる部品にわけてそれぞれにデフォルトのフックを設定し, さらに LaTeX, HTML, Jekyll といった異なる出力フォーマットごとに用意しています. `render_*()` という一連の関数群は, 例えば `render_latex()`, `redner_html()`, など出力フォーマットごとにそれぞれ異なる, 組み込みの出力フックを提供するためにあります. 出力フックはドキュメント内で設定すべきですが, `knitr::knit()` が文書を処理する前にフックを設定したなら `render_*()`, たとえば `render_markdown()`, `render_html()` を最初に呼び出さなければなりません. `hooks_markdown()` などの `hooks_*()` 関数で, 設定を変えることなくこれらの出力フックにアクセスすることができます.

以降は各フォーマットでの詳細を記します.

### LaTeX: `render_latex()`

出力ファイルタイプが LaTeX の場合, デフォルトのフックはほとんどのチャンク出力を `verbatim` 環境で囲んで出力し, `inline` 出力における数値を指数表記で出力します (詳細は [チャンク出力の制御](#output)のデモを参照してください). `plot`, `chunk` フックはより複雑な処理をしています.

- デフォルトでは `plot` フックは出力の信頼性を維持するため多くの要因に影響されます. 
  - たとえばグラフィックデバイスが `tikz` ならば, `\input{}` コマンドが使われますし^[訳注: tikz の画像は LaTeX ソースコードで記述されるため, `\input{}` でテキストファイルとして読み込む必要があります.], それ以外では通常は `\includegraphics{}` コマンドが使われます.
  - `out.width`, `out.height` オプションに依存して, フックはグラフのサイズをリセットします (たとえば `\includegraphics[width=.8\textwidth]{file}` のように). 1つのチャンクに複数のグラフがある場合, `fig.show='hold'` を設定するとともに, 複数の画像を適切なサイズで横に並べて表示できるように設定できます (たとえば `.45\textwidth`^[訳注: 本文幅の45%] とすれば横に2つのグラフを並べられます). 
  - tikz のグラフは `\input{}` で挿入するため, このやり方は正しくありませんが, チャンクオプション `resize.width` と `resize.height` は複数の tikz グラフを横に並べることができます (`\resizebox{resize.width}{resize.height}{file.tikz}` という書き方によって. もしいずれかのオプションが `NULL` なら `!` で置き換えられます. 詳細は LaTeX パッケージの `graphicx` のドキュメントを参照してください). このフック関数によってユーザーは自動レポート生成の全能力を使いこなせます --- 単一チャンクの複数グラフとグラフのサイズの設定が可能になるだけでなく, base R のグラフィックスや grid 系のグラフィックス (例: **ggplot2**), あるいはグリッド系のグラフを並べて表示することもできます --- この機能がなかったら, R で1つのウィンドウにこういった複数のグラフを1つにまとめるのがどんなに難しいことか考えても見てください^[訳注: 現在は `patchwork` や `cowplot` パッケージなどの登場により, そこまで難しいことではなくなりつつあります].
  - グラフのアラインを決めるために `fig.align` には4つの値が用意され (`default`, `left`, `right`, `center`), 簡単に画像を中央揃えにできます (`fig.align='center'` によって).
- デフォルトの `chunk` フックは主にチャンクの装飾に使われています.
  - LaTeX の `framed` パッケージがユーザーの TeX ソフトウェアパッケージ (TeXLive か MikTeX か他の何か^[訳注: あるいは Yihui 氏による TinyTeX とか]) にインストールされているなら, `chunk` フックはカスタマイズした背景色 (デフォルトでは薄灰色) にした `kframe` 環境に全ての出力を挿入することで, チャンクの視認性を向上させます (他の地の文よりも強調されますが, とても目立つというほどでもないはずです).
  - 最後に, 全ての出力が `knitrout` 環境で囲まれます. この環境はユーザーが LaTeX で再定義できます.

### Sweave: `render_sweave()`

ソースコードを `Sinput` 環境に挿入し, その出力を `Sioutput` 環境に挿入し, そしてチャンク全体を `Schunk` 環境に挿入します. このテーマの使用にはスタイルファイル `Sweave.sty` か, 少なくともこれら3つの環境を定義することが必要です.

### Listings: `render_listings()`

Sweave 同様に,  [`Sweavel.sty`](https://github.com/yihui/knitr/blob/master/inst/misc/Sweavel.sty) が使用されます.

### HTML: `render_html()`

HTML ファイルに書き出すにあたって, フックは出力を自動で調整します. 基本的にチャンクによる出力はクラス付きの `div` レイヤーに挿入されます. たとえば, ソースコードは `<div class="knitr source"></div>`, チャンク全体は `<pre></pre>` に, インラインの出力は `<code class="knitr inline"></code>` に書き出されます^[訳注: R Markdown では最終的に出力されるHTMLはさらに pandoc などの処理を経由しているため, これとは異なります].

### Markdown: `render_markdown()`

ソースコードとその出力はスペース4つでインデントされます. GitHub Flavored Markdown のため, ソースコードは ```` ```r ```` と ```` ``` ```` の間に挿入され,  出力部分は ```` ``` ```` と ```` ``` ```` の間に挿入されます.

### Jekyll: `render_jekyll()`

このサイト^[訳注: オリジナルが掲載されている Yihui 氏のサイトのこと]を構築するために, Jykell 用に特別にいくつかのフックを用意する必要がありました. これらは実際かなり単純なものです. R ソースコードはハイライト環境に挿入し言語を `r` に設定する, 残りの出力部分は `text` 言語に設定したハイライト環境 (ほとんど何もハイライトしない) に挿入するだけです. 現在, グラフは Markdown の構文に従って書き出されます.

### reStructuredText: `render_rst()`

コードは `::` の後に挿入され, スペース4個でインデントされるか, `sourcecode` ディレクティブに挿入されます.

## オプションフック

他のチャンクオプションの値に応じて別のチャンクオプションの値を変えたいとき, `opts_hooks` をつかってそれを実行することがあるかもしれません. オプションフックは対応するチャンクオプションの値が `NULL` 以外であるときに実行されます. たとえば `fig.width` を常に `fig.height` 以上の値に調整することができます.


```{.r .numberLines .lineAnchors}
knitr::opts_hooks$set(fig.width = function(options) {
  if (options$fig.width < options$fig.height) {
    options$fig.width <- options$fig.height
  }
  options
})
```

`fig.width` は `NULL` になることがないため, このフック関数は常にチャンクの直前の, チャンクオプションが確認される前に実行されます. 以下のコードチャンクは上記のフックを設定することで, `fig.width` の実際の値は初期値の `5` の代わりに `6` が適用されます.

````r
```{r fig.width = 5, fig.height = 6}
plot(1:10)
```
````


\begin{center}\includegraphics[width=1\linewidth,height=1\textheight,keepaspectratio]{knitr_files/figure-latex/unnamed-chunk-2-1} \end{center}

訳注: `knit_hooks` 同様に, `opts_hooks` にも `restore()` メソッドが用意されています.



<!--chapter:end:03-hooks.Rmd-->

<!---
title: Examples
subtitle: Source and output of demos
date: '2017-02-03'
layout: example
list_pages: true
--->

# 使用例 {#examples}

ソースと出力のデモ

オリジナルのページ: https://yihui.org/knitr/demos/

オリジナルの更新日: 2017-02-03

----

:::{.infobox .memo data-latex="{memo}"}
**訳注**: 現在は knitr の主な利用場面は R Markdown との併用だと思うので, それらと関係の薄いページは翻訳していません. また, 編集上の問題から, ここで挙げられているページのうち翻訳済みのものは全てナビゲーションバーの「用例」パートでもリンクされています.
:::

Github の [knitr-examples](https://github.com/yihui/knitr-examples) はより豊富なコレクションになっています. このページはむしろドキュメント用途として作られています. 他のユーザーによる [**knitr**のショーケース](#showcase) を見ることもできます.

-  2011-12-03 [(未翻訳)  Minimal examples -  Examples for Rnw, Markdown, HTML and LaTeX](https://yihui.org/knitr/demo/minimal)
-  2011-12-04 [キャッシュ - キャッシュ機能の使用例について](#cache)
-  2011-12-05 [マニュアル - パッケージマニュアルについて](#manual)
-  2011-12-06 [(未翻訳)  LyX -  Using knitr with LyX](https://yihui.org/knitr/demo/lyx)
-  2011-12-07 [(未翻訳)  Code Externalization -  Use an external R script with your document](https://yihui.org/knitr/demo/externalization)
-  2011-12-08 [(未翻訳)  Beamer -  Using knitr in beamer slides](https://yihui.org/knitr/demo/beamer)
-  2011-12-09 [グラフィックス - **knitr** におけるグラフィックスの力について](#graphics)
-  2011-12-10 [`Listings` - `listings` と `knitr` の併用](#listings)
-  2012-01-14 [チャンク参照/マクロ - チャンクの再利用方法](#reference)
-  2012-01-18 [(未翻訳)  Child documents -  Input child files into the main document](https://yihui.org/knitr/demo/child)
-  2012-01-22 [(未翻訳)  Package vignettes -  How to build package vignettes with knitr](https://yihui.org/knitr/demo/vignette)
-  2012-01-25 [チャンク出力の制御 - チャンクの6種類の出力とインライン出力を操作する](#output)
-  2012-01-26 [(未翻訳)  Quick reporting -  Build a report based on an R script](https://yihui.org/knitr/demo/stitch)
-  2012-02-01 [(未翻訳)  Org-mode -  Use knitr in Org-mode](https://yihui.org/knitr/demo/org)
-  2012-02-02 [(未翻訳)  RStudio -  knitr support in RStudio](https://yihui.org/knitr/demo/rstudio)
-  2012-02-11 [(未翻訳)  Pretty printing -  Print highlighted source code of a function](https://yihui.org/knitr/demo/pretty)
-  2012-02-12 [(未翻訳)  Upload images -  Publish images from chunks in the web](https://yihui.org/knitr/demo/upload)
-  2012-02-24 [(未翻訳)  Sweave -  Transition from Sweave to knitr](https://yihui.org/knitr/demo/sweave)
-  2012-02-27 [(未翻訳)  Eclipse -  Configure Eclipse to work with knitr](https://yihui.org/knitr/demo/eclipse)
-  2012-02-29 [`framed` パッケージ - **knitr** における LaTeX のデフォルトスタイル](#framed)
-  2012-03-16 [`knitr` のエディタ - Emacs, TeX Maker, TeXShop, WinEdt, そして TextMate などについて](#editors)
-  2012-05-01 [(未翻訳)  HTML5 slides -  making HTML5 slides with pandoc and knitr](https://yihui.org/knitr/demo/slides)
-  2012-05-04 [(未翻訳)  Language engines -  Use other languages in knitr](https://yihui.org/knitr/demo/engines)
-  2012-11-09 [(未翻訳)  JavaScript -  Combine R and JS applications like D3](https://yihui.org/knitr/demo/javascript)
-  2013-02-10 [(未翻訳)  WordPress -  Publish blog posts from R + knitr to WordPress](https://yihui.org/knitr/demo/wordpress)
-  2013-03-06 [(未翻訳)  Pandoc -  Convert Markdown to other formats via Pandoc](https://yihui.org/knitr/demo/pandoc)
-  2013-03-11 [`knitr` のショーケース - ユーザーたちによる使用例](#showcase)


<!--chapter:end:04-examples.Rmd-->

<!---
title: Frequently Asked Questions
date: '2017-02-17'
--->

# よくある質問 (FAQ) {#FAQs}

オリジナルのページ: https://yihui.org/knitr/faq/

オリジナルの更新日: 2017-02-17

-----

この FAQ は [issues](https://github.com/yihui/knitr/issues) や私 (Yihui) のブログやEメールに届いた質問などを蓄積したものです. 個人的な考えとして, 私は FAQ という概念の大ファンでもありませんし, FAQ はときとしてほとんどバグのようなものであると思っています. ソフトウェアパッケージの作者は, ユーザーがそんな質問を頻繁にする理由を考えるべきです (ユーザーが全員愚かであるから, という考えには賛成できません). 少なくとも私は, 128件もの質問を1つ1つ読み返すほど忍耐強くありませんし, これを読んでいるあなたにも同様のことをさせるつもりはありません.

1. 「**knitr**が動かないんだけど...」
    - まず最初に, あなたの R パッケージ (`update.packages()` を使います) とたぶん R 本体も (ところで, 現在のあなたの[Rのバージョン](https://cran.rstudio.com)は何ですか?) 更新してください. それから動くかどうかを確認してください. もしそれでも動かなかったら,「必要最低限の再現例 (minimal reprex)」のファイルと `library(knitr); sessionInfo()` の実行結果を [issue](https://github.com/yihui/knitr/issues) に投稿してください.
1. 「パッケージのサイトの説明が役に立たないときはどこで質問するのがいいですか?」
    - 何を質問したいかにもよりますが, 以下のような選択肢があります^[訳注: 英語が苦手な場合は, [Stack Oveflow 日本語版](https://ja.stackoverflow.com/), または [R-wakalang](https://r-wakalang.slack.com/messages/general/) などのコミュニティがあります] (特に最初の2つは私もよく巡回しています).
        - (推奨) **[Stack Overflow](http://stackoverflow.com/questions/tagged/knitr)**: 一般的な質問 (より専門的で早い回答がつきます). `r` と `knitr` タグを忘れずにつけてください. 
        - **[Github issues](https://github.com/yihui/knitr/issues)**: バグ報告と機能追加の要望のみにしてください.
        - **[knitr mailing list](https://groups.google.com/group/knitr)** または **[R-help](http://www.r-project.org/mail.html)** のメーリングリスト: 一般的な質問とその回答が, 一般公開されるEメールによってやりとりされます.
        - **私の個人的なEメール**: 本当にプライベートな問題でない限り[非推奨](https://yihui.org/en/2017/08/so-gh-email/)です^[訳注: リンク先の投稿を要約すると, Yihui氏個人で次から次へと来る質問をさばくのは限界があるし, オープンコミュニティは多くの回答者がいて既出の質問に対する答えも共有でき効率的だ, ということです].
        - **Twitter** ([\@xieyihui](http://twitter.com/xieyihui)): 本当に簡単な問題だと確信があるなら.
1. 「**knitr** のソースドキュメントを書くのに最適のエディタソフトは何ですか?」
    - 初心者にとってはたぶん [RStudio](https://www.rstudio.com) が良いです. **knitr** は [LyX](/knitr/demo/lyx/),  [Emacs/ESS](http://ess.r-project.org/), WinEdt, Tinn-R and や[その他多くのエディタ](#editors)でサポートされています.
1. 「`>` とか `+` とかのプロンプト記号はどこへいったんですか? 出力にこれがないと落ち着きません」
    - [私はこれらが意味をなさないと考えている](https://yihui.org/en/2013/01/code-pollution-with-command-prompts/)ので, デフォルトではこれらは除去されています. R の本に `>` や `+` を載せるのを嫌う理由はこのようなものです: 本に書かれた R コードを読む際に, **こいつらは私の精神をめちゃくちゃにひねりちぎろうとし, 私の両眼に流血を欲してきます**. マジで `1 + 1` ではなく `> 1+1` という表記を読むのがお好みな奇特な方は, [チャンクオプション](#options) の章にあるように `prompt` オプションで設定できます^[訳注: Yihui 氏がこれを嫌う理由はここでも簡潔に書かれています. R Markdown においては, それ以外の方法でもコードの装飾をカスタマイズできます. たとえばこの翻訳版でなされているようにコピーボタンや行番号をつけたりできます.].
1. 「ワーキングディレクトリとはなんですか? コードチャンク内でワーキングディレクトリを変更できますか?」
    - あなたはそういうことをしないほうがよいです. ワーキングディレクトリは常に `getwd()` で分かります (出力ファイルは全てここに書き出されます) が, コードチャンクは入力ファイルがあった場所で評価しています. R コード実行中のワーキングディレクトリ変更は一般的に, よくない使い方 (バッドプラクティス) です. 詳しくは issue [#38](https://github.com/yihui/knitr/issues/38) での議論をご覧ください. また, できることなら常に, 絶対パスによるディレクトリ指定も回避するべきです. 代わりに相対パス指定を使ってください. そのようなコードは再現性を損なうからです. ここまで言ってもまだコードチャンク内で `setwd` を使おうというのなら, 新しく設定したワーキングディレクトリは指定したチャンクにのみ適用され, 以降のコードチャンクでは本来のワーキングディレクトリに差し戻されることを留意してください.
1. 「出力された灰色の背景色ボックスが狭すぎます.」
    - それはボックスが狭すぎるからではありません. ボックス幅は現在行の幅が適用されます. つまりあなたの出力のほうが広すぎるのです. ページのマージンを超えるような出力を避けるため, もっと小さな `width` オプションを設定してください (例: `options(width = 60)`, 詳細は[example 038](https://github.com/yihui/knitr-examples/blob/master/038-output-width.Rnw) を参照してください.)
1. 「コードチャンクにリテラルや生のコードを書く方法は? たとえばパースせずにコードチャンクを書きたいです. チュートリアルに便利だと思います.」
    - チャンクヘッダを破壊する必要があります. たとえば ```` ```{r}`r''` ``` ```` のように, チャンクヘッダの前後に空の文字列を加えます(issues [#443](https://github.com/yihui/knitr/issues/443)). あるいはチャンクヘッダに[ゼロ幅スペース](https://ja.wikipedia.org/w/index.php?title=%E3%82%BC%E3%83%AD%E5%B9%85%E3%82%B9%E3%83%9A%E3%83%BC%E3%82%B9&oldid=78849992)を追加します. 詳細は [example 065](https://github.com/yihui/knitr-examples) を見てください.
    - インラインの R コードでは `knitr::inline_expr()`  を使うことになるでしょう (**knitr** ver. 1.8 以降で使用できます). R Markdown で書いている場合は, あるトリックを使えます. `` `r `` **直後に**改行を入れ (実際にやるときは直後にスペースを入れないでください), 二重のバッククオート (バックティック) のペアでインラインの評価式全体を囲みます. 例えば以下のようになります. この挙動の説明に興味があるなら [このページ](https://yihui.org/en/2017/11/knitr-verbatim-code-chunk/) を見てください (訳注: 未翻訳).

**訳注**: ソースコードでの記述

`````
````
```{r, eval=TRUE}`r ''`
1 + 1
```
````
`````

実際に表示されるもの

````
```{r, eval=TRUE}
1 + 1
```
````

インライン式では以下のようになります


```

ここに生のインラインR評価式を出力: `` `r
1+1` ``.
```

ここに生のインラインR評価式を出力: `` `r
1+1` ``.

8. 「何かお役に立てることはありますか?」
    - [paypal](https://paypal.me/YihuiXie) でドネートできますし, たのしい GIF アニメを私に[ツイート](https://twitter.com/xieyihui)したり, Github で **knitr** のリポジトリをフォークしたりコードの改善に貢献したりできます.
    - パッケージや [knitr book](http://amzn.com/1498716962) を引用してください. R で `citation('knitr')` をしてみてください.
9. 「このドキュメントの修正や小さな変更を投稿するにはどうすればいいですか?」
    - R パッケージを修正したい場合は [リポジトリ](https://github.com/yihui/knitr) へ移動し, ツールバー右上の Edit ボタンを押します. それから必要な修正をします. 投稿の概要を書いて, **Propose file change** をクリックして, プルリクエストを投稿してください.
    - このウエブサイト (https://yihui.org/knitr) 上の修正や変更提案は, ページ左側の `Edit this page` を押して, あとは Github の手順に従ってください^[訳注: 原典ではなくこの翻訳版に対する修正提案は https://github.com/Gedevan-Aleksizde/knitr-doc-ja で受け付けています].

<!--chapter:end:06-FAQ.Rmd-->

# (PART) 用例 {-}

<!--chapter:end:20-examples-header.Rmd-->

<!---
title: Editors for knitr
subtitle: Work with Emacs, Texmaker, TeXShop, WinEdt and TextMate, etc
date: '2012-03-16'
slug: editors
--->

# `knitr` のエディタ {-#editors}

Emacs, TeX Maker, TeXShop, WinEdt, そして TextMate などについて

オリジナルの記事: https://yihui.org/knitr/demo/editors/

オリジナルの更新日: 2012-03-16

----

:::{.infobox .tip data-latex="{tip}"}
**訳注**: このページは R Markdown ではなく, Rnw を想定した説明です. R Markdown は基本的に RStudio での編集が最も使いやすいと思われます.
:::

私は以前 [Lyx]($lyx) (未翻訳), [RStudio]($RStudio) (未翻訳) [Emacs Org-mode]($org) (未翻訳), [Ecripse]($ecripse) (未翻訳) について書きました. その他にも, [Texmaker](http://www.xm1math.net/texmaker/) や Windt といったエディタで **knitr** を使うことができます. ポイントは R を呼び出して **knitr** パッケージを読み込み, それから `knit()`または `knit2pdf()` を呼び出すことです.

## Texmaker {-}

`User --> User Commands --> Edit User Commands` から, Rnw 文書を処理するためのカスタムコマンドを定義できます.

\begin{figure}

{\centering \includegraphics[width=1\linewidth,height=1\textheight,keepaspectratio]{ddIBH} 

}

\caption{Texmaker でユーザーコマンドを定義する}(\#fig:demo-def-user-command-texmaker)
\end{figure}

Rの実行ファイルパスが `PATH` 環境変数にない場合, 上記のコマンドに `Rscript.exe` のフルパスを書く必要があります. こんなかんじに:

```bash
"C:/Program Files/R/R-2.14.2/bin/Rscript.exe" -e "knitr::knit2pdf('%.Rnw')"
```

`Rscript.exe` の場所さえ知っていれば, R を開いて `R.home('bin')` を実行すれば見つけられます. そうすればどんな Rnw 文書ファイルに対しても, この  `knitr` コマンドでコンパイルすることができます.

\begin{figure}

{\centering \includegraphics[width=1\linewidth,height=1\textheight,keepaspectratio]{xKoeT} 

}

\caption{Texmaker で knitr コマンドでコンパイルする}(\#fig:demo-compile-texmaker)
\end{figure}

文書をコンパイルするには左向きの矢印をクリックし, PDF を表示するのに右矢印をクリックします. もちろん上記の設定は Windows のものですが, 他のシステムでも同じ要領です. `Rscript.exe` を `Rscript` に置き換えてください (実際は Windows 環境でも `Rscript` が使えます).

## TeXStudio {-}

一例として, 基本的には Texmaker と同様に設定することができます (Henrik Nyhus と [Paul J. Hurtado](https://twitter.com/MathBioPaul/status/691446297304272897) に感謝)

\begin{figure}

{\centering \includegraphics[width=1\linewidth,height=1\textheight,keepaspectratio]{VFcvTUB} 

}

\caption{TeXStudio での knitr}(\#fig:demo-editor-texstudio)
\end{figure}

上級者向けオプション (左下) を解放すれば, 下部に `Commands ($PATH)` が現れるでしょう. ここに `R` フォルダのパスを入力できます (例: 引用符なしで `C:\Program Files\R\R-3.3.2\bin\x64`). それからユーザーコマンド (そして他のコマンドからも呼び出せる) に `Rscript.exe -e "knitr::knit2pdf('%.Rnw')"` とするだけです.

いつでもこのコマンドを実行できます (ホットキーは Alt-Shift-F1) し, サイレントモードでも knitr で PDF を生成できます. F7 キーでいつでも表示されている PDF を更新できます. 代わりの方法として, コマンドの末尾に `| txs:///view-pdf` を追加する方法があります. これはあるコマンドの実行後に実行する別のコマンドを分けるパイプ記号です. よって基本的に F7 を押すだけで事足りるようになります.

しかし, もし BibLaTeX のような文献引用パッケージを使うならば, まだかなり非効率です. コンパイルの前に bib ファイルの処理を手動で行う必要があり, よって最低でも2回手動でコンパイルする必要があり (そしてあなたは何回必要か正確に分からないでしょう). その間に不要なビューアの呼び出しが発生します. TeXstudio は文献処理ツール起動したりコンパイルしたり, それらを繰り返すのに優れていますが, ビルドと閲覧 (F5) を何度も押す度にこれが必要になり,  そこでデフォルトのコンパイラを `Rscript.exe -e "knitr::knit2pdf('%.Rnw')"` で置き換えることで **knitr** を使用し続けられるわけです (`R` フォルダを `$PATH` に設定している前提です). その隣の "Repeat contained compilation commands" が押されたままであることを確認してください.

もちろん, これならば少なくとも F5 (または F6) を押すだけで TeXstudio デフォルトの標準の　LaTeX コンパイラの代わりに `knitr` が実行されます.  この方法のおそらく唯一の欠点は, ほとんどの人にとって LaTeX コードのデバッグがいっそう困難になることです. ほとんどのエラーが「`texify.exe` "had status 1"」という曖昧な警告に置き換わってしまうことです (ユーザーコマンド経由で **knitr** を実行するのではこれは改善できません). そのような状況でもログファイルも時には有用ですが, ログファイルは `.Rnw` ファイルではなく  **knitr** の生成した `.tex` を参照するので,   このファイルを開いてどこがおかしいのかを見つけようとする必要があります.

しかし物事の全体で見れば, TeXstudio ならば 他のあらゆる LaTeX 書き込み機能の恩恵を得つつ `knitr`  を使うことができます. その上, 不便な回避策が必要な RStudio と比較すれば, `BibLaTeX-Chicago` のような `Biber` ベースの文献引用パッケージの使用もはるかに簡単になります.

## WinEdt {-}

WinEdt の[R-Sweave](http://www.winedt.org/Config/modes/R-Sweave.php) モードは現在 **knitr** をサポートしています. 自分自身で WinEdt の設定をしたいなら, よくお読みください.

以下の手引書は [Phil Chalmers](https://github.com/philchalmers) によるものです. 私は全てを確認していませんが, おおよそ良さそうだと思います.

1. `Options -> Execution Modes -> PDFTeXify` へ移動する. それから実行可能な `Rscript.exe` (例: `C:\Program Files\R\R-2.14.2\bin\Rscript.exe`) を探し, それを選択する
2. `Switches` で `-e` を入力し, `Parameters` で `"knitr::knit2pdf('%n%t')"` を入力する.

そして `F9` をタイプすれば, PDF を開くのを含めて全てが一度に実行されます.

Phil に感謝.

## Emacs/ESS {-}

12/9 以降, **knitr** は公式に [ESS](http://ess.r-project.org) にサポートされています. Debian/Ubuntu をお使いの場合, 以下のようにしてインストールできます.

```bash 
sudo apt-get install ess
```

ESS で **knitr** を使う方法に関する[短い動画](https://web.archive.org/web/20161225094148im_/http://cdn.screenr.com/video/d43075)があります.

(歴史的背景に興味がある人向け) [Simon Potter](http://sjp.co.nz/posts/emacs-ess-knitr/) と [lucialam](https://constantmindmapping.wordpress.com/2012/06/12/knitr-and-emacs/) の両氏が Emacs/**knitr** についてブログに書いています.

## Gedit {-}

[gedit](https://en.wikipedia.org/wiki/Gedit) で外部ツールとして定義することができます. 以下は David Allen による方法です. 感謝. 

```bash
Rscript -e "library(knitr); knit('$GEDIT_CURRENT_DOCUMENT_NAME')"
```

## Sublime {-}

Andrew Heiss による [KnitrSublime](https://github.com/andrewheiss/KnitrSublime) パッケージは Sublime Text 2 で LaTeX で **knitr** を使用する基本的な機能をサポートしています.

## Vim {-}

Jakson Aquino のおかげで [Vim-R-Plugin](http://www.vim.org/scripts/script.php?script_id=2628) **knitr** の包括的なサポートをしています.

## TextMate {-}

Applescript for TextMate 2 で使用するアプローチとして [#252](https://github.com/yihui/knitr/issues/252#issuecomment-6034068) や Chris Fonnesbeck のリポジトリ [knitr.tmbundle](https://github.com/fonnesbeck/knitr.tmbundle) を参照してください.

## TeXShop {-}

[TeXShop](http://pages.uoregon.edu/koch/texshop/) で **knitr** を動作させる設定は簡単です. TeXShop の `Engines` ディレクトリ (大抵の場合は `~/Library/TeXShop/Engines/`) の `Knitr.engine` ファイルに以下を書き込むだけです.

```bash 
#!/bin/bash
export PATH=$PATH:/usr/texbin:/usr/local/bin
Rscript -e "library(knitr); knit('$1')"
latexmk -pdf "${1%.*}"
```

[Cameron Bracken](http://cameron.bracken.bz/sweave-for-texshop) と [Fabian Greimel](http://yihui.org/en/2012/06/enjoyable-reproducible-research/#comment-601032753) の厚情に感謝.

## TeXworks {-}

追加ツールの設定に関して TeXworks も Texmaker と似ています. 以下は Ubuntu でのスクリーショットです. StackExchange 回答してくれた [Speravir](http://tex.stackexchange.com/a/85165/9128) に感謝. (Windows/Mac OS も `Rscript` が `PATH` にある限り同様にできるはずです).

(ref:demo-editors-texworks) TeXworks で **knitr** を使う

\begin{figure}

{\centering \includegraphics[width=1\linewidth,height=1\textheight,keepaspectratio]{d6tE6} 

}

\caption{(ref:demo-editors-texworks)}(\#fig:demo-editors-texworks)
\end{figure}

## Kile {-}

tirip01 が以下のような方法を指摘しています.

1. `Build` タブを開き, `New..` を選んで, `knitr` とタイプし `Finish` を押します. `General` タブから `Comannd` タブで `Rscript` と入力し, その下の `Options` フィールドで `-e "knitr::knit2pdf('%source')"` とタイプします.

    ![](https://securecdn.disqus.com/uploads/mediaembed/images/564/4837/original.jpg)

1. `Advanced` タブへ移動し, Rnw を `Source extension` に設定し, pdf を `Target extension` に設定します.

    ![](https://securecdn.disqus.com/uploads/mediaembed/images/564/4838/original.jpg)

1. メニューから `Compile` を選択します.

Dr Marek Gągolewski も [Configure Kile for knitr under GNU/Linux](http://www.rexamine.com/2013/04/configure-kile-for-knitr/) というブログ投稿でもっと複雑なアプローチを解説しています.

## Tinn-R {-}

Tinn-R は **knitr** v2.3.7.3 以降からサポートを開始しました. 

<!--chapter:end:demo/2012-03-16-editors.Rmd-->

<!---
title: Package framed
subtitle: Default LaTeX style in knitr
date: '2012-02-29'
slug: framed
--->

# `framed` パッケージ {-#framed}

**knitr** における LaTeX のデフォルトスタイル

オリジナルのページ: https://yihui.org/knitr/demo/framed/

オリジナルの更新日: 2012/2/29

----

:::{.infobox .tip data-latex="{tip}"}
このページは `knitr` 単体の場合を解説しています. R Markdown の場合, `framed` が関わるのは基本的にコードチャンクの表示スタイルのみです.
:::

デフォルトでは **knitr** はタイプセットに [framed](http://www.ctan.org/pkg/framed) という LaTeX パッケージを使用しています. 代表的な特徴として, 薄灰色の影がつけられます. このページではいくつかのトリックと既知の問題を紹介します.

[よくある質問](#FAQs)で挙げたように, 影付きボックスからはみ出ることがあるかもしれません. その際は `options('width')` でより小さい値を設定してください.

## 要素の概要 {-}

テキストのはみ出しはさておき, 図もまた影付きの余白を超えるかもしれません. 図の幅が広すぎる場合, LaTeX は `kframe` 環境で問題が合った旨を警告します. `kframe` は **knitr** チャンク出力を包むために使用します. 既知の問題では,  issue [#154](https://github.com/yihui/knitr/issues/154) で PNG を使った場合があります. 確実にページ余白を超えないようにするため, **knitr** は以下のコマンドを LaTeX プリアンブルで使用します.

```tex
%% maxwidth is the original width if it's less than linewidth
%% otherwise use linewidth (to make sure the graphics do not exceed the margin)
\makeatletter
\def\maxwidth{ %
  \ifdim\Gin@nat@width>\linewidth
    \linewidth
  \else
    \Gin@nat@width
  \fi
}
\makeatother
```

出力が LaTeX の場合, チャンクオプション `out.width` はデフォルトで `'\\maxwidth'` 設定されます.

## 影付きボックスのパディング {-}

もしデフォルトのレイアウト (パディングがまったくない)が倹約しすぎると感じた場合, この LaTeX コマンドでパディングを 5mm に設定します.

```tex
\setlength\fboxsep{5mm}
```

**framed** パッケージのデフォルトのスタイルが気に入らない場合, [listings](#listings) や自分自身で定義した出力[フック](#hooks) に切り替えられます.

## **framed** と互換性のない環境 {-}

二段組の文書内での `figure*` とは相性がよくありません. この状況に対処するアプローチに1つとして, [knitr-twocolumn.pdf](https://github.com/yihui/knitr/releases/download/doc/knitr-twocolumn.pdf) を参照してください.

Tufte handout/book クラスを使う場合, `fullwidth` 環境と **framed** パッケージは併用できません. 可能性のある解決策として, issue [#222](https://github.com/yihui/knitr/issues/222) の議論を参照してください.

**lineno** パッケージとも併用できません. [Michael's の投稿](http://groups.google.com/group/knitr/browse_thread/thread/b0d6723386371139)を参照してください.

<!--chapter:end:demo/2012-02-29-framed.Rmd-->

<!---
title: Listings
subtitle: Using knitr with listings
date: '2011-12-10'
slug: listings
--->

# `Listings` {-#listings}

`listings` と `knitr` の併用

オリジナルのページ: https://yihui.org/knitr/demo/listings/

オリジナルの更新日: 2011/12/10

----

:::{.infobox .tip data-latex="{tip}"}
このページは主に R Markdown ではなく Rnw を想定していることに注意してください.
:::

**knitr** では, LaTeX の `listings` パッケージで結果を装飾するためにの出力[フック](#hooks)を簡単に定義することができます. 以下のようなスニペットを使うことになるでしょう.


```{.r .numberLines .lineAnchors}
## a common hook for messages, warnings and errors
hook_lst_bf <- function(x, options) {
  paste("\\begin{lstlisting}[basicstyle={\\bfseries}]\n", x,
    "\\end{lstlisting}\n",
    sep = ""
  )
}
knit_hooks$set(source = function(x, options) {
  paste("\\begin{lstlisting}[language=R,numbers=left,stepnumber=2]\n", x,
    "\\end{lstlisting}\n",
    sep = ""
  )
}, output = function(x, options) {
  paste("\\begin{lstlisting}[basicstyle={\\ttfamily}]\n", x,
    "\\end{lstlisting}\n",
    sep = ""
  )
}, warning = hook_lst_bf, message = hook_lst_bf, error = hook_lst_bf)

## empty highlight header since it is not useful any more
set_header(highlight = "")
```

見て分かるように, **knitr** は全てをユーザーに公開してます. 必要なのはこれらの R コードの部品と出力を適切な環境で包むことです.

**ちょっと待ってください**, 上記のコードをコピペしないでください. これはすでに少々の機能を追加した上で `render_listings()` 関数として **knitr** に組み込まれています. これが使用例になります.

- **knitr** で `listings` をつかう
  - Rnw ソース: [knitr-listings.Rnw](https://github.com/yihui/knitr/blob/master/inst/examples/knitr-listings.Rnw)
  - LyX ソース: [knitr-listings.lyx](https://github.com/yihui/knitr/blob/master/inst/examples/knitr-listings.lyx)
  - PDF 出力: [knitr-listings.pdf](https://github.com/yihui/knitr/releases/download/doc/knitr-listings.pdf)

PDFの出力のスクリーショットを1つお見せします:

\begin{figure}

{\centering \includegraphics[width=1\linewidth,height=1\textheight,keepaspectratio]{PKupQ} 

}

\caption{knitr での listings の使用}(\#fig:demo-listings-knitr)
\end{figure}

LaTeX のスタイルファイル `Sweavel.sty` を提供してくれた Frank Harrell に感謝します.

## さらなる listings オプション.

**listings** パッケージには膨大な数の使用可能なオプションがあるので, その全性能を引き出すにはマニュアルを読み込んでください. 以下はエラーメッセージで開業する方法の例を提示しています. この[Rnw ソース](https://gist.github.com/2209775)をダウンロードすることができます.  ポイントは `breaklines=true` オプションです.

\begin{figure}

{\centering \includegraphics[width=1\linewidth,height=1\textheight,keepaspectratio]{3313T} 

}

\caption{listings の出力の改行}(\#fig:demo-listings-knitr-break-line)
\end{figure}

<!--chapter:end:demo/2011-12-10-listings.Rmd-->

<!---
title: Control output
subtitle: Manipulate six types of chunk output and inline output
date: '2012-01-25'
slug: output
--->

# チャンク出力の制御 {-#output}

チャンクの6種類の出力とインライン出力を操作する

オリジナルのページ:

オリジナルの更新日: 2012-01-25

----

[main manual](https://github.com/yihui/knitr/releases/download/doc/knitr-manual.pdf) の導入の通り, **knitr** は R コードチャンクの評価のために **`evaluate`** パッケージを使用し, その出力は6種類あります. **ソースコード**, **通常テキスト出力**, **メッセージ文**, **警告文**, **エラー文**, そして**グラフ**です. 以降ではこれらをうまく制御する方法を要約します.

1. **ソースコード**: チャンクオプション `echo` を使います, 例: `echo=FALSE` は R コードを隠します
1. **通常テキスト出力**: `results` オプションを使います (`markup` はテキストをマークアップし, `asis` はテキストをそのまま出力し, `hide` は結果を隠します)
1. **メッセージ文**: `message` オプションを使います (`FALSE` は出力されるメッセージ文を隠します)
1. **警告文**: `warning` オプションを使います (`FALSE` は出力される警告文を隠します)
1. **エラー文**: `error` オプションを使います (`FALSE` はエラー発生時点で R の処理を停止し, `TRUE` は出力にエラー文を表示します)
1. **グラフ**: `fig.keep` オプションを使います. (`none` は全てのグラフを破棄し, `all` は全ての低水準プロットに分けて保存し, `high` は高水準作図として保存します)

これらのオプションは互いに独立しており, 他のタイプの出力への影響を気にすることなく自由に切り替えることができます.

**knitr** の全ての[オプション]($options)は R の評価式から値を取ることができます. これは main manual で条件評価 (conditional evaluation) の機能として紹介したものです. 端的に言うなら `eval=dothis` は実際の値 `value` はグローバル環境の `dothis` とい変数から取られた値になるということです. この変数を操作することで, 1つ1つのチャンクの評価をオン・オフ切り替えることができます.

## `echo` オプションの発展的な使い方 {-}

チャンクオプション `echo` は `TRUE/FALSE` だけでなく, 数値のベクトルを取ることで出力文を選択することができます. このベクトルのインデックスはコードチャンクを完全な R 評価式単位でインデックスします. たとえば `echo=1` は出力内の最初のソースコード出力のみを含めることを意味します. 以下はこの例の完全版です.

```
<<hide-par, echo=3:4>>=
## 表示させたくない「醜い」コード
par(mar = c(4, 4, 0.1, 0.1), cex.lab = 0.95, cex.axis = 0.9,
    mgp = c(2, 0.7, 0), tcl = -0.3)
plot(mtcars[, 1:2])
plot(mtcars[, 4:5])
@
```

評価式 `par()` はこのコードチャンクに必須ではなく, 読み手の集中力を逸らすことすらあります. そこで出力からこれを隠したいとなるでしょう. このケースでは, 最初の評価式 (コメントのことです) も見せたくありません. `echo=3:4` ならば3, 4番めの評価式が出力に含まれることを意味します. 評価式のインデックスが行番号と同じである必要はありません. 代わりに 1,2番めの評価式を削除するという意味で `echo=-(1:2)` とすることもできます.

ソースコードを部分部分に分けて選択すると, 読み手は混乱するかもしれません. (相対的に) 完全な部分集合を選ぶために, ほとんどの場合は `a:b` または `-(a:b)` を使うべきでしょう. しかし, 誰もあなたに禁じることはできません.

``` 
% 3, 5 番めの評価式を選択
<<hide-par, echo=c(3, 5)>>=
## 表示させたくない「醜い」コード
par(mar = c(4, 4, 0.1, 0.1), cex.lab = 0.95, cex.axis = 0.9,
    mgp = c(2, 0.7, 0), tcl = -0.3)
plot(mtcars[, 1:2])
par(mar = c(4, 4, 1, 0.5)) # reset margins
plot(mtcars[, 4:5])
@
```

## インライン出力 {-}

チャンクの出力とは別の出力タイプがあります.  インライン R コードの出力です (例: `\Sexpr{t.test(x)$p.value}`). 数値の出力は特別な扱われ方をします. 非常に大きいか小さい数値は指数表記で書き出されます. 指数表記になるしきい値は R オプションの `scipen` (詳細は `?options` を参照してください) によります. 基本的に $10^4$ より大きいか, $10^{-4}$ より小さい場合に指数表記になります (負の数でも絶対値が同様に評価されます). 出力フォーマット (LaTeX とか HTML とか) に応じて, **knitr** は `$3.14 \times 10^5$` や `3.14 &times; 10<sup>5</sup>` というように適切なコードで出力します.

もう1つの R オプション `digits` は, どの桁で丸めるべきかを制御します. デフォルトの `options(scipen = 0, digits = 4)` が気に入らないならば, こんなふうに最初のチャンクで変更できます.


```{.r .numberLines .lineAnchors}
## 10^5 以上ならば指数表記され, 2 桁で丸める
options(scipen = 1, digits = 2)
```

R Markdown での例:

```md
インラインコードでは「``` `r'' 1+1` ```」と表示される
```

インラインコードでは「2」と表示される

上記の例では他の地の文とともに `2` と表示されます.

Rnw の例 (LaTeX):

```
Inline code looks like this \Sexpr{1+1}
```

R HTML の例:

```html 
<p>Inline code looks like this <!--rinline 1+1 --></p>
```

R HTML 文書では, デフォルトでは結果の文字列は `<code></code>` で囲まれます. 出力から `<code></code>` タグをなくしたいなら, R コードを `I()` で囲むだけです. 例:

```html 
<p>Inline code looks like this <!--rinline I(1+1) --></p>
```

さらに別の用例を[スタックオーバーフローの投稿](http://stackoverflow.com/q/14124022/559676)で見ることができます.

## Long lines of text output {-}

通常, R はテキスト出力時に `width` オプション (`options(width = ??)` で設定されたもの) を尊重します. たとえば `rnorm(100)` として見てください. `width` のデフォルト値は `75` に設定されていますが, あなたが LaTeX を使っているならこれより小さい値を望むかもしれません. いくつかのケースでは, 小さな値を設定していたとしても実際の出力の幅が広すぎることがあるかもしれません. それは大抵の場合, R がこのオプションを尊重してないためです. 私はこの問題にあまりできることはありませんが, **knitr** 側でハックする方法がいくつかあります. その例の1つは issue [#421](https://github.com/yihui/knitr/issues/421) で見られます^[訳注: `citation()` の出力が `width` に対応していない場合の問題です].

## 1メッセージに1コメントを {-}

R でメッセージ文を書き出すのに `cat()` を使うのが好きな人がたくさんいますが, これは大変よくない使い方です. このような形にメッセージ文は非表示にするのが大変だからです. 本当に **メッセージ** を提示したい場合は, `message()` 関数を使うべきです. 正規のメッセージは `suppressMessages()` で表示を抑制したり, **knitr** で補足したりすることができ便利です. パッケージ開発者の中にはこの問題に注意を払ってない人がいます. そのようなパッケージを読み込んだりパッケージの関数を使ったりすると, 表示を抑制できない偽物のメッセージ文を見かけます. パッケージのスタートアップメッセージはじっさい必要ですが, それは `packageStartupMessage()` で表示すべきです. **knitr** の `message=FALSE` でメッセージを消せないなら, それはパッケージ開発者に変更を要求するときです.

同様に, 警告を発したいならまさに `warning()` を使うべきです.

というわけで, あなたの `cat` の振る舞いに気をつけてください!

## その他 {-}

チャンクで `echo=FALSE, results='hide'` を使用した時, 出力に余分な空白行があるかもしれません. 空白行がいらないのなら, issue [#231](https://github.com/yihui/knitr/issues/231) で解決できるかもしれません.

<!--chapter:end:demo/2012-01-25-output.Rmd-->

<!---
title: Chunk Reference/Macro
subtitle: How to reuse chunks
date: '2012-01-14'
slug: reference
--->

# チャンク参照/マクロ {-#reference}

チャンクの再利用方法

オリジナルのページ: https://yihui.org/knitr/demo/reference/

オリジナルの更新日 2012/01/14

----

:::{.infobox .tip data-latex="{tip}"}
**訳注**: このページは全編を通して Sweave の構文で書かれていますが, R Markdown のチャンクでも同様のことが可能です.
:::

Sweave には `<<chunk-label>>` (`<<>>=` と違い `=` がない点に注意) という構文でチャンクを再利用するためにチャンクを参照する機能があります. たとえば

``` 
<<chunk1>>=
1 + 1
@

<<chunk2>>=
<<chunk1>>
@
```

`chunk2` では `chunk1` のコードが挿入されます. この機能は **knitr** でも有効ですが **knitr** はさらに任意の (制限なし) 階層での再帰的なチャンク参照をサポートしています (Sweave は1段階までしかサポートしていませんでした). つまりあるチャンクはさらに別のチャンクを参照しているチャンクを参照できるということです.

この `<<chunk-label>>` 構文と同様のものは markdown の構文でも機能します. それ以前の, 他のチャンクを含んでいる, 名前を付けた markdown チャンクを再利用することができます. あなたの使っている R Markdown エディタが「予期しないトークンです」と構文に警告していたとしてもです.

**訳注**:  R Markdown でもチャンク中に `<<`, `>>` で囲んでラベルを書くことでチャンク参照ができます.

**knitr** でチャンクを再利用するアプローチは他にもあります.

1. 再利用したいチャンクと同じ名前のラベルを使う
1. そのチャンクを参照する `ref.label` オプションを使う

## 同じラベルを使用する {-}

1番目のアプローチの例です

```sweave
<<chunk1, echo=TRUE, results='hide'>>=
1 + 1
@

<<chunk1, echo=FALSE, results='markup'>>=
@
```

2番目のチャンクは空なので, **knitr** は同じ名前を持ち, 空でないチャンクを探し, そのチャンクのコードを使用します. ポイントは他のチャンクを使用するためにはチャンクの中身を空にしておくことです. 問題は, この2つのチャンクの MD5 ハッシュ値が異なるため, 両方のキャッシュを残すことができないというものです. **knitr** はラベル1つにつき1まとまりのキャッシュしか取ることができません.

## チャンクオプション `ref.label` {-}

2番目のアプローチの例です.

```sweave
<<chunk1, echo=TRUE, results='hide'>>=
1 + 1
@

<<chunk2, ref.label='chunk1', echo=FALSE, results='markup'>>=
@
```

2番目のチャンクはラベルが異なるので, キャッシュが取れない問題はなくなりました. 明らかに, 第2のアプローチはより汎用的な解決法です.

この機能は出力文書において R のコードと出力を分離することを可能とします. 簡単な例として, 論文を書く際に本文中に R コードを表示しないために `echo=FALSE` を設定することができます. そして補遺 (Appendix)  のセクションでこの R コードを掲載するため, チャンク参照を使います (`eval=FALSE, ref.label=...` オプションを使います).

<!--chapter:end:demo/2012-01-14-reference.Rmd-->

<!---
title: Graphics
subtitle: Power of graphics in knitr
date: '2011-12-09'
slug: graphics
--->

# グラフィックス {-#graphics}

**knitr** におけるグラフィックスの力について

オリジナルの記事: https://yihui.org/knitr/demos/graphics

オリジナルの更新日: 2011/12/9

----

:::{.infobox .tip data-latex="{tip}"}
このページは主に R Markdown ではなく Rnw を想定していることに注意してください.
:::

## グラフィックスマニュアル {-}

グラフィックスマニュアルでは **knitr** のグラフィックスに彩りを加えるものを紹介します.

- グラフィックスマニュアルのソースと出力は以下です
  - Rnw ソース: [knitr-graphics.Rnw](https://github.com/yihui/knitr/blob/master/inst/examples/knitr-graphics.Rnw)
  - LyX ソース: [knitr-graphics.lyx](https://github.com/yihui/knitr/blob/master/inst/examples/knitr-graphics.lyx)
  - PDF 出力: [knitr-graphics.pdf](https://github.com/yihui/knitr/releases/download/doc/knitr-graphics.pdf)

刊行にあたって R グラフィックスの改善の余地が大いにあることに気づくかもしれません. R があなたにもたらすものを鵜呑みにしないでください. あなたのグラフを美しくプロフェッショナルにすることを考える時間です.

マニュアルからいくつかスクリーショットを提示します.

(ref:demo-graphics-tikz) **knitr** 上の tikz グラフィックス

\begin{figure}

{\centering \includegraphics[width=1\linewidth,height=1\textheight,keepaspectratio]{HCkka} 

}

\caption{(ref:demo-graphics-tikz)}(\#fig:demo-graphics-tikz)
\end{figure}

(ref:demo-graphics-ggplot2) **knitr** 上の **`ggplot2`**

\begin{figure}

{\centering \includegraphics[width=1\linewidth,height=1\textheight,keepaspectratio]{BTEiu} 

}

\caption{(ref:demo-graphics-ggplot2}(\#fig:demo-graphics-ggplot2)
\end{figure}

 [`tufte-handout`](http://code.google.com/p/tufte-latex/) クラスの作者に感謝します. 上記の例はこれを利用しています. そして **tikzDevice**  パッケージは文書クラスと一貫したフォントスタイルのグラフをもたらしてくれます (セリフフォントを使用しています).

## カスタムグラフィックデバイスについての補足 {-}

[チャンクオプション](#options) の `dev` は3つの引数をとる R 関数として定義されたカスタムグラフィックデバイスに対応します. これは `pointsize` 10 を使用した PDF デバイスの例です^[訳注: R Markdown の場合, 日本語表示は後述するように `cairo_pdf` のほうが簡単です.].


```{.r .numberLines .lineAnchors}
my_pdf <- function(file, width, height) {
  pdf(file, width = width, height = height, pointsize = 10)
}
```

これでチャンクオプションでこのデバイスを使えるようになりますが, 1つ重要なことを覚えておいてください. **knitr** はグラフファイルに対して適切なファイル拡張子を推測できないため, `fig.ext` オプションも同時に指定する必要があります. 最終的に, このカスタムデバイスはこのように使われます.

````r
```{r dev='my_pdf', fig.ext='pdf'}
plot(rnorm(100))
```
````

文書全体でこのデバイスを使用したい場合は, もちろん `\SweaveOpts{}` を使ってグローバルに設定することもできます.

## デバイスに追加の引数を与える {-}

`dev.args` オプションを通してグラフィカルデバイスをよく制御できます.  `pointsize = 10` とハードコーディングする代わりに, チャンクに `dev.args = list(pointsize = 10)` を与えることができます. これが例です.

````r
```{r dev='pdf', dev.args=list(pointsize=10)}
plot(rnorm(100))
```
````

`dev.args` はリストなので, デバイスの引数として可能なものを取るべきです. たとえば `pdf()` には `dev.args=list(pointsize=11, family='serif')`. `dev.args` の全ての要素はチャンクのグラフィカすデバイスに与えられます.

## R グラフィックスにハイパーリンクを付ける {-}

**tikzDevice** パッケージのおかげで, R グラフィックスではほとんどの LaTeX コマンドを使うことができます. ハイパーリンクを付ける例を示します: [links.Rnw](https://gist.github.com/1937313) (Jonathan Kennel に感謝).

特記事項として, あなたは `\usepackage{hyperref}` を **tikzDevice** パッケージのメトリックのリストに与える必要があります. そうでなければ `\hyperlink` や `\hypertarget` コマンドは認識されません.

## マルチバイト文字のエンコーディング {-}

あなたのグラフにマルチバイト文字が含まれている場合, `pdf()` デバイスの `encoding` オプションを指定する必要があります. issue [#172](https://github.com/yihui/knitr/issues/172) を参照してください. 可能なエンコーディングのリストは以下で確認できます.


```{.r .numberLines .lineAnchors}
list.files(system.file("enc", package = "grDevices"))
```

```
##  [1] "AdobeStd.enc"  "AdobeSym.enc"  "CP1250.enc"    "CP1251.enc"   
##  [5] "CP1253.enc"    "CP1257.enc"    "Cyrillic.enc"  "Greek.enc"    
##  [9] "ISOLatin1.enc" "ISOLatin2.enc" "ISOLatin7.enc" "ISOLatin9.enc"
## [13] "KOI8-R.enc"    "KOI8-U.enc"    "MacRoman.enc"  "PDFDoc.enc"   
## [17] "TeXtext.enc"   "WinAnsi.enc"
```

```{.r .numberLines .lineAnchors}
## 例: pdf.options(encoding = 'CP1250')
```

このような警告メッセージを目にした場合, エンコーディングの設定が必要かもしれません.

```
Warning: conversion failure on '<var>' in 'mbcsToSbcs': dot substituted for <var>`.
```

別の手段として, `pdf`  の代わりに  `cairo_pdf` を使うというものがあります. (issue [#436](https://github.com/yihui/knitr/issues/436) を参照してください)[^use-cairo-pdf][^use-rmarkdown]:


```{.r .numberLines .lineAnchors}
options(device = function(file, width = 7, height = 7, ...) {
  cairo_pdf(tempfile(), width = width, height = height, ...)
})
```

もし Windows 環境でこれが失敗するのなら, issue [#527](https://github.com/yihui/knitr/issues/527) を確認してください.

[^use-cairo-pdf]: 訳注: 投稿したい論文雑誌などから特別な要件がない限り, 現在は `cairo_pdf` のほうが簡単だと思います.
[^use-rmarkdown]: 訳注: R Markdown であれば, `dev = 'cairo_pdf'` で十分です.

## 装飾フォント {-}

`pdf()` のドキュメントによれば, `useDingbats` 引数は小さな円を含む PDF のファイルサイズを減らしてくれる可能性があります. RStudio 上で **knitr** を使っている場合, このオプションはデフォルトで無効になっています. 巨大な散布図を含む場合, ソース文書に `pdf.options(useDingbats = TRUE)` と書くことで有効になるでしょう. 詳細な議論は issue [#311](https://github.com/yihui/knitr/issues/311) を参照してください.

## アニメーション {-}

チャンクオプション `fig.show='animate'` が設定されコードチャンクで複数のグラフが生成されている場合, 全てのグラフが統合されアニメーションになります. LaTeX の出力では, LaTeX パッケージの **animate** が使われ, HTML/Markdown の出力に対しては, デフォルトでは [FFmpeg](http://ffmpeg.org) が使われ [WebM](http://www.webmproject.org) 動画が作られます. FFmpeg をインストールする際に  **libvpx** のサポートを有効にする必要があることに注意してください. Linux および Windows ユーザーは FFmpeg ウェブサイトのダウンロードリンクを確認してください (バイナリ版では **libvpx** は有効になっています). OSX ユーザーは, [Homebrew](http://brew.sh) 経由で FFmpeg をインストールできます.


```{.bash .numberLines .lineAnchors}
brew install ffmpeg --with-libvpx
```

<!--chapter:end:demo/2011-12-09-graphics.Rmd-->

<!---
title: Manual
subtitle: The package manual
date: '2011-12-05'
slug: manual
--->

# マニュアル {-#manual}

パッケージマニュアルについて

オリジナルのページ https://yihui.org/knitr/demos/manual

オリジナルの更新日: 2011/12/5

:::{.infobox .tip data-latex="{tip}"}
訳注: このページは **knitr** ドキュメントのトップページではありません.
:::

パッケージのマニュアルそれ自体は **knitr** のほとんどの機能の良い使用例となりえます. このマニュアルは LyX で書かれ (([knitr-manual.lyx](https://github.com/yihui/knitr/blob/master/inst/examples/knitr-manual.lyx))), Rnw ソース ([knitr-manual.Rnw](https://github.com/yihui/knitr/blob/master/inst/examples/knitr-manual.Rnw)) にエクスポートして PDFファイル [knitr-manual.pdf](https://github.com/yihui/knitr/releases/download/doc/knitr-manual.pdf) を生成できます.

マニュアルのコンパイルには3つのパッケージが追加で必要です. **`rgl`**, **`animation`**, **tikzDevice** です.

LyX で **knitr** を使う手順は [LyX page](https://yihui.org/knitr/demo/lyx/) を参照してください.

<!--chapter:end:demo/2011-12-05-manual.Rmd-->

<!---
title: Cache
subtitle: Examples for the cache feature
date: '2011-12-04'
slug: cache
--->

# キャッシュ {-#cache}

キャッシュ機能の使用例について

オリジナルのページ: https://yihui.org/knitr/demo/cache/

オリジナルの更新日: 2011-12-04

----

チャンクオプションの `cache=TRUE` でキャッシュを有効にでき, `cache.path` でキャッシュ用ディレクトリを指定できます. \@ref(options) 章『オプション』を参照してください.

## キャッシュの例 {-}

キャッシュ機能は私の文書の多くで広く使われています. たとえば **knitr** の [メインマニュアル](#manual) や [グラフィックス](#graphics) です. ここでは, もう少しいくつかの例を挙げます.

- 基本的な例
  - 巨大なデータをキャッシュする [056-huge-plot.Rmd](https://github.com/yihui/knitr-examples/raw/master/056-huge-plot.Rmd) ([出力](https://github.com/yihui/knitr-examples/blob/master/056-huge-plot.md)
  - Rtex の構文を使った例: [knitr-latex.Rtex](https://github.com/yihui/knitr/blob/master/inst/examples/knitr-latex.Rtex)
- キャッシュの依存関係の自動的な構成
  - Rnw ソース: [knitr-dependson.Rnw](https://github.com/yihui/knitr-examples/blob/master/017-auto-dependson.Rnw)
  - チャンクオプション `autodep=TRUE` と関数 `dep_auto()` によって, **knitr** チャンク間の依存関係を解決することが可能となり, `dependson` オプションを指定するための手作業をいくらか省くことができます.
  
## 重要な補足事項 {-}

キャッシュがいつ再構成されるかと, キャッシュされないチャンクについてよく理解するため, [メインマニュアル](https://github.com/yihui/knitr/releases/download/doc/knitr-manual.pdf) (英語版 PDF) のキャッシュに関するセクションを, とても注意深く読まなければなりません.

キャッシュに影響する要素を今一度繰り返します (いかなる変更も古いキャッシュを無効にします):

1. `include` を除く全てのチャンクオプション, たとえば `tidy=TRUE` を `FALSE` に変えるだけでも古いキャッシュは破棄されますが, `include` は例外です.
1. チャンク内のR コードのわずかな変更, スペースや空白行の追加・削除であっても, 古いキャッシュを削除することにつながります

極めて重要な注意事項として, 副次的な作用をもつチャンクはキャッシュ**すべきでない**ということを挙げます. **knitr** は `print()` による副次作用を維持しようとしますが, さらなる別の副次作用は保存されません. ここでチャンクをキャッシュすべきでないケースをいくつか挙げます.

1. `options('width')`, `pdf.options()`,  または `opts_chunk$set()`, `opts_knit$set()`, `knit_hooks$set()` のような他のなんらかの **knitr** のオプションを設定する時
1. キャッシュされたチャンクでの `library()` によるパッケージの読み込みと読み込まれたパッケージはキャッシュされないチャンクによっても扱われます. (キャッシュされたチャンクでパッケージを読み込み, 使うのは全く OK です. **knitr** はキャッシュされたチャンクでパッケージのリストを保存するためです. しかし, キャッシュされないチャンクが, それまでのキャッシュされたチャンクでどのパッケージをロードしたかを知ることはできません.)

さもなければ次回からはチャンクはスキップされ, そこでの設定はすべて無視されます. このようなチャンクでは明示的に `cache=FALSE` を使わなければなりません.

`source()` と `setGeneric()` は, コードがローカル環境で評価された場合でもグローバル環境にオブジェクトを作成するという副次作用を持ちます. **knitr** 0.4 より前ではこれらのグローバルオブジェクトをキャッシュすることはできませんでした (たとえば issue [#138](https://github.com/yihui/knitr/issues/138) を見てください). しかし ver. 0.4 以降は **knitr** が `globalenv()` で作成されたオブジェクトを確認し, 他のオブジェクト同様に保存するようになったため, キャッシュできるようになりました.

キャッシュされたチャンクで使われるパッケージのリストは保存されますが, パッケージ名をキャッシュする方法としては完璧ではありません. かりにパッケージを読み込み, あとで除外したとしても, **knitr** はそれを知ることはできません (新たに読み込まれたパッケージしか捕捉できません). Issue [#382](https://github.com/yihui/knitr/issues/382) で解説されているように, キャッシュディレクトリにある `__packages` ファイルを手動で編集しなければなりません.

## キャッシュにはまだ何かありますか? {-}

以上のオブジェクトがキャッシュに影響するのはいかにも納得できることですが, 再現可能性のある研究ではキャッシュは他の変更によって無効化されるという点でよりいっそう厳格になることがあります. 典型例の1つはソフトウェアのバージョンです. 2つのバージョンが異なるRに, 異なる結果を出させるのは不可能ではありません. この場合, 我々はこう設定するかもしれません.


```{.r .numberLines .lineAnchors}
knitr::opts_chunk$set(cache.extra = R.version.string)
knitr::opts_chunk$set(cache.extra = R.version) # あるいはプラットフォームの違いも考慮する
```

これによって, キャッシュされた結果は特定のバージョンの R でのみ適用されます. R をアップグレードし文書を再コンパイルしたとき, 全ての結果は再計算されます.

同様に, キャッシュが特定の環境でのみ保存されるように, このオプションに他の変数も設定したいかもしれません. これは野心的な例です.


```{.r .numberLines .lineAnchors}
## キャッシュは特定の R のバージョンとセッションでのみ有効
## キャッシュは最大で1ヶ月間保存される (来月は再計算される)
knitr::opts_chunk$set(cache.extra = list(
  R.version, sessionInfo(), format(Sys.Date(), "%Y-%m")
))
```

この `cache.extra` オプションの別の良い使用例が issue [#238](https://github.com/yihui/knitr/issues/238) で示されています. この例ではキャッシュはファイルの更新日時と関連付けられています. つまり, データファイルが変更されれば, キャッシュは自動的に再構成されることになります^[訳注: ファイルの更新日時を取得するには base R の `file.mtime` 関数が便利とのことです].

注: キャッシュ条件にさらにオブジェクトを導入する際に, `cache.extra` 以外の任意のオプション名を使用することができます. たとえば `cache.validation` も呼び出せます. 全てのチャンクオプションはキャッシュの確認時に考慮されるためです.

## キャッシュディレクトリを入力ファイル名と連動させる {-}

ときとしてデフォルトとは異なる入力ファイルに対して異なるキャッシュディレクトリを使いたいことがあるかもしれません. 解決策の1つが issue [#234](https://github.com/yihui/knitr/issues/234) で提示されています. しかし, 自己完結性を高めるため, この設定はソースドキュメントの内部で行うことを推奨します (`opts_chunk$set(cache.path = ...)` を使ってください.).

## より細かいキャッシュのとり方 {-}

上級者はチャンクオプション `cache` に対して `TRUE` か `FALSE` かだけでなく, 数字で設定するなどしてもっと粒度の細かいキャッシュが欲しいと考えるかもしれません. `cache = 0, 1, 2, 3` として, `0` は `FALSE`, `3` は `TRUE` と同じで, `cache = 1` は計算の結果のみキャッシュから読み込む (`evaluate::evaluate()` から), よってコードは再評価されませんが, 他の箇所, 出力フックや作成されたグラフの保存といった箇所は評価されます. `cache = 2` ならば, `1` とほとんど同じですが, 唯一の違いはグラフファイルがすでに存在する場合, 再保存されない点です. これはグラフが大きい場合, いくらか時間の削減になります. 以前のRセッションで記録されたグラフが別のRセッション, あるいは別のバージョンで安全に再保存されるという保証はないため, `cache = 1` よりは `cache = 2` を推奨します.

`cache = 1, 2` の場合は少数のチャンクオプションだけがキャッシュに影響します. オプション名は `knitr:::cache1.opts`, `knitr:::cache2.opts` で確認してください. 基本的に, コード評価に影響しないチャンクオプションが変更されてもキャッシュは無効になりません. たとえば `echo` を `TRUE` から `FALSE` に変えるとか, `fig.cap = '新しいキャプション'` と設定するなどの変更です. しかし, `eval` を `TRUE` から `FALSE` に変えた場合, あるいは `cache.path='foo/'` を `'bar/'` に変えた場合, キャッシュは再構成されます.

いくつかの例が, [example #101](https://github.com/yihui/knitr-examples/) ([出力](https://github.com/yihui/knitr-examples/blob/master/101-cache-levels.md)) で見られます.

この方法では, 計算と文書の出力レンダリングを分離することが可能となり,
キャッシュを破棄することなく出力を調整するのに役立ちます. issue [#396](https://github.com/yihui/knitr/issues/396), [#536](https://github.com/yihui/knitr/issues/536) を確認してください.

## 乱数生成器 (RNG) の再現可能性 {-}

乱数生成器 (RNG) が関係するチャンクの再現性を維持するため,  **knitr** は `.Random.seed` をキャッシュし, 各チャンクの評価直前に復元します. しかし1つ問題があります. チャンク A, B がキャッシュされていたとします. この時A, B の間に新たにチャンク C を挿入したとします (3つのチャンクは全て内部で RNG を使用しています). RNG が変更されたため B が更新されるはずですが, `.Random.seed` は副次作用であるため, これに反して実際には B は更新されません. はっきり言うと, B の再現性は偽りのものとなります.

RNG の関係する再現可能性を保証するために, `.Random.seed` とキャッシュを関連付ける必要があります. これが変更される度にキャッシュも更新されなければなりません. これは `cache.extra` オプションで **評価されない** R 評価式を参照することで簡単にできます. たとえば以下のように.


```{.r .numberLines .lineAnchors}
knitr::opts_chunk$set(cache.extra = rand_seed)
```

`?rand_seed` を確認してください (これは評価されない R 評価式です). この場合は各チャンクは最初に `.Random.seed` が最後の実行から変更されたかどうかを確認します. `.Random.seed` が異なるならキャッシュは再構成されます.

<!--chapter:end:demo/2011-12-04-cache.Rmd-->

<!---
title: knitr showcase
subtitle: Examples from other users
date: '2013-03-11'
slug: showcase
--->

# `knitr` のショーケース {-#showcase}

ユーザーたちによる使用例

オリジナルのページ: https://yihui.org/knitr/demo/showcase/

オリジナルの更新日: 2013-03-11

----

:::{.infobox .tip data-latex="{tip}"}
訳注: 更新日時からも分かるように, このリストは古く, また膨大であるため個別の翻訳はいたしません. R に限らず, 活発に利用されているオープンソースソフトウェアは頻繁に更新されるため, 非公式の解説はしばしば out-of-date になりうる, ということに注意してください.
:::

:::{.infobox .memo data-latex="{memo}"}
以下のリンク集は外部サイトによる `knitr` の使用事例集です (もしリンクしてほしくない, あるいはしてほしいということであれば私 (Yihui) に気軽に連絡してください)
:::

## ウエブサイト {-}

- [RPubs](http://rpubs.com/): Easy web publishing from R
- [knitr in a knutshell](http://kbroman.github.io/knitr_knutshell), a short tutorial by Karl Broman
- [R learning resources](http://www.ats.ucla.edu/stat/r/) at UCLA by Joshua Wiley et al (dynamically built with **knitr**)
- [knitr on ShareLaTeX](http://www.sharelatex.com/learn/Knitr) (an online LaTeX editor)
- [Rcpp Gallery](http://gallery.rcpp.org/): Articles and code examples for the **Rcpp** package
- [Slidify](http://slidify.org/): reproducible HTML5 slides made easy
- [One Page R](http://onepager.togaware.com/) Literate Data Science by Graham Williams
- [Reproducible graphics with R and ggplot2](http://www.scribd.com/doc/93360631/Presentation) by Baptiste Auguié
- [ML/stats notes](https://github.com/johnmyleswhite/MLNotes) by John Myles White
- [A French introduction to R](http://alea.fr.eu.org/pages/intro-R) by Julien Barnier (also see [CRAN](http://cran.r-project.org/other-docs.html))
- Applications of R in Business Contest ([knitr's entry](http://www.inside-r.org/howto/knitr-elegant-flexible-and-fast-dynamic-report-generation-r); [announcement](http://www.revolutionanalytics.com/news-events/news-room/2012/applications-of-r-in-business-competition.php))

## 書評 {-}

[Dynamic Documents with R and knitr](http://amzn.com/1498716962) に対する書評のリストです.

- A [book review](http://www.jstatsoft.org/v56/b02/) in the _Journal of Statistical Software_ by Amelia McNamara
- A [book review](http://www.maa.org/press/maa-reviews/dynamic-documents-with-r-and-knitr-0) in _MAA Reviews_ by Peter Rabinovitch
- A [book review](http://www.tug.org/books/reviews/tb109reviews-xie.html) on [TUGboat](http://www.tug.org/books/#xie-knitr) by Boris Veytsman
- [A book review](http://rpubs.com/safeisrisky/knitr_booksummary) on RPubs by RK
- [A book review](http://dx.doi.org/10.1080/00031305.2015.1014676) in _The American Statistician_ by Quan Zhang

## `knitr` によるソリューション {-}

- [a knitr Howto page](http://biostat.mc.vanderbilt.edu/wiki/Main/KnitrHowto) in Vanderbilt Biostatistics Wiki
- [Plain Text, Papers, Pandoc](http://kieranhealy.org/blog/archives/2014/01/23/plain-text/) by Kieran Healy
- [R Markdown output formats for Tufte-style handouts](https://github.com/sachsmc/tufterhandout) by Michael Sachs
- [Blogging with Rmarkdown, knitr, and Jekyll](http://brendanrocks.com/blogging-with-rmarkdown-knitr-jekyll/) by Brendan Rocks
- [Blog with Knitr and Jekyll](http://jfisher-usgs.github.com/r/2012/07/03/knitr-jekyll/) by Jason C Fisher
- [Creating HTML5 slides with RStudio, knitr and pandoc](http://gastonsanchez.wordpress.com/2012/11/19/creating-html5-slides-with-rstudio-knitr-and-pandoc/) by Gaston Sanchez
- [A framework to create bootstrap styled HTML reports from knitr Rmarkdown](https://github.com/jimhester/knitrBootstrap) by Jim Hester (a [preview](http://htmlpreview.github.io/?https://raw.github.com/jimhester/knitrBootstrap/master/inst/examples/all.html))
- [Creating a Business Dashboard in R](http://fishyoperations.com/creating-a-business-dashboards-in-r.html) by Bart Smeets
- Carl Boettiger has cool blog posts demonstrating how to publish a post to Wordpress.com with knitr and RWordPress purely in R, with images uploaded to [Imgur](http://www.carlboettiger.info/archives/3974) and [Flickr](http://www.carlboettiger.info/archives/3988) respectively
- [knitr + cactus + TwitterBootstrap + Jquery](http://geospaced.blogspot.com/2012/05/knitr-cactus-twitterbootstrap-jquery.html) by Barry Rowlingson (includes a smart use of jQuery to add links to R functions)
- [Interactive reports in R with knitr and RStudio ](http://lamages.blogspot.co.uk/2012/05/interactive-reports-in-r-with-knitr-and.html) and [Interactive HTML presentation with R, googleVis, knitr, pandoc and slidy ](http://lamages.blogspot.com/2012/05/interactive-html-presentation-with-r.html) by Markus Gesmann
- [Stacked bar plots with several descriptive nodes](http://tex.stackexchange.com/a/87897/9128) by ADP
- [Blogging from R to Wordpress](http://wkmor1.wordpress.com/2012/07/01/rchievement-of-the-day-3-bloggin-from-r-14/) by William K. Morris
- A demo on using `tidy=TRUE` or the listings environment so code chunks can stay inside page margins ([link to StackExchange](http://tex.stackexchange.com/q/41471/9128))
- [How to Use Knitr with a Rakefile](http://lincolnmullen.com/blog/how-to-use-knitr-with-a-rakefile/) by Lincoln A. Mullen
- [Reproducible Research with Word?](http://ericpgreen.com/reproducible-research-with-word/) by Eric P. Green
- [knitr, slidify, and Popcorn.js](http://ramnathv.github.io/slidifyExamples/examples/popcornjs) by Ramnath Vaidyanathan ([#466](https://github.com/yihui/knitr/issues/466))
- [Transparent, reproducible blogging with nanoc and knitr](http://chogg.name/blog/2015/03/19/nanoc_knitr/) by Charles Hogg
- [Using knitr and R to make instructor/student handout versions](http://lukemiller.org/index.php/2015/11/using-knitr-and-r-to-make-instructorstudent-handout-versions/) by Luke Miller
- [Thesis template to generate LaTeX files using R with knitr](https://github.com/asardaes/R-LaTeX-Template) by Alexis Sarda

## R パッケージ {-}

- [The Github Wiki of the cda package](https://github.com/baptiste/cda/wiki) by Baptiste Auguie
- [website of ggbio package](http://tengfei.github.com/ggbio/) by Tengfei Yin
- [The sampSurf Package](http://sampsurf.r-forge.r-project.org/) by Jeffrey H. Gove
- [the RHadoop Wiki](https://github.com/RevolutionAnalytics/RHadoop/wiki) by Revolution Analytics
- [The ggmcmc package examples](http://xavier-fim.net/packages/ggmcmc/) by Xavier Fernández-i-Marín
- [tabplot: Tableplot, a visualization of large datasets](http://cran.r-project.org/package=tabplot) by Martijn Tennekes and Edwin de Jonge (see its PDF vignette)
- the [ggplot2 transition guide](https://github.com/djmurphy420/ggplot2-transition-guide) to version 0.9.0 by Dennis Murphy et al
- a few packages on Bioconductor: [ReportingTools](http://www.bioconductor.org/packages/2.12/bioc/vignettes/ReportingTools/inst/doc/knitr.html) and [RGalaxy](http://www.bioconductor.org/packages/2.12/bioc/html/RGalaxy.html)
- [the dendextend package](https://github.com/talgalili/dendextend) by Tal Galili
- [An Introduction to CHNOSZ](http://chnosz.net/vignettes/anintro.html) vignette by Jeffrey M. Dick using Tufte style

## 教材 {-}

以下は Roger Peng の厚情によってシェアされた Cousera の講座 [_Computing for Data Analysis_](https://www.coursera.org/course/compdata) 向けの **knitr** の講義です.

<iframe width="100%" height="450" src="//www.youtube.com/embed/YcJb1HBc-1Q" frameborder="0" allowfullscreen></iframe>

さらに以下も **knitr** と関連する講座の資料です.

- [BMI 826-003](http://kbroman.github.io/Tools4RR/) (Tools for Reproducible Research) by Karl Broman, University of Wisconsin-Madison
- [Reproducible Research](https://www.coursera.org/course/repdata) by Roger Peng, Coursera
- [Introduction to data science](http://www.unomaha.edu/mahbubulmajumder/data-science/fall-2014/) by Mahbubul Majumder, University of Nebraska at Omaha
- [BIOS 301](http://fonnesbeck.github.com/Bios301/) (Introduction to Statistical Computing) by Chris Fonnesbeck, Vanderbilt University
- [Math 344](http://www.calvin.edu/~rpruim/courses/m344/S12/) (Probability and Statistics) by Randall Pruim, Calvin College
- [Stat 506](http://www.math.montana.edu/~jimrc/classes/stat506/) (Advanced Regression) by Jim Robison-Cox, Montana State University
- [STT 3820](http://www1.appstate.edu/~arnholta/classes/STT3820/index.htm) by Alan T. Arnholt, Appalachian State University
- [Math 747](http://www.math.mcmaster.ca/~bolker/classes/m747/) (Topics in math biology) by Ben Bolker, McMaster University
- [STAT 319](http://www.personal.psu.edu/yoh5109/319spring2013/) (Applied Statistics in Science) by Yuan Huang, Penn State University (also see the tutorial [Create Dynamic R Statistical Reports Using R Markdown](https://onlinecourses.science.psu.edu/statprogram/markdown))
- [some notes](http://bcb.dfci.harvard.edu/~aedin/courses/ReproducibleResearch/) on reproducible research by Aedin Culhane, Harvard University
- [STAT 622](http://www.stat.rice.edu/~marina/stat622.html) (Bayesian Data Analysis) by Marina Vannucci, Rice University
- [STA613/CBB540](http://genome.duke.edu/labs/engelhardt/courses/cbb540.html) (Statistical methods in computational biology) by Barbara Engelhardt, Duke University
- [Stat 590](http://statacumen.com/teaching/sc1/) (Statistical Computing) by Erik Erhardt, University of New Mexico
- [STAT 497C](https://onlinecourses.science.psu.edu/stat497r/node/101) (Topics in R Statistical Language) by Eric Nord, Penn State University
- [CSSS-Stat 567](http://www.stat.washington.edu/~hoff/courses/567/) (Statistical Analysis of Social Networks) by Peter Hoff, University of Washington
- [Math 15](http://msemac.redwoods.edu/~darnold/math15/) (Statistics) by David Arnold, College of the Redwoods
- [STAT 545A](http://www.stat.ubc.ca/~jenny/STAT545A/) Exploratory Data Analysis by Jennifer Bryan, University of British Columbia
- [STAT545](http://www.stat.purdue.edu/~varao/STAT545/main.html) Introduction to Computational Statistics, by Vinayak Rao, Purdue University
- [Sta 101](https://stat.duke.edu/courses/Fall14/sta101.001/) Data Analysis and Statistical Inference, by Mine Çetinkaya-Rundel, Duke University
- [GEOL 6370](http://strata.uga.edu/6370/) Data Analysis in the Geosciences, by Steven M. Holland, University of Georgia

## ワークショップ・プレゼンテーション {-}

以下は Joshua Wiley の厚情によって作成・シェアされたチュートリアル資料です.

<iframe width="100%" height="450" src="//www.youtube.com/embed/p2VBzhLVz3o" frameborder="0" allowfullscreen></iframe>

文字に起こされたプレゼンテーション:

- [New tools and workflows for data analysis](https://speakerdeck.com/jennybc/new-tools-and-workflows-for-data-analysis) by Jennifer Bryan ([video](http://www.fields.utoronto.ca/video-archive/2015/02/318-4374))
- [Geospatial Data in R and Beyond](http://www.maths.lancs.ac.uk/~rowlings/Teaching/UseR2012/) by Barry Rowlingson
- [Broom Spatial R Class](http://davenportspatialanalytics.squarespace.com/blog/2012/6/19/notes-from-a-recent-spatial-r-class-i-gave.html) by Frank Davenport ([PDF](https://dl.dropbox.com/u/9577903/broomspatial.pdf))
- [Visualizing Categorical Data](http://euclid.psych.yorku.ca/datavis.ca/courses/VCD/R/output/) by Michael Friendly
- [ggplot2 workshop notes](http://www.ling.upenn.edu/~joseff/avml2012/) by Josef Fruehwald for AVML 2012
- [R Introduction for UCL PhDs](http://www.homepages.ucl.ac.uk/~uctpfos/files/introPHD.html) by Florian Oswald at University College London
- [Introduction to R lectures for ECPR Winter School 2013](http://zfazekas.github.com/teaching/2013/02/17/ecpr-intro-to-R/) by Zoltán Fazekas, University of Southern Denmark
- [R for the brave](https://dl.dropbox.com/u/4149392/R_for_the_brave.pdf) by Will Pearse
- [Introduction to knitr: The R Markdown (Rmd) format](http://lcolladotor.github.com/Rmd-intro/) by L. Collado Torres for JHSPH Biostat computing club
- [Stop Clicking, Start typing](http://mwfrost.com/r_slides/r_slides.html) by Matt Frost
- [そろそろRStudioの話でもしてみようと思う](http://www.slideshare.net/wdkz/rstudio-13866958) by 和田 計也
- [Introduction to Data Analysis and Visualization using R](https://speakerdeck.com/vinayakh/introduction-to-data-analysis-and-visualization-using-r) by Vinayak Hedge
- [Creating publication quality graphics using R](http://metvurst.wordpress.com/2013/07/15/creating-publication-quality-graphics-using-r-3/) by Tim Salabim
- [Reproducible Research Using Knitr/R](https://github.com/umd-byob/byob/tree/master/presentations/2013/0903-knitr_reproducible_research) by Keith Hughitt

## 書籍 {-}

- [The Analysis of Data](http://theanalysisofdata.com/) by Guy Lebanon (written with R Markdown)
- [Dynamic Report Generation with R and knitr](http://www.crcpress.com/product/isbn/9781482203530) by Yihui Xie (written with LyX + the `knitr` module)
    - [Dynamic Report Generation with R and knitr, Second Edition](https://www.crcpress.com/product/isbn/9781498716963)
- [Text Analysis with R for Students of Literature](http://www.matthewjockers.net/2013/09/03/tawr/) by Matthew L. Jockers
- [Data Analysis for the Life Sciences](https://leanpub.com/dataanalysisforthelifesciences) by Rafael Irizarry and Michael Love
- [Using R for Introductory Statistics, Second Edition](http://www.crcpress.com/product/isbn/9781466590731) by John Verzani
- [Learning R: A Step-by-Step Function Guide to Data Analysis](http://shop.oreilly.com/product/0636920028352.do) by Richard Cotton (written with AsciiDoc + `knitr`)
- [Introductory Fisheries Analysis with R](http://derekogle.com/IFAR/) by Derek H. Ogle
- [The Statistical Sleuth In R](http://www.math.smith.edu/~nhorton/sleuth/) by Nicholas Horton, Kate Aloisio, and Ruobing Zhang (`knitr` + LaTeX)
- [Regression Modeling Strategies](https://groups.google.com/forum/?hl=en&fromgroups#!topic/knitr/bYj3Zn11hjE) (`knitr` + LaTeX)
- [Latent Variable Modeling using R: A Step-By-Step Guide](http://blogs.baylor.edu/rlatentvariable/)
- [Biolostatistical Design and Analysis using R](http://yihui.org/en/guestbook/#comment-456270476)
- [Statistics for Experimental Economists: Elegant Analysis with R](http://www.experimentalecon.com/papers/estats.pdf) by Mark A. Olson
- [R과 Knitr를 활용한 데이터 연동형 문서 만들기](http://ksbapp.com/)
- [El arte de programar en R: un lenguaje para la estadística](http://www.imta.gob.mx/biblioteca/libros_html/el-arte-de-programar-en-r/) by Julio Sergio Santana and Efraín Mateos Farfán
- [PH525x series - Biomedical Data Science](http://genomicsclass.github.io/book/) by Rafael Irizarry and Michael Love
- [Data Science for Fundraising: Build Data-Driven Solutions Using R](http://a.co/0axoWr6) by Ashutosh Nandeshwar and Rodger Devine

## 論文・レポート {-}

- [Our path to better science in less time using open data science tools](https://www.nature.com/articles/s41559-017-0160) by Julia S. Stewart Lowndes _et al_, Nature Ecology & Evolution **1**, Article number: 0160 (2017)
- Eglen, SJ; Weeks, M; Jessop, M; Simonotto, J; Jackson, T; Sernagor, E. A data repository and analysis framework for spontaneous neural activity recordings in developing retina. GigaScience 2014, 3:3 <http://dx.doi.org/10.1186/2047-217X-3-3>
    - plus [an interview](http://blogs.biomedcentral.com/gigablog/2014/03/26/carmen-reproducible-research-and-push-button-papers/) to the first author
    - [Q&A on dynamic documents](http://blogs.biomedcentral.com/gigablog/2014/04/16/qa-on-dynamic-documents/)
- [Programming tools: Adventures with R](http://www.nature.com/news/programming-tools-adventures-with-r-1.16609) by Sylvia Tippmann, _Nature_ **517**, 109–110 (01 January 2015) doi:10.1038/517109a
- [Rebooting review](http://dx.doi.org/10.1038/nbt.3202), _Nature Biotechnology_ **33**, 319 (2015)
- [Rule rewrite aims to clean up scientific software](http://dx.doi.org/10.1038/520276a), _Nature_ **520**, 276–277 (2015)
- [A Guide to Reproducible Code](http://www.britishecologicalsociety.org/publications/guides-to/) (Guides to Better Science), by the British Ecological Society
- [2017 Employer Health Benefits Survey](http://www.kff.org/health-costs/report/2017-employer-health-benefits-survey/) by Kaiser Family Foundation (2017)
    - Referenced by the New York Times article "[While Premiums Soar Under Obamacare, Costs of Employer-Based Plans Are Stable](https://www.nytimes.com/2017/09/19/health/health-insurance-premiums-employer.html)"
- [Irregularities in LaCour (2014)](http://stanford.edu/~dbroock/broockman_kalla_aronow_lg_irregularities.pdf) by David Broockman, Joshua Kalla, and Peter Aronow, a rebuttal paper with retraction letter from Donald P. Green
    - LaCour, Michael J. & Donald P. Green. 2014. "When contact changes minds: An experiment on transmission of support for gay equality[2]." Science 346(6215): 1366.
- [Granger-causality analysis of integrated-model outputs, a tool to assess external drivers in fishery](https://twitter.com/Margarita_RH/status/1090590323406000129) by Margarita Rincón, Rachele Corti, Bjarki Elvarsson, Fernando Ramos, and Javier Ruiz.
- [Harry Alastair V.](https://twitter.com/alharry/status/1088323057230405633), Butcher Paul A., Macbeth William G., Morgan Jess A. T., Taylor Stephen M., Geraghty Pascal T. (2019) Life history of the common blacktip shark, Carcharhinus limbatus, from central eastern Australia and comparative demography of a cryptic shark complex. Marine and Freshwater Research. https://doi.org/10.1071/MF18141
- Piwowar HA, Vision TJ. (2013) Data reuse and the open data citation advantage. PeerJ 1:e175 <http://dx.doi.org/10.7717/peerj.175>
- [Some great short courses](http://www3.nd.edu/~mclark19/projects.html) on R, generalized additive models, and machine learning, etc, by Michael Clark, Center for Social Research, Notre Dame
- [An Introduction to Mediation Analysis](https://github.com/JWiley/mediation) by Joshua F. Wiley
- ORANGE REPORT: [Annual Report of the Swedish Pension System](http://secure.pensionsmyndigheten.se/download/18.76cf683d13f231c03cd1bc/Orange+Report+2012.pdf) by the Swedish Pensions Agency
- [2011 Census Open Atlas Project](http://www.alex-singleton.com/2011-census-open-atlas-project/) by Alex Singleton
- [openWAR](http://arxiv.org/abs/1312.7158): An Open Source System for Evaluating Overall Player Performance in Major League Baseball
- [Design and Analysis of Bar-seq Experiments](http://www.g3journal.org/content/early/2013/10/30/g3.113.008565/suppl/DC1) by Robinson et al., 2014
- [Data and program code for meta-analyses of population health and health services research questions](https://github.com/timchurches/meta-analyses) by Tim Churches
- [Genomic analysis using R and knitr](https://github.com/EBI-predocs/knitr-example) by Konrad Rudolph
- [Assessing the 2016 Budget reforms](https://github.com/grattaninstitute/Assessing-2016-Super-tax-reforms) by John Daley and Brendan Coates
- [CFPB Data Point: Becoming Credit Visible](https://s3.amazonaws.com/files.consumerfinance.gov/f/documents/BecomingCreditVisible_Data_Point_Final.pdf) by the CFPB Office of Research
- [A parametric texture model based on deep convolutional features closely matches texture appearance for humans](https://github.com/tomwallis/cnn_texture_appearance) by Wallis et al.
- [Revisiting the effect of red on competition in humans (supplementary information)](http://www.biorxiv.org/content/biorxiv/suppl/2016/11/21/086710.DC1/086710-1.pdf) by Laura Fortunato and Aaron Clauset
- [Epiviz Web Components: reusable and extensible component library to visualize functional genomic datasets](https://f1000research.com/articles/7-1096/v1) by Jayaram Kancherla, Alexander Zhang, Brian Gottfried, and Hector Corrada Bravo

## 多言語でのラッパー {-}

- [knitr-ruby](https://github.com/ropensci/knitr-ruby): a Ruby wrapper
- [Flask-FlatPages-Knitr](https://github.com/fhirschmann/Flask-FlatPages-Knitr): Knitr preprocessing for Flask-FlatPages

## ブログの投稿 {-}

- [Using knitr and pandoc to create reproducible scientific reports](http://galahad.well.ox.ac.uk/repro/) by Peter Humburg
- [Reproducible research, training wheels, and knitr](http://civilstat.com/?p=1521) by Jerzy Wieczorek
- [Don't R alone!](http://www.noamross.net/blog/2013/1/7/collaborating-with-r.html) A guide to tools for collaboration with R by Noam Ross
- [Getting Started with R Markdown, knitr, and Rstudio 0.96](http://jeromyanglim.blogspot.com.au/2012/05/getting-started-with-r-markdown-knitr.html), [How to Convert Sweave LaTeX to knitr R Markdown](http://jeromyanglim.blogspot.com/2012/06/how-to-convert-sweave-latex-to-knitr-r.html) and [Converting Sweave LaTeX to knitr LaTeX: A case study](http://jeromyanglim.blogspot.com/2012/06/converting-sweave-latex-to-knitr-latex.html) by Jeromy Anglim
- [Tools for making a paper](http://conjugateprior.org/2013/03/tools-for-making-a-paper/) by Will Lowe
- [Integrate data and reporting on the Web with knitr](http://blog.revolutionanalytics.com/2012/09/data-reporting-knitr.html) by me as a guest blog post on Revolution Analytics
- [knitr: A flexible R authoring tool](http://rpubs.com/JoFrhwld/UseR_Sept) (HTML5 slides) by Josef Fruehwald
- [Planting seeds of reproducibility with knitr and markdown](http://citizen-statistician.org/2012/10/08/planting-seeds-of-reproducibility-with-knitr-and-markdown/) by Mine Çetinkaya-Rundel (the Citizen-Statistician blog)
- [A closer look at "How economists get tripped up by statistics"](http://biostat.mc.vanderbilt.edu/wiki/Main/GradStudentsHelpfulExamples) by Laurie Samuels
- [Latex Allergy Cured by knitr](http://timelyportfolio.blogspot.com/2012/04/latex-allergy-cured-by-knitr.html)
- [knitr Performance Report-Attempt 1](http://timelyportfolio.blogspot.com/2012/04/knitr-performance-report-attempt-1.html)
- [Easier literate programming with R](http://aliquote.org/memos/2012/04/02/easier-literate-programming-with-r) by Christophe Lalanne
- [knitR - eine Alternative zu Sweave?](http://www.blogofolio.de/2012/05/knitr-eine-alternative-zu-sweave/) by Christian B.
- [Better R support in pygments by monkey patching SLexer](http://blog.felixriedel.com/2012/05/better-r-support-in-pygments-by-monkey-patching-slexer/) by f3lix
- [被knitr包给震撼到了](http://xccds1977.blogspot.com/2012/05/knitr.html) by [\@xccds](https://twitter.com/xccds)
- [Reproducible Research](http://torsneyt.wordpress.com/2012/05/19/reproducible-research/) by Tom Torsney-Weir (on Vim and Marked)
- [为什么Markdown+R有较大概率成为科技写作主流？](http://www.yangzhiping.com/tech/r-markdown-knitr.html) by 阳志平
- [Governance Indicators](http://www.russellshepherd.com/d/?q=blog/governance-indicators) by Russell Shepherd
- [Petrol prices adjusted for inflation](http://mcfromnz.wordpress.com/2012/07/28/petrol-prices-adjusted-for-inflation/) by Matt Cooper
- [Creating beautiful reports from R with knitr](http://blog.revolutionanalytics.com/2012/08/creating-beautiful-reports-from-r-with-knitr.html) by David Smith
- [An R-based Research Notebook](http://www.tomtorsneyweir.com/research-notebook/) by Tom Torsney-Weir
- [knitR, Markdown, and Your Homework](http://learningdata.wordpress.com/2012/09/30/knitr-markdown-and-your-homework/) by Jarrett Byrnes
- [Color Palettes in HCL Space](http://www.trestletechnology.net/2012/10/color-palettes-in-hcl-space/) by Trestle Technology, LLC
- [Introduction to R and Biostatistics](http://fellgernon.tumblr.com/post/35587597245/introduction-to-r-and-biostatistics-2012-version) by Leonardo Collado Torres
- [Reproducible Research using R and Bioconductor](http://onertipaday.blogspot.com/2012/12/italian-bio-r-day-2012-slides-on.html) by Paolo Sonego
- [Bioinformaticians Need Lab Notebooks Too](http://reasoniamhere.com/bioinformaticians-need-lab-notebooks-too/) by Nacho Caballero
- [From OpenOffice noob to control freak: A love story with R, LaTeX and knitr](http://machine-master.blogspot.com/2013/03/from-openoffice-noob-to-control-freak.html) by Christoph Molnar
- [Including an interactive 3D rgl graphic in a html report with knitr](http://stla.overblog.com/including-an-interactive-3d-rgl-graphic-with-knitr) by Stéphane Laurent
- [Create HTML or PDF Files with R, Knitr, MiKTeX, and Pandoc](http://rprogramming.net/create-html-or-pdf-files-with-r-knitr-miktex-and-pandoc/) by Justin Meyer
- [Reproducible research with R, Knitr, Pandoc and Word](http://quantifyingmemory.blogspot.com/2013/02/reproducible-research-with-r-knitr.html) by Rolf Fredheim
- [Visualizing Farmers' Markets Geo Data using googleVis, plyr, knitr and Markdown using R](http://dataillumination.blogspot.com/2013/03/visualizing-farmers-markets-geo-data_19.html) by Peter Chen
- [2013 NSF Graduate Research Fellowship statistics](http://www.scribd.com/doc/132971631/NSFstats2013) by Elson Liu
- [Ben Bolker's notes on workflows, pipelines, reproducible research, etc.](http://stevencarlislewalker.wordpress.com/2013/07/12/ben-bolkers-notes-on-workflows-pipelines-reproducible-research-etc/) by Steve C Walker
- [Playing with R, ggplot2 and knitr](http://www.complementarytraining.blogspot.se/2013/10/playing-with-r-ggplot2-and-knitr.html) by Mladen Jovanović
- [A simple bootstrap-based knitr template](http://watson.nci.nih.gov/~sdavis/blog/a_simple_bootstrap-based_knitr_template/) by Sean Davis
- [Automated Blogging](http://blog.r-enthusiasts.com/2013/12/04/automated-blogging.html) by Romain François
- [How to avoid scandals using knitr](http://www.mango-solutions.com/wp/2014/01/how-to-avoid-scandals-using-knitr/) by Mango Solutions
- Fast-track publishing using knitr: [Part I](http://gforge.se/?p=928), [Part II](http://gforge.se/?p=960), [Part III](http://gforge.se/?p=1005) by Max Gordon
- [Basic data-frame manipulations in R](http://therostrumblog.wordpress.com/2014/01/29/basic-data-frame-manipulations-in-r/) by THE ROSTRUM
- [Reproducibility is not just for researchers](http://www.dataschool.io/reproducibility-is-not-just-for-researchers/) by Kevin Markham
- [Tools for statistical writing and reproducible research](http://theincidentaleconomist.com/wordpress/tools-for-statistical-writing-and-reproducible-research/) by Bill Gardner
- [knitr ではじめるデータ分析レポート作成 \~基礎編\~](http://qiita.com/hereticreader/items/a3000cb7d5b17ad71731) by Yu ISHIKAWA
- [Starting data analysis/wrangling with R: Things I wish I'd been told](http://reganmian.net/blog/2014/10/14/starting-data-analysiswrangling-with-r-things-i-wish-id-been-told/) by Stian Håklev
- [Knitr's best hidden gem: spin](http://deanattali.com/2015/03/24/knitrs-best-hidden-gem-spin/) by Dean Attali
- [Why use KnitR for scientific publishing?](http://randomlifedata.com/2015/04/why-use-knitr-for-scientific-publishing/) by Rob Les Davidson
- [From Code to Reports with knitr & Markdown](http://www.predictiveanalyticsworld.com/patimes/from-code-to-reports-with-knitr-050915/) by Andrew Brooks
- [Top 10 data mining algorithms in plain R](http://rayli.net/blog/data/top-10-data-mining-algorithms-in-plain-r/) by Ray Li
- [Knotes on Knitr](http://www.jonzelner.net/knitr/r/reproducibility/2016/06/02/knitr/) by Jon Zelner
- [A reproducibility horror story](http://blog.revolutionanalytics.com/2016/08/a-reproducibility-horror-story.html)
- [Reproducible Analytical Pipeline](https://gdsdata.blog.gov.uk/2017/03/27/reproducible-analytical-pipeline/) by Matt Upson
- [Composing reproducible manuscripts using R Markdown](https://elifesciences.org/elife-news/composing-reproducible-manuscripts-using-r-markdown) by Chris Hartgerink, Tilburg University

<!--chapter:end:demo/2013-03-11-showcase.Rmd-->

# (APPENDIX) 補遺 {-}

<!--chapter:end:30-appendix-header.Rmd-->

<!---
title: Objects
subtitle: Objects to manipulate options, patterns and hooks
date: '2017-02-03'
slug: objects
--->


# オブジェクト {#objects}

オプションを操作するオブジェクト, パターン, そしてフックについて

オリジナルのページ: https://yihui.org/knitr/objects/

オリジナルの更新日: 2017-02-03

----

**knitr** パッケージはオプションと設定を制御する特別なオブジェクト (以下では `obj` と表記) を使用します. このオブジェクトは次のようなメソッドを持ちます.

- **`obj$get(name)`**: `name` という名前のオプションを返します. または, `name` が2以上の長さの名前つきベクトルである場合はリストを返します. そして `name` が与えられなかった場合は全てのオプションのリストを返します
- **`obj$set(...)`**: オプションを永続的に変更します. 引数 `...` には `tag = value` の形式, または `list(opt1 = value1, opt2 = value2)` のようなオプションのリストとして与えられます
- **`obj$merge(values)`**: 新しいオプションリストを現在のリストに一時的にマージして, そのリストを返します (もとのリストは変更されません)
- **`obj$restore()`**: オブジェクトを元に戻します

**knitr** のこれらのオブジェクトはユーザーから見ることができます.

- **`opts_chunk`**, **`opts_current`**: [チャンクオプション](#chunk-options)を管理します
- **`opts_knit`**: **knitr** [パッケージ全体のオプション](#package-options)を管理します
- **`knit_hooks`**: [フック関数](#hooks)を管理します
- **[`knit_patterns`](#patterns)**: 入力ドキュメントから R コードを取りだすための正規表現を管理します
- **`knit_engines`**: R 以外の他の言語の処理に対する関数を管理します

`knit_patterns` 以外の他の全てのオブジェクトはデフォルトの初期値が設定されており, `knit_paterns` は特に指定がない場合は入力ファイルのタイプに基づいて自動的に決定されます. `knit_hooks` オブジェクトは1番よく使うことになるでしょう. そして残りの3つは直接使用する機会は少ないと思います. たとえば `opts_chunk` はたいていはコマンドラインで直接実行するよりも入力ファイル内で設定します.

各チャンクが依存している **knitr** の設定は, そのチャンクが実行されるまえに設定**されなければなりません**. スクリプトの一番最初の, 他のどのチャンクよりも先に実行される位置に「`knit`
 の設定用チャンク」を書き, その中で `cache = FALSE`, `include = FALSE` オプションを設定することをおすすめします. 設定用チャンクには, このチャンク自身の実行時に有効化される設定を想定する命令文は含めては**いけません**^[訳注: チャンクオプションの評価はそのチャンクの実行直前になるため, ここでの変更は次のチャンクまで有効にならない, という意味です]. 設定用チャンクはこのようになるでしょう^[訳注: この例は `.Rnw` のものであり,  `<< ... >>=` というヘッダの記法も`.Rnw` で使われることに注意してください. R Markdown では `{r, ...}` を使います (参考: \@ref(chunk-options) 章)].

````r
```<<setup, cache=FALSE, include=FALSE>>=
library(knitr)
opts_knit$set(upload.fun = imgur_upload, self.contained = FALSE,
              root.dir = '~/R/project')
@
```
````

技術的な注意点として, これらのオブジェクトはクロージャに近い性質であることを挙げます. これらは関数によって返される, 関数リストです. 詳細は, エクスポートされてない `knitr:::new_defaults` 関数を確認してください. チャンクオプションはクロージャによっても管理されています.

<!--chapter:end:31-objects.Rmd-->

<!---
subtitle: A list of regular expressions to extract R code and chunk options from the
  input document
date: '2017-02-03'
slug: patterns
--->

# パターン {#patterns}

入力文書から Rコードとチャンクオプションを取得するための正規表現のリストについて

オリジナルのページ: https://yihui.org/knitr/patterns/

オリジナルの更新日: 2017-02-03

----

[オブジェクト](#objects) の `knit_patterns` は **knitr** のパターンを管理します. たとえば現在の正規表現パターンのリストを取得するには `knit_patterns$get()` が使えます. パターンリストには以下のコンポネントが含まれています.

- **`chunk.begin`**: コードチャンクの開始時点のパターンです. チャンクオプションを取りだすための, `()` で定義されたグループが含まれていなければなりません.
- **`chunk.end`**: チャンクの終了時点のパターンです (文芸的プログラミングにおける本来の意味とは異なります. 本来は通常のテキストの開始時点を意味していました. 詳細は \@ref(options) 章の `filter.chunk.end` オプションを参照してください).
- **`chunk.code`**: このパターンにマッチする文字を除去することで, チャンクから R コードを取り出します.
- **`inline.code`**: 地の文のなかにまぎれたインライン R コードを取りだすためのパターンです (つまり, コードチャンクの分離には使いません). `chunk.begin` 同様に, グループが含まれていなければなりません.
- **`inline.comment`**: インラインコメントのパターンです (このパターンにマッチするインライン R コードのトークンは, 各行から除去されます)
- **`header.begin`**: 文書のヘッダの開始点を見つけるためのパターンです. 出力文書にヘッダ情報を挿入する際に使われます (例: LaTeX のプリアンブルコマンド, HTML のCSS スタイル) 
- **`document.begin`**: 文書の本文開始点を見つけるためのパターンです (たとえば LaTeX プリアンブルを取りだすのに使うとして, tikz のグラフィックスを外部化したり, シンタックスハイライトのためにコードを挿入したりすることができます)

パターンが `NULL` である場合, 何もマッチしません.

Sweave のように, **knitr** には2種類の R コードがあります. パラグラフのようなまとまりである「コードチャンクと」, 地の文の中にあって実行される「インライン R コード」です. 文書内のチャンクのオプションは `label, opt1=TRUE, opt2=FALSE, opt3='character.string'` という形式になります (オプションは `,` と `=` で繋げられ, ラベルのみが `label` が暗黙に `value` とみなされるため, ラベルのみが `value` を必要としません).

## ビルトインパターン

**knitr** には `all_patterns` を保存するいくつかのビルトインパターンがあります.


```{.r .numberLines .lineAnchors}
library(knitr)
str(all_patterns)
```

**`Knitr`** は最初に適切なパターンのセットを決定するために入力文書の中身を探索します. もしこの自動検出が失敗したら, 入力文書のファイル拡張子名から判定し, **knitr** は自動で上記のリストから適切な形式を選び出します. たとえば入力ファイルが `file.Rnw` ならは `all_patterns$rnw` を, `file.html` ならば `all_patterns$html` を, というふうに使用します.

`pat_rnw()`, `pat_html()`, `pat_md()`, `pat_tex()`, `pat_brew()` という一連の便利な関数はビルトインパターンを設定するために使われます.

<!--chapter:end:33-patterns.Rmd-->

