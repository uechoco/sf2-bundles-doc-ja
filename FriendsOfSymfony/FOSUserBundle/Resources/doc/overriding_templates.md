FOSUserBundle の標準テンプレートのオーバーライド
=================================================

FOSUserBundle をアプリケーションに組み込み始めると、
バンドルから提供されている標準テンプレートをオーバーライドする必要があることにきっと気づくでしょう。
テンプレート名を設定することはできませんが、
Symfony2 フレームワークはバンドルのテンプレートをオーバーライドする方法を2つ提供しています。

1. `app/Resources` ディレクトリに 同じ名前の新しいテンプレートを定義する
2. `FOSUserBundle` の子供(child)として新しいバンドルを作成する

### 例: 標準の layout.html.twig のオーバーライド

`Resources/views/layout.html.twig` テンプレートをオーバーライドすることを強くおすすめします。
というのも、FOSUserBundle によって提供されるページは、アプリケーションの他のページと同じ見た目だからです。
このレイアウトテンプレートのオーバーライドの例を、上に挙げた方法を使って実演してみます。

FOSUserBundle によって提供される標準の `layout.html.twig`:

``` twig
<!DOCTYPE html>
<html>
    <head>
        <meta charset="UTF-8" />
    </head>
    <body>
        <div>
            {% if is_granted("IS_AUTHENTICATED_REMEMBERED") %}
                {{ 'layout.logged_in_as'|trans({'%username%': app.user.username}, 'FOSUserBundle') }} |
                <a href="{{ path('fos_user_security_logout') }}">
                    {{ 'layout.logout'|trans({}, 'FOSUserBundle') }}
                </a>
            {% else %}
                <a href="{{ path('fos_user_security_login') }}">{{ 'layout.login'|trans({}, 'FOSUserBundle') }}</a>
            {% endif %}
        </div>

        {% for key, message in app.session.getFlashes() %}
        <div class="{{ key }}">
            {{ message|trans({}, 'FOSUserBundle') }}
        </div>
        {% endfor %}

        <div>
            {% block fos_user_content %}
            {% endblock fos_user_content %}
        </div>
    </body>
</html>
```

見て分かるとおり初歩的であまり複雑な構造も持っていないので、
アプリケーションに合わせたレイアウトファイルに置き換えたいと思うでしょう。
このテンプレートの中で注意しておきたい主な事柄は、`fos_user_content` という名前のブロックです。
これはそれぞれのバンドルのアクションから来るコンテンツが表示されるブロックですので、
標準のレイアウトファイルをオーバーライドして使いたいレイアウトファイルには、
このブロックを含めなければいけません。

次の Twig テンプレートファイルは、バンドルによって提供されているレイアウトファイルを
オーバーライドして使うためのレイアウトファイルの例です。

``` twig
{% extends 'AcmeDemoBundle::layout.html.twig' %}

{% block title %}Acme Demo Application{% endblock %}

{% block content %}
    {% block fos_user_content %}{% endblock %}
{% endblock %}
```

この例では、`AcmeDemoBundle` という名前の架空のアプリケーションバンドルからレイアウトテンプレートを拡張しています。
`content` ブロックは、各ページのメインコンテンツが描画される場所です。
これが `fos_user_content` ブロックがそれの内側に置かれている理由です。
これが、アプリケーションの見た目を保ちながらもアプリケーションに統合された FOSUserBundle の動作から来る出力結果を、
望んんだ形式に導いてくれるでしょう。

**a) 新しいテンプレートを app/Resources** に定義する

バンドルのテンプレートをオーバーライドする最も簡単な方法は、
新しいテンプレートを `app/Resources` フォルダに配置するだけです。
`FOSUserBundle` ディレクトリの中の `Resources/views/layout.html.twig` に位置するレイアウトテンプレートをオーバライドするために、
新しいレイアウトテンプレートを `app/Resources/FOSUserBundle/views/layout.html.twig` に配置しましょう。

お分かりの通り、この方法でテンプレートをオーバーライドするパターンには、
`app/Resources` ディレクトリにバンドルクラスの名前でフォルダを作成します。
そして、元のバンドルのディレクトリ構造を維持しながら、このフォルダに新しいテンプレートを追加します。

**b) 子バンドルを作成し、Template** をオーバーライドする

**Note:** 

```
この方法は上で説明した方法よりも複雑です。
テンプレートと同様にコントローラーもオーバーライドする予定が無い限りは、
もう一方の方法を使うことをお勧めします。
```

上に挙げた通り、FOSUserBundle の子として定義したバンドルを作り、
FOSUserBundle の中にあるのと同じ配置で新しいテンプレートを置くこともできます。
最初にやりたことは、`getParent` メソッドをバンドルクラスでオーバーライドすることです。

``` php
// src/Acme/UserBundle/AcmeUserBundle.php
<?php

namespace Acme\UserBundle;

use Symfony\Component\HttpKernel\Bundle\Bundle;

class AcmeUserBundle extends Bundle
{
    public function getParent()
    {
        return 'FOSUserBundle';
    }
}
```

バンドルクラスの `getParent` メソッドでバンドル名を返すことで、
バンドルが FOSUserBundle の子であることを Symfony2 フレームワークに知らせています。

ここまでで、バンドルが FOSUserBundle の子として宣言されているので、
親のバンドルのテンプレートをオーバーライドすることができます。
レイアウトテンプレートをオーバーライドするためには、
`src/Acme/UserBundle/Resources/views` ディレクトリに `layout.html.twig` という名前で新しいファイルを作成するだけです。
このファイルが、FOSUserBundle のディレクトリに対して全く同じパスに配置されているかに気をつけてください。

子のバンドルでテンプレートをオーバーライドした後は、開発環境であっても、
オーバーライドを効かせるためにキャッシュをクリアしなければなりません。

FOSUserBundle が提供している他のすべてのテンプレートも、
このドキュメントに示した2つの方法のどちらかを用いて同じようにすることで、
オーバーライドすることができます。

### Twig以外のテンプレートエンジンを設定する

バンドルの設定を用いて、Twig 以外のテンプレートエンジンを設定することができます。
以下の例は、PHP テンプレートエンジンを使うための設定です。

``` yaml
fos_user:
    # ...
    template:
        engine: php
```

FOSUserBundle は、Twig テンプレートエンジンのための標準テンプレートだけしか提供していませんので、
使おうとしているテンプレート全てを作成しなければならないでしょう。
名前や配置は、ファイルの拡張子が `.twig` の代わりに `.php` を使う事以外は、
おなじになるでしょう。

> **TIP** Translated Info: 2011/09/24 uechoco 98b1d15c
