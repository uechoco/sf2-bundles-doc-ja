FOSUserBundle
=============

FOSUserBudnleは、Symfony2においてデータベースを背景にもつユーザー体系のサポートを追加します。
ユーザー管理のための柔軟な骨組みを提供し、
ユーザーログインや登録、パスワードの復旧などの共通タスクを取り扱います。

含まれている機能:

- ユーザーはDoctrine ORM、MongoDB ODM、CouchDB ODMによってストア出来る
- REST-fulな認証
- メールによる確認オプションも含めた、登録サポート
- パスワードリセットのサポート
- ユニットテスト済み

**注意:** このバンドルは[symfony's repository](https://github.com/symfony/symfony)と同期して開発されています。

ドキュメント
-------------

ドキュメント一式は`Resources/doc/index.md`に格納されていて、バンドルに含まれています:

[Read the Documentation](https://github.com/FriendsOfSymfony/FOSUserBundle/blob/master/Resources/doc/index.md)

インストール
------------

インストール手順は[documentation](https://github.com/FriendsOfSymfony/FOSUserBundle/blob/master/Resources/doc/index.md)にあります。

ライセンス
----------

このバンドルはMITライセンス下にあります。ライセンスの全文はバンドルの中にあります:

    Resources/meta/LICENSE

関連
-----

UserBundleは[knplabs](https://github.com/knplabs)の主導です。
また[contributors](https://github.com/FriendsOfSymfony/FOSUserBundle/contributors)の一覧も御覧ください。

問題のレポートや機能の要求
---------------------------------------

問題や機能のリクエストは[Github issue tracker](https://github.com/FriendsOfSymfony/FOSUserBundle/issues)にトラッキングされています。

バグをレポートするときは、バンドルの開発者が簡単に複製してステップを踏むだけで問題を再現できるように、
[Symfony Standard Edition](https://github.com/symfony/symfony-standard)を使ってビルドした基本プロジェクトを再現してもらえると助かります。

> **TIP** Translated Info: 2011/09/19 uechoco 837707a0
