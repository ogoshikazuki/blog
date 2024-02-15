---
title: "Renovate を Mend-hosted から Self-Hosting with GitHub Actions に変更した"
emoji: "🔄️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["renovate"]
published: true
---

# はじめに

今関わっているプロジェクトで、依存パッケージの更新ツールに[Renovate](https://github.com/renovatebot/renovate)を採用しており、元々Renovateが提供している[Mend-hosted GitHub App](https://github.com/apps/renovate)を使用していたが、GitHub ActionsによるSelf-Hostingに移行した。
理由は、Mend-hosted GitHub Appは30分のタイムアウトが設定[^1]されており、その時間内に処理しきることができなかったのと、GoのPrivate Packageの更新がMend-hosted GitHub Appではできなかったから。
Self-Hostingへの移行の際、つまずきポイントがいくつかあり、そこそこの時間を要してしまったため、同じことになる人が1人でも減ればと思い、記事にするにした。

[^1]: https://github.com/renovatebot/renovate/discussions/26589

# つまずきポイントとその対応

## 1. branchPrefixとbranchPrefixOldを変更する

今回、[Mend-hosted GitHub App](https://github.com/apps/renovate)の稼働と並行してSelf-Hostingの動作検証をしていた関係で、移行前のRenovateのPRと動作検証のPRが同時に立つことになった。この時、Renovateは同じ内容のPRが立っている場合、以下の挙動をする性質がある。
- PRが新しく立てず、既存のPRを更新する
- もしPRが他のユーザーに更新されていた場合、何もしない

ここで、Mend-hosted GitHub AppとGitHub ActionsによるSelf-Hostingはユーザーが異なるため、後者の何もしないという動きをする。これを防ぎ、新たにPRを立てるためには、[branchPrefix](https://docs.renovatebot.com/configuration-options/#branchprefix)と[branchPrefixOld](https://docs.renovatebot.com/configuration-options/#branchprefixold)を変更すれば良い。これにより、ブランチが異なる為、「同じ内容」と判定されず、新規にPRを立てるようになる。

## 2. requireConfig: "ignored"を設定する

Renovateの設定項目は、[Repository config](https://docs.renovatebot.com/configuration-options/)と、[Self-hosted config](https://docs.renovatebot.com/self-hosted-configuration/)に分かれる。そして、設定の仕方は、[Global config](https://docs.renovatebot.com/getting-started/running/#global-config)と、repository config fileに分かれる。それぞれの対応は、以下の通り。

||Repository config|Self-hosted config|
|-|-|-|
|Global config|設定可能|設定可能|
|Repository config file|設定可能|設定不可|

Renovateの実行主体が、複数のリポジトリを実行対象とするなら、Global configとRepository config fileに分けることになるが、今回は対象のリポジトリ内でGithub Actionsを回して、そのリポジトリでのみRenovateを実行するので、Global configのみとした。
この場合、Repository config fileは不要となるが、RenovateはデフォルトではRepository config fileが必須となるので、これを無視するために、[requireConfig](https://docs.renovatebot.com/self-hosted-configuration/#requireconfig)を`ignored`に設定する必要がある。

## 3. ignorePresetsを設定する

Renovateには設定のpresetが数多く存在するので、これをうまく活用し、個別設定が必要な部分のみを追加で設定する、というのが基本になる。ここで、presetで設定されるものは、追加で設定したものでオーバーライドされるというのが期待される挙動ではないだろうか。
実際、Mend-hosted GitHub Appでは想定通りの挙動だった。が、GitHub ActionsによるSelf-Hostingでは何故かオーバーライドされないという挙動をした。Renovateのバージョンは合わせていたので何故こうなるのかは今でも不明。
しかし解決策は用意されていて、無視したいpresetは[ignorePresets](https://docs.renovatebot.com/configuration-options/#ignorepresets)を指定すれば良い。

# 最後に

今回紹介したつまずきポイントは、いずれもDebugログを注意深く読めば解決できる内容だった。しかし、数万行にも及ぶDebugログから必要な情報を探すのはなかなか骨の折れる作業だった。この記事が1人にでも役に立つと嬉しい。
