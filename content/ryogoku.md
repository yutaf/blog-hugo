+++
date = "2015-03-25T23:11:48+09:00"
draft = false
title = "I made a deployment tool - ryogoku"
tags = ["deploy", "bash", "ryogoku"]

+++

rsync によるデプロイツール・[ryogoku](https://github.com/yutaf/ryogoku) を作成した。  
I made a deployment tool, [ryogoku](https://github.com/yutaf/ryogoku).  
It uses `rsync` to deploy.
<!--more-->

## why

会社でデプロイの自動化を進めており、何か良いツールはないか探していた。  
ちなみに自分の会社は以下の様な条件での仕事が多い。  

* 受託業務中心。納品するのは php アプリケーション  
* プロダクション・サーバーがクライアントの所有で自由にツール類をインストールできない。git や ruby 等が入っていないことが殆ど。
* web サーバー2台の db サーバー1台という小規模な構成が殆ど。  

これらの条件に見合うようなデプロイツールを探して、色々情報を集めた。  
特に参考になったのはこのページ。  

[開発メモ#1 : Cinnamon によるデプロイ](http://d.hatena.ne.jp/naoya/20130118/1358477523)

そして、最終的には以下の様な理由で自作することにした。  

* capistrano は複雑そう。また、php のアプリで ruby のツールを使いたくなかった。
* fabric も検討したが、デザイナー含めた他のメンバーに homebrew や pip 等のインストール作業をさせるのがハードルだと感じた。
* [deploy](https://github.com/tj/deploy) というシェルスクリプトのデプロイツールを発見し、インストールが楽なこととシンプルな使い方がとても良いと感じた。  
[【個人メモ】デプロイするためにdeployを使ってみる](http://qiita.com/futoase/items/c2ac39cfe28813b79bc4)  
しかし、デプロイ先サーバーでの git インストールが必須な点と、複数台のホストに対して実行出来ない点が自分の目的とそぐわなかった。

以上から、[deploy](https://github.com/tj/deploy) をベースに、殆どの unix マシンで使用可能な rsync 使ったツールを作成することにした。  

I wanted simple deployment tool suitable for my work.  
My work is like  

* making php applications that are outsourced from various clients.
* Production servers are properties of clients, and softwares like git or ruby are rarely installed in them.  
And also I am not permitted to install softwares.
* Server cluster is consisted of 2 web servers and 1 database server in most cases.

I started searching tools to match these conditions.  
But I eventually decide to make my own one after a while surfing the web.  
Because  

* capistrano seemed to be too complicated.
* fabric seemed much better than capistrano, but I thought it was still hard for my team to install homebrew or pip things.
* I found [deploy](https://github.com/tj/deploy) on github.  
that's cool deployment tool because installation and usage were very simpler than others.  
But it requires git to be installed in deployment servers and doesn't support deployment to multiple hosts.  
That didn't match my purpose.

So I decided to make my own one based on [deploy](https://github.com/tj/deploy).  
Thank you, [deploy](https://github.com/tj/deploy).

## What it is like - ryogoku

<https://github.com/yutaf/ryogoku>

### Installation

Clone the repository from github, and

```
$ make install
```

### How it works

<img src="/images/ryogoku-01.png" class="image">

### Configuration

`ryogoku.conf` を git レポジトリルートに作成し、以下の様な内容を書く。  

Create `ryogoku.conf` file which has content like below at the git repository root.  

```
[prod]
user rob
host 128.199.170.128 128.199.244.193
path /var/www/html
ref origin/master
pre-rsync ./bin/pre-rsync
post-deploy /var/www/html/bin/update.sh && /var/www/html/bin/update.prod.sh
umask 002
```

内容について簡単に説明すると、  

* デプロイ先は  
`rob@128.199.170.128:/var/www/html` `rob@128.199.244.193:/var/www/html`
* `ref` はデプロイする git のリビジョンを指定
* `pre-rsync` は rsync を行う前に実行するコマンド
* `post-deploy` は rsync を行ったあとに実行するコマンド
* `umask` はファイルパーミッションを設定する

What directives in this configuration file means are

* deploying to  
`rob@128.199.170.128:/var/www/html` `rob@128.199.244.193:/var/www/html`
* `ref` is the git revision to be deployed.
* `pre-rsync` defines commands that are executed before rsync.
* `post-deploy` defines commands that are executed after rsync.
* `umask` defines the file permissions.


### Execution

```
$ ryogoku prod
```

これで、master リビジョンが prod で指定したホストにデプロイされる。  

`master` revision is deployed to the hosts defined in `prod` section by this command.  
