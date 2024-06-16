---
title: "Omakub"
date: 2024-06-15T18:20:23+09:00
description: ""
tags:
- essay
- ubuntu
draft: true
---
つい数日前に[Omakub](https://omakub.org/)というプロジェクトを知ったので、早速試してみることにした。
このブログもOmakubをインストールしてTyporaというMarkdownエディタを試している。

Omakubを一言で表せば、モダンな開発環境を用意できるオープンソースのプロジェクトである。

開発環境のオープンソースといえばdotfilesや[Starship](https://starship.rs/)のような設定ファイルをまとめて公開しているのではなく、Ubuntuをインストールしたあとにスクリプトをダウンロードして途中でいくつかのプロンプトを実行してログアウトするだけでよい。これはかつて存在していた[Boxen](https://github.com/boxen/boxen)というプロジェクトに近いかもしれない。

Omakubはおそらく「Omakase + Ubuntu」でOmakubということなのだろう。
私の記憶ではrubocop-rails-omakaseというリポジトリが登場したばかりで、デスクトップ環境も含めてOmakaseしてしまおうというプロジェクトだ。最近よく見かける*Opinionated*なプロジェクトのひとつと思ってよいだろう。

> Package management is only half the battle of getting a great development experience going on Linux, though. 
> (しかし、パッケージ管理は、Linuxで優れた開発体験を得るための戦いの半分に過ぎない。)

> The other half lies in the dotfiles that control the configuration. Linux gets great power from how customizable it is, but that also presents a paradox of choice and a tall learning curve.
> (残りの半分は、コンフィギュレーションをコントロールするドットファイルにある。Linuxはカスタマイズが可能なことから大きな力を得るが、それは同時に選択肢の多さと学習曲線の高さというパラドックスにもなる。)

>  Having good, curated defaults that integrate all the many tools in a coherent feel and look can help more developers acquire a taste for Linux, which they may then later inspire a fully bespoke setup (or not!).
>  (多くのツールを首尾一貫した感触と外観で統合する、優れた、管理されたデフォルトを持つことは、より多くの開発者がLinuxの味を覚えるのに役立つ。)

Omakubで重要なのはデスクトップのカスタマイズをまるごとDHHないし、オープンソースコミュニティにOmakaseしてしまおうというコンセプトにある。従来のプロジェクトではプログラミングに関するソフトウェアは自動でインストールされたが、デスクトップの壁紙やテーマ、フォントに至るところまでOmakubは変更を試みる。

今から振り返ることおよそ20年ほど前、かつてのフィーチャーフォンやWindows XPの時代は個性的な待受画面ないしデスクトップが存在していた。近年のMacやWindowsではそもそもカスタマイズができなくなりつつあるなか、Linuxは自由なままであると認識できる。

そんなLinuxでゼロから自分好みのデスクトップ環境を作るのは非常に骨の折れる作業である。Linuxを使う人は皆Arch LinuxやnixOSのようなディストロを日常的に使いこなせるような人とは限らない。特に私はUbuntuやZorin、Linux MintのようなOSを最低限のカスタマイズで使い続けてきた。

OmakubをインストールするとAlacritty、Neo Vimなどに加えてFuzzy FinderやEZA、BATのようなアプリケーションもすぐに使える。これらのツールは興味はあったものの、今まで一度も試すことができなかった。

それは毎回OSをクリーンインストールするたびにそのツールをインストールしなければならないという手間を考えると、モダンなツールであっても試すことはないのだ。

同様の理由で私は`vim`ではなく`vi`をこよなく愛用してきたが、最低限のカスタマイズだとわざわざ`vim`をインストールしようとまでは思わなかった。一時期は必ず`vim`をインストールしていたが、`vi`でも十分なことに気づいた。

Omakubはもちろんディストロではないが、もはやディストロのように思える。無数に存在するUbuntuベースのディストロでもここまで積極的にプリインストールで「実用的な」状態に仕上がっているものは記憶上にはない。[^1]

[^1]: Linuxユーザーは常に同じOSを使わず、DistroWatchで定期的に知らないOSを嗜む趣味がある

初めて自分がPCを使った頃を覚えているだろうか。家電量販店のいわゆるメーカーPCについてきた無数のプリインストールソフトウェア。はじめてMac OSXに乗り換えて洗練されたUIを持ったアプリケーション。Linuxを試してWindowsやMacを模倣したようなソフトウェアの数々。どれも新鮮な記憶で、Omakubはその時と似たような記憶を思い出せた。

いざ使ってみると普段自分が使っているUbuntuであるにも関わらず思ったように動作せずに正直難しい。例えばAlacrittyではコピーペーストを気軽にはできないし、DHHはOmakubを[Framework](https://frame.work/)で使うことを想定しているためマルチディスプレイ環境だと結局マウス操作からは離れられない。[^2]

[^2]: Frameworkが日本に発送される日をずっと初期の頃から待ち続けているが、円安もあるのか未だに未対応のままである

残念ながらVSCodeを起動すると毎回フリーズするという不具合はあるものの、インストールプロセスでは特に大きい不具合もなかった。DHHが作ったおかげかRubyも3.3.3がすぐに使える状態だった。Issueを見る限りでは[PHP](https://github.com/basecamp/omakub/pull/87)もインストールできるようになるので、Ruby界隈のためのプロジェクトというわけでもなさそうだ。

ただ今後のことを考えると正直私はいくらDHHがメンテナンスを行うとはいえ、このプロジェクトが今後もずっと続くとは考えにくい。

この手のプロジェクトはユーザーが増えれば増えるほど対応しなければならないツールやプログラムの数も増えるだろう。そしてIssueの数も今は1桁で収まっているが、これも今がマイナーだからに過ぎない。

私がこのプロジェクトを見つけた時点で2900個以上のGitHub Starを獲得しているので良くも悪くもこのコンセプトは受け入れられていると考えられる。とはいえ先程の「選択肢の多さと学習曲線の高さ」を考えると、このプロジェクトはRailsに匹敵するほどの知名度は得られるまで続くとも正直考えにくい。

少なくともOmakubはリリースされたばかりのUbuntu 24.04を対象にしているので、LTSがサポートされる期間の2034年ないし2028年くらいまではせめてほそぼそと続いていて欲しいと願う。

しかしこうして「すぐに使える」状態であることも事実である。

私もこれまで何度か自分のdotfilesのリポジトリを作っては挫折してきたが、Omakubのスケルトンの箇所を真似することができればUbuntuで自分だけのデスクトップをカスタマイズする方法の参考にはできそうだ。
