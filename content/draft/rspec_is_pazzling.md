---
title: "RSpecはパズルである"
date: 2024-08-02T07:19:12+09:00
description: ""
tags:
- rspec
draft: true
---

趣味で普段書くプログラムは極力Rubocopのガイドに沿うようにしている。

個人的にはその反動でRubocop Omakaseなどを使ったり、単純にRufoといった軽いフォーマッタを使うこともある。

私の本来の目的は実用的なプログラムを書くことのはずなのだが、Rubocopの意見にいちいち耳を傾けるとプログラムを書くことより、むしろ設計が目的になっている錯覚に陥る。

今回はまさにそんな心境について書いてみようと思う。

## テストダブルについて

一番最初に私がテストダブルについて知ったのはレビューコメントだったと思う。

そのコメントはかつて私が働いていた会社のクローズドソースなので引用はできないのだが、こういった処理は`allow`を使うとよいですよというコメントを見たときだ。

私はRSpecをそれまでも使ってはいたのだが、いわゆる`assert`ベースの書き方が多くてダブルを使わないコードに関してはごまかしのコードを使っていた:

```ruby
class MyHttpClient
  include HTTParty

  def self.fetch_some_page(page)
    if Rails.env.test?
      return "fake response"
    end

    get("/foo/#{page}").body
  end
end
```

この手のHTTPのライブラリには専用のモックやスタブが使えるライブラリがまた別に存在することもあるのだが、必ずしもそういったものが存在するとは限らない。

```ruby
class MyHttpClient
  include HTTParty

  def self.fetch_some_page(page)
    get("/foo/#{page}").body
  end
end

describe MyHttpClient do
  before do
    allow(MyHttpClient).to receive(:fetch_some_page)
      .with(page).and_return("fake response")
  end
end
```

`allow`コマンドを使うことでこの`MyHttpClient`というクラスで定義したメソッドとは全く別のメソッドを上書きできるので、元のコードはそのままにそのテストを書くことができる。

RSpecの書籍を購入してもモックやダブルといった概念は当時理解しきれなかったので、この`allow`と`double`を知ってからは世界が一気に変わった。

## rubocopとの葛藤

モックやスタブ、あるいはテストダブルという概念は使っていくと造詣が深まっていく。

`allow`という概念を知って、それを実際に使いこなせるようになるまではやはりそれなりに時間を要した。

例えばこういうケース:

```ruby
class MyHttpClient
  def fetch_some_page(page)
    self.class.get("/foo/#{page}").body
  end
end
```

この場合はクラスメソッドではなく、インスタンスメソッドであるので次のようなコードを書く:

```ruby
describe MyHttpClient do
  before do
    allow_any_instance_of(MyHttpClient).to receive(:fetch_some_page)
      .with(page).and_return("fake response")
  end
end
```

こうするとrubocopが次のような警告を発する。

```
C: RSpec/AnyInstance: Avoid stubbing using allow_any_instance_of.
    allow_any_instance_of(MyHttpClient).to receive(:fetch_some_page)
    ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
```
