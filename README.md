sf2-bundles-doc-ja リポジトリ
==============================

このプロジェクトは experimental (実験的) です。

趣旨
----

様々な Symfony2 用バンドルのドキュメントの翻訳

なぜ必要
--------

サードパーティ製の Symfony2 バンドルの多くは
バンドル内に英文をドキュメントを保持しているが、
その多くは多言語に翻訳されることを意識されていない。
翻訳しても、置く場所を毎回用意する必要があるのは大変である。

そこで、様々なバンドルの翻訳ドキュメントを一括に取り集めたリポジトリを作り、
そこに溜めていくのはどうかと思い、このリポジトリを試験的に設置した。

構成指針(第１版)
-----------------

### ディレクトリ構造

まずはgithubのリポジトリを想定して、以下の構造を提案する。

書式:

<pre><code>  /githubユーザー名
    /githubリポジトリ名
      /以下、各リポジトリのドキュメント構造を忠実に再現
</code></pre>

例:

<pre></code>  /FriendsOfSymfony
    /FOSTwitterBundle
      /README.md
    /FOSUserBundle
      /README.markdown
      /UPDATE.md
      /Resources
        /doc
          /index.md
          /command_line_tools.md
          /...
</code></pre>

### ドキュメントの書式

原文と同じ書式(ex: markdown, reStructuredText, text, ...)にしましょう。

最初はこの README.md を除くすべてを reStructuredText で記述したいとは思ったのですが、
そうすると原文が更新されたときに更新に追随するのが辛くなりそう。
原文と同じ書式ということにしましょう。
