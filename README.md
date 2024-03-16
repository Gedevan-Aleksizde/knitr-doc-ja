『knitr: Rによる美麗で柔軟そして高速な動的レポート生成』日本語版
================

本文はこちらです: <https://gedevan-aleksizde.github.io/knitr-doc-ja/>

## これは何?/What’s this?

- [Yihui 氏のリポジトリ](https://github.com/rbind/yihui) の `knitr`
  パッケージの解説『knitr: Elegant, flexible, and fast dynamic report
  generation with R』を翻訳した文書です. 氏のサイトの全てのコンテンツは
  [CC BY-NC-SA 4.0 (表示 - 非営利 - 継承 4.0
  国際)ライセンス](https://creativecommons.org/licenses/by-nc-sa/4.0/deed.ja)
  で提供されています.
- よって, あくまで個人的な翻訳であり, 「公式」ではありません.
- 翻訳の底本は `yihui` ディレクトリに submodule
  で紐付けられたバージョンに準拠しています.

This is an unofficial Japanese translation of Yihui’s **knitr**
documentation, which is licensed under [CC BY-NC-SA
4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/). The original
documentation by Yihui is [here](https://yihui.org/knitr/).

## 関連リンク

- “[R Markdown
  Cookbook](https://bookdown.org/yihui/rmarkdown-cookbook/)”
  ([近日翻訳予定](https://github.com/Gedevan-Aleksizde/rmarkdown-cookbook))
- “[R Markdown: The Definitive
  Guide](https://bookdown.org/yihui/rmarkdown/)”
  ([近日翻訳予定](https://github.com/Gedevan-Aleksizde/rmarkdown-book))
- “[bookdown: Authoring Books and Technical Documents with R
  Markdown](https://bookdown.org/yihui/bookdown/)”
  ([近日翻訳予定](https://github.com/Gedevan-Aleksizde/bookdown))

<details>
<summary>
運用手順
</summary>

# 環境

再現性のため renv を使っている. `rmdja@development`
を使っているので再現性は厳密ではない.

# 更新手順

もっとかんたんな方法はないか?

1.  `cd yihui & git pull` で submodule を更新する
2.  更新差分を
    `https://github.com/yihui/yihui.org/compare/${lastcommit}..master`
    とかで見る.
    - TODO: どこで lastcommit を取得する?
3.  差分を見て変更された箇所に対応する
4.  `bookdown::render_book(input = "source/", c("rmdja::gitbook_ja", "rmdja::pdf_book_ja"))`
    でビルドする.
    - 元のリポジトリはYihui氏のブログをビルドするためのものだが,
      ブログの全訳は作っていないので本体のワークフローを真似する必要はない.

</details>
