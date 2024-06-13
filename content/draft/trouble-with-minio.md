---
title: "Minioの画像が表示できない問題"
date: "2024-06-07"
description: "自宅サーバーでMinioを運用しているが、画像表示に苦労し、設定変更やデバッグを通じてCarrierwaveのasset_hostやfog_publicの設定を調整する必要があった。"
tags:
- minio
- rails
draft: true
---

前提として私は自宅サーバー、今風に表現するとHomelabでMinioの運用をしている。
Minioを扱うのは特に初めてというわけでもないし、3年前にもMinioに対する投稿を行っていたようだ。

しかし今週はMinioの画像を表示したくてかなり苦労した。
こういうやや特殊な環境だとクラウドのようにただアップロードできればよいというわけではなくて、何が原因なのかひとつずつ段階を踏んでデバッグを行っていく必要がある。

![ネットワーク図](/images/20240607.webp)

慣れないのだけれども、せっかくなので図におこしてみた。
この図で表現したかったのは私は`nginx`のリバースプロキシを設定している。
同一のLANではないので`minio`の管理画面ないし、エンドポイントのURLにはドメイン経由でしかアクセスできないことになっている。

以前このような問題は発生しなかったのだが、そのときは`web`と`minio`はそれぞれ同一ネットワークのコンテナ同士で通信していた。
今回は仮想マシン同士ではあるが実際のLAN上で運用している。
クラウドではないのでスケーリングをする予定はないのだが、デプロイはKamalを利用することにしている。

Railsアプリケーションから任意のファイルをMinioにアップロードすることはできる。
今回はCarrierwaveを利用した。
ActiveStorageでもそこまで変わりはないと思う。

```ruby
CarrierWave.configure do |config|
  config.fog_credentials = {
    provider:              'AWS',
    aws_access_key_id:     ENV['FOG_ACCESS_KEY_ID'],
    aws_secret_access_key: ENV['FOG_SECRET_ACCESS_KEY'],
    use_iam_profile:       false,
    endpoint:              'http://192.168.2.2:9000', # 重要
    path_style:            true
  }
  config.fog_directory  = 'test-app'
  config.fog_public     = false
end
```

最初はこのような形式だった。
ここでは`minio`のアドレスを`192.168.2.2`としている。

またMinioの場合は`path_style`を`true`に変更する必要がある。
Active Storageの場合は`force_path_style`を指定するとよい。

RailsからMinioへファイルは問題なくアップロードできるし、開発用のPCはどちらのネットワークにもアクセスできるので表示もできていた。

このときアップロードした画像のURLを見てみた:

```html
<img src="http://192.168.2.2:9000/test-app/uploads/person/avatar/1/john.jpg?X-Amz-Expires=600...">
```

当然といえば当然なのかもしれないが、URLが`minio`のLAN側のアドレスを指しているので通常のネットワークからは表示できない。

どうやらCarrierwaveの[`asset_host`](https://stackoverflow.com/a/13109245)を指定する必要があるらしい:

```ruby
CarrierWave.configure do |config|
  config.fog_credentials = {
    provider:              'AWS',
    aws_access_key_id:     ENV['FOG_ACCESS_KEY_ID'],
    aws_secret_access_key: ENV['FOG_SECRET_ACCESS_KEY'],
    use_iam_profile:       false,
    endpoint:              'http://192.168.2.2:9000',
    path_style:            true
  }
  config.fog_directory  = 'test-app'
  config.fog_public     = true # 重要
  config.asset_host     = 'https://assets.example.com'
end
```

最初は単純に`asset_host`のみ変更したのだが、変更が反映されずにずいぶん悩んだ。
それから`fog_public`を`true`に変更する必要があった。
画像のURLにパラメータが付与されなくなるので、直接URLで参照できるようになってしまうのは注意する必要がある。

```html
<img src="https://example.com/uploads/person/avatar/1/john.jpg">
```

さて、ここでも随分悩んだのだけれどもこのURLのおかしい箇所に気づいただろうか。
何故か`asset_host`を指定するとbucketがURLに含まれなくなってしまうようだ。

```ruby
CarrierWave.configure do |config|
  config.fog_credentials = {
    provider:              'AWS',
    aws_access_key_id:     ENV['FOG_ACCESS_KEY_ID'],
    aws_secret_access_key: ENV['FOG_SECRET_ACCESS_KEY'],
    use_iam_profile:       false,
    endpoint:              'http://192.168.2.2:9000',
    path_style:            true
  }
  config.fog_directory  = 'test-app'
  config.fog_public     = true
  config.asset_host     = 'https://assets.example.com/test-app' # 重要
end
```

```html
<img src="https://example.com/test-app/uploads/person/avatar/1/john.jpg">
```

これはバグっぽい挙動なのでいずれ余裕があればCarrierwaveかFogか、あるいはAWS側のライブラリかはわからないがPRを作りたいと思う。
文章にまとめてしまうとあまり苦労が伝わらなかったかもしれないが、最初は`CORS`関連のエラーかと思ってNginxProxyManagerのヘッダーを変更したがうまくいかず随分悩んだ。
最初にわざわざネットワークの図を用意したのはNginx側のヘッダーに時間を費やしたからだ。
最終的には全く関係なかったけれども。
