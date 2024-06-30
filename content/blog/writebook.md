---
title: "Writebook"
date: 2024-07-01T00:37:24+09:00
description: "Writebookというアプリケーションが無料で公開されており、その中でpuma-devやProcfileなどの興味深い技術が使われていることを紹介しています。"
tags:
- rails
---

37signalsのサイト「Once」では、[Writebook](https://once.com/writebook)というアプリケーションが無料で提供されています。

![Writebook](/images/writebook-01.webp)

MRSK(Kamal)が公開された頃から、AWSからVPSのようなデプロイ環境への移行が進んでいるようです。私も普段はオンプレミス環境を使用しているため、このような動向は好ましいと感じています。

詳しくはOnceのページを参照していただきたいのですが、Onceの理念は非常に興味深いものです。特に注目すべきは、インストールスクリプト経由であのCampfireのソースコードを入手できる点です。

通常、有償のRailsのソースコード一式を購入することは稀ですが、DHHやBasecampのソースコードを見られる機会は貴重です。ただし、1ドル160円の円安時代に6万円もするため、購入は難しい価格設定です。

いつかCampfireを購入してみたいと思っていたところ、Writebookが無償で公開されたのは願ってもないことでした。Docker経由で配布されているのでソースコードを取得しようと思っていましたが、ZIPファイルも提供されており大変助かります。

今回は、Writebookの中で興味深いと思った箇所をいくつか紹介します。

## puma-dev

まず注目したのは[puma-dev](https://github.com/puma/puma-dev)というプロジェクトです。

かつてPowというプロジェクトがありましたが、Linuxに移行してからはこの手の開発ツールを使っていませんでした。puma-devの存在は知っていましたが、試す機会がありませんでした。

Writebookの`bin/setup`には、開発環境で`https://writebook.test`にアクセスすると自動的にサーバーが起動する設定が用意されています。`.test`ドメインの設定はできましたが、サーバーが反応しなかったため、確認はできませんでした。一度設定ができれば、Omakubのようなスクリプトを用意して自動化したいと考えています。

## Procfile

次に興味深いのは`Procfile`です。

ForemanかOvermindかはわかりませんが、`bin/dev`ではありません。私は通常`docker-compose`コマンドで各種サーバーを起動しますが、直接`redis-server`コマンドを起動しています。開発環境ではDockerを使ってローカルにサーバーをインストールしない方が良いと考えていましたが、puma-devを使うなら常時サーバーを起動しておく方が便利かもしれません。docker-compose.ymlに依存しない開発手法もあると知り、目から鱗でした。

## app/assets/stylesheets

Writebookは純粋なCSSで書かれています。

興味深いのは、これらのCSSをどのようにまとめているかという点です。通常、Railsでは`application.css`がデフォルトで存在し、Sprocketsがファイルを変換します。しかし、Writebookには`application.css`が存在しません。

```erb
<%= stylesheet_link_tag :all, "data-turbo-track": "reload" %>
```

代わりに`:all`を指定することで、すべてのCSSリンクが自動的に生成されます。単一ファイルにコンパイルする場合、CSSのファイルサイズは大きくなりますが、ファイルごとにリクエストを分けて差分を抑えようとしているのかもしれません。JavaScriptファイルもImportmapを使っているので、生成されたHTMLのソースファイルを見れば理解しやすいでしょう。

## app/models/first_run.rb

Writebookは、初回インストール時に`FirstRun.create!`を実行します。

これは純粋なRubyクラスで、いわゆるServiceクラスのような使われ方をしています。Railsではあまり見かけないパターンであり、その実装は非常に勉強になりました。

## app/models/

DHHや37signalsでは、ServiceクラスやFormオブジェクトよりも、Concernを使ってモデルにクラスを`extend`することで機能を追加する方法が推奨されています。例えば`Account::Joinable`や`Leaf::Editable`などの命名規則があり、Concernは *-able* を接尾語として用いることが多いですが、`Authorization`や`User::Role`といった名前も見られます。必ずしも *-able* で終わる必要はないという点が興味深いです[^1]。

[^1]: 私が以前携わったプロジェクトでは、必ず *-able* の命名を守っており、それが普通だと思っていました。

## app/controllers/concerns/

続いてコントローラーについてです。

モデルも見やすいコードですが、コントローラーもほとんどのメソッドが5行以内で非常に見やすいです。また、コントローラーのConcernを見る機会は少ないですが、`include`することで自動で`set_`系のメソッドが実行されるようです。

## test、routes.rb

最後に興味深いのはテストと`routes.rb`です。RSpecやFactoryBotを使っていない点は非常に興味深いです。また、`routes.rb`の書き方もResourceベースに加えて、`direct`や`route_for`といった見慣れない構文も発見できました。

## まとめ

改めてRailsは非常に洗練されたフレームワークだと再認識しました。

他にも興味深い箇所は多々ありますが、全てを追うことも書くことも難しいため、今回はこの辺りにしておきます。TurboやStimulusのコードについても、時間をかけて調べていきたいです。

将来的には、このような整然としたコードをメンテナンスできるよう、今後もRuby on Railsについての知識を深めていきたいと思います。
