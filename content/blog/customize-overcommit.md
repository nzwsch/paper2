---
title: "Customize Overcommit"
date: 2024-06-08T02:00:40+09:00
description: ""
tags:
- overcommit
---

GitHubのCopilotを使い始めてから私のコーディング作業は劇的に変化した。

LLMの生成するコードの品質が高いのもあるが、それ以上にCopilotはコミットメッセージを自動で生成してくれるのが特に便利だ。

```
076f44d Add _scraping.html.erb
9cd992a Add scraping_mailer.rb
4e36d00 Add create_scraping_job.rb
389617c Add scrapings_controller.rb
d320ffc Add _header.html.erb
```

これは私のあるリポジトリから抜粋したコミットメッセージだ。
[xkcdのコミック](https://xkcd.com/1296/)ほど雑ではないが、コミットメッセージが機能していない点は共通している。

純粋にこれらのコミットログを見てみると該当のファイル名の他にファイルを追加していたり、単純にAddよりもMofifyやFixという単語のほうがふさわしそうなものが多い。
特に個人で開発しているのでこの手のコミットメッセージを精査してもあとからこれらのメッセージが活きることはまずない。

それがCopilotを導入してからコミットはボタンを押して数秒待つだけでよくなった。

依然としてコミットメッセージを見返す機会はおそらくないのだが、私はこれに毎月10ドルの価値を見出している。

```
aec309a Refactor VideoEntry model to use download_video_file method instead of create_video_file_with_href_and_label
00354ec Add lazy loading for placeholder images in video_entries.js and include javascript file in index.html.erb
402603c Add javascript files to asset pipeline configuration
e1cc44c Add jquery-rails gem to Gemfile and Gemfile.lock
e8d7384 Update thumbnail display in video_entries/index.html.erb
```

こちらも実際に私がCopilotを利用したリポジトリから抜粋している。
少なくともコミットメッセージの内容をみるにやや冗長な表記のときもあるが、先述したコミットメッセージと比べると誰が見ても雲泥の差である。

前置きが長くなってしまったのだが、Copilotを使っていて不満も当然ある。

それは稀にコミットメッセージが重複することと、Conventional Commitsのメッセージになったことだ。

前者はコミットの内容が全く異なるにも関わらず全く同じメッセージが出力されるときがある。

[Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/)はかいつまむとコミットメッセージに`feat:`とか`chore:`みたいな見出しをつけようという運動のように思う。
個人的にはあまり必要性を感じないどころか、不要なので排除できるのならば排除したい。

不満点はあるものの、よくも悪くも気にしなければ済むのだがブログをつけ始めたこともあり重い腰をあげることにした。

一般的にはコミット前にフックを実行するのは`.git/hooks`の中にあるファイルを編集すればよいが、今回はタイトルにあるようにOvercommitを使うことにした。
同様のプロジェクトでHuskyもあるが、Rubyで書きたかったからだ。

overcommitをインストールして次のファイルをコピーする:

```yaml
# .overcommit.yml
plugin_directory: 'lib/overcommit'

CommitMsg:
  RemoveCommitTag:
    enabled: true
  DuplicateCommitMsg:
    enabled: true
```

```ruby
# lib/overcommit/commit_msg/remove_commit_tag.rb
module Overcommit::Hook::CommitMsg
  # Removes commit tag like "chore: " from commit message
  class RemoveCommitTag < Base
    def run
      commit_message = File.read(commit_message_file)

      File.open(commit_message_file, "w") do |file|
        file.write(commit_message.gsub(types_regex, ""))
      end

      :pass
    end

    def types_regex
      types = %w[build chore ci docs style refactor perf test feat fix]
      /^(#{types.join("|")}): /
    end
  end
end
```

```ruby
# lib/overcommit/commit_msg/duplicate_commit_msg.rb
module Overcommit::Hook::CommitMsg
  # Fail if commit message contains a duplicate commit message
  class DuplicateCommitMsg < Base
    def run
      commit_message = File.read(commit_message_file)
      commit_message = commit_message.split("\n").first.to_s

      result = execute(["git", "log", "--pretty=format:%s", "-n", "10"])
      commit_messages = result.stdout.split("\n")

      if commit_messages.include?(commit_message)
        [:fail, "Duplicate commit message: #{commit_message}"]
      else
        :pass
      end
    end
  end
end
```

まだ運用して数コミットだけれども、大枠はこれでよいと思う。
可読性はよいがだんだんOvercommitを使わなくても良い気がしてきたので、Rubyのみで実行できるスクリプトでもよいかもしれない。
