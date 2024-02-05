---
title: "Github Actions であるディレクトリ配下に存在するディレクトリごとに同じJobを回す"
emoji: "🤖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["githubactions"]
published: false
---

# 背景

あるプロジェクトにおいて、以下のようなディレクトリ構成となっており、
```
📁path
  📁to
    📁foo
    📁bar
    📁baz
```
ここで`/path/to`配下の`foo`,`bar`,`baz`それぞれに対し同じ Job を Github Actions で回したいということがあり。対応した。
やりたいことはシンプルだが、意外と時間かかったこともあり、メモと共有を兼ねて記事にすることにした。
