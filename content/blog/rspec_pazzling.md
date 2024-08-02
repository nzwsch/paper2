---
title: "RSpecはパズルである"
date: 2024-08-02T07:19:12+09:00
description: "Rubocopのガイドに従う一方で、個人的なプログラム作成やRSpecのモックやスタブの実践では、より柔軟な方法を模索しつつも、Rubocopの制約や警告に対する葛藤を抱えながら改善を試みている。"
tags:
- rspec
---

趣味で普段書くプログラムは極力Rubocopのガイドに沿うようにしている。

個人的にはその反動でRubocop Omakaseなどを使ったり、単純にRufoといった軽いフォーマッタを使うこともあった。[^1]

[^1]: 最近は制約が多い言語が増えているが、Rubyは調整できてよい

私の本来の目的は実用的なプログラムを書くことのはずなのだが、Rubocopの意見にいちいち耳を傾けるとプログラムを書くことより、むしろ設計が目的になっている錯覚に陥る。

今回はまさにそんな心境について書いてみようと思う。

## テストダブルについて

一番最初に私がテストダブルについて本格的に調べ始めたのはとあるレビューコメントがきっかけだった。

そのコメントはクローズドソースなので引用はできないのだが、こういった処理は`allow`を使うとよいですよというコメントを見たときだ。

私はRSpecをそれまでも使ってはいたのだが、railsのscaffoldで生成したspecファイルのような書き方をなぞる程度の書き方しかしていなかった。

Rubyに限らず、もしHTTPリクエストのようにテストをしにくい概念は次のように書いていた:

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

モックやスタブ、あるいはテストダブルという概念を実践しだすとRSpecに対する理解も深まっていく。[^2]

[^2]: モックに依存しすぎることの反対意見があるようだが、単体テストは結果より設計に重きを置くべきだと思う

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

`allow_any_instance_of`を使うかわりに次のコードに書き換えよとのことである:

```ruby
describe MyHttpClient do
  let(:my_http) { instance_double(described_class) }

  before do
    allow(described_class).to receive(:new).and_return(my_http)
    allow(my_http).to receive(:fetch_some_page)
      .with(page).and_return("fake response")
  end
end
```

なぜせっかく`allow_any_instance_of`があるのにこんな回りくどいコードが推奨されるのかというと、スタブのスコープを最小にするのが意図なのだと思う。

上記のコードで厄介なバグが発生とかは記憶にないが、このようにRubocop自体がかなりオピニオンベースでここはこうしろ、ああしろと指図してくる。

幸いCopilotがあるので修正自体は楽ではあるが、この手の書き換えはある程度書き手が例示しないと直せない。

`allow_any_instance_of`の警告は無視したりすることもあるが、次第に対応できるならしてみようと思えてくる。

## receive_chain_message

最近知った`receive_chain_message`というメソッドも便利である。

例えばこのようなコードに使う:

```ruby
class Post
  scope :published, -> { where(published: true) }
end

describe Post do
  let(:post) { create(:post) }

  before do
    allow(Post).to receive_message_chain(:published, :find_each)
      .and_yield(post)
  end
end
```

`Post.published.find_each`を実行したときに、実際のクエリではなくモックした`post`を返す必要があるときだ。

```
C: RSpec/MessageChain: Avoid stubbing using receive_message_chain.
    allow(Post).to receive_message_chain(:published, :find_each)
                   ^^^^^^^^^^^^^^^^^^^^^
```

`allow_any_instance_of`もそうだが、この手の便利な書き方はおよそRubocopでおよそ許容されない。

知ったばかり、使い始めたばかりの瞬間に却下されるのはなんだか複雑だ。

そしてこの警告の内容に沿って修正するのであればこのように書かなければならない。

```ruby
describe Post do
  let(:post) { create(:post) }

  before do
    published = instance_double(Post::ActiveRecord_Relation)
    allow(described_class).to receive(:published).and_return(published)
    allow(published).to receive(:find_each).and_yield(post)
  end
end
```

ここまで来ると警告はないものの何を書いているのかだんだんわからなくなってくる。

これは単純に`Post.published`というスコープだからよいが、ではもしここに`Post.featured.published.latest`などのようにさらに増えたらと思うとぞっとする。

あまり納得がいかなかったので、検索してみた。

> Another option is to extract a named method, and then stub that instead.
> [RSpec + Rubocop - why receive_message_chain is a code smell?](https://stackoverflow.com/a/53941389)

意訳すると別のテストしやすいメソッドを用意するべきという回答なのだけれども、まさに目から鱗が落ちるようだった。[^3]

[^3]: なお興味深いのは、この回答がおよそ6年前に行われていたということである

上記のコメントをそのまま受け取るとおそらくはこのように修正する:

```ruby
class Post
  scope :published, -> { where(published: true) }

  def self.with_published(&block)
    published.find_each(&block)
  end
end

describe Post do
  let(:post) { create(:post) }

  before do
    allow(described_class).to receive(:with_published).and_yield(post)
  end
end
```

> This has the added benefit of making the code a little clearer by naming a concept. 
> (There's nothing preventing you from adding this method even if you use a test fixture.)

このコードを追加したことでテストも理解しやすくなる。

実際に私のコードは`Post.published.find_each`の挙動を書き換えるのではなく`yield`側のコードを書き換えて対応したが、可読性は高まったと思う。

他にもRSpecはやれ`describe`のネストは深くしすぎるなとか、`let`は5個以上定義するなと、まともにrubocopに対応すると制約が多くてうんざりする。

趣味でコードを書く利点は時間的な制約はないので、じっくりとこの問題に向き合うことができる。

そもそもRSpecは何も工夫をしないと本体のコード以上に難解なコードになりやすい。

rubocopの意見に耳を傾けてみると、RSpecの書き方は一定のパターンがなんとなく見えてくる。

いくつかのプログラミング言語ではゴルフに例えられるが、RSpecはさながらパズルのようなものである。
