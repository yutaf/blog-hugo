+++
date = "2015-06-11T13:10:53+09:00"
draft = false
title = "Continuous integration with gitbucket and jenkins"
tags = ["deploy", "gitbucket", "jenkins"]

+++

<!--
Have you ever thought of these kind of things ?  

- **We want to develop with github, but our codes must be placed in private network...**  
- **Github Enterprise is too expensive...**  
- **We want to run continuous integration with Travis CI or CircleCI to catch `pull request` events and build. But github is needed to use those services...**

I have no idea to bring your team github.  
But I found that we can do almost same things all in your private zone with `GitBucket` + `jenkins`.  
-->

<!--
GitBucket is OSS almost like GitHub, easy to use and free.  
Jenkins can catch pull requests with its plugins.  
-->

<!--more-->

伊藤直也さんが書いた [GitHub 時代のデプロイ戦略](http://d.hatena.ne.jp/naoya/20140502/1399027655) という記事がある。  

`GitHub のイベントを契機に CI as a Service にデプロイを担当させる`

去年この記事を読んだときは、先端を行っている会社はこんなことをやっているのか、すごい、としか思えなかった。  
そもそも `GitBucket` を使っている自分の会社では無理だろうと。  

しかし、GitBucket をしばらく運用し、プルリクエストを覚え、jenkins でテストを回そう、ということを進めていくうちに、`GitBucket` + `Jenkins` で案外ここに書いてあることと近いことができるのが分かってきた。

## requirement

- [GitBucket](https://github.com/takezoe/gitbucket)
- [Jenkins](https://jenkins-ci.org/)
- [GitHub pull request builder plugin](https://wiki.jenkins-ci.org/display/JENKINS/GitHub+pull+request+builder+plugin)

## GitHub pull request builder plugin

github のプルリクエストを契機にJenkins に自動マージ、任意のジョブを実行させることができるプラグイン。  
実際にはgithub に限らず、GitBucket でも使うことが出来る。  


<!--
また、Jenkins の Job 設定の際に使用できる `Environment Variables` があり、どのブランチからどのブランチに pull request が送られたかもわかるようになっている。  
以下、プラグインのドキュメントより引用。  

```
Environment Variables
The plugin makes some very useful environment variables available.

・ ghprbActualCommit
・ ghprbActualCommitAuthor
・ ghprbActualCommitAuthorEmail
・ ghprbPullDescription
・ ghprbPullId
・ ghprbPullLink
・ ghprbPullTitle
・ ghprbSourceBranch
・ ghprbTargetBranch
・ sha1
```
-->

## Settings

<https://github.com/takezoe/gitbucket/wiki/Setup-Jenkins-GitHub-pull-request-builder-plugin>

リンクに従って、設定を行う。

GitBucket と Jenkins の url は以下。  
GitBucket url : `http://gitbucket.sample.co.jp/`  
Jenkins url : `http://192.168.10.11:8080`  

- jenkinsbot ユーザーを gitbucket に作成  
  jenkinsbot ユーザーが対象のレポジトリにアクセスできるように、Group に入れたり、Collaborators に入れたりしておく
- gitbucket レポジトリ `http://gitbucket.sample.co.jp/root/test` を作成
- gitbucket レポジトリのwebhook url に `http://192.168.10.11:8080/ghprbhook/` を設定。  
  <img src="/images/continuous-integration-with-gitbucket-and-jenkins-01.png" class="image">
- `http://gitbucket.sample.co.jp/jenkinsbot/_application` で Personal access tokens を作成
- Jenkins の設定ページで GitHub Pull Request Builder のセクションを設定する  
  `http://192.168.10.11:8080/configure`  
  <img src="/images/continuous-integration-with-gitbucket-and-jenkins-02.png" class="image">

  - GitHub server api URL: `http://gitbucket.sample.co.jp/api/v3`
  - Access Token: 上で取得したものを入力
  - 保存
- Jenkins でジョブを作成  
  testjob を作成  
  `http://192.168.10.11:8080/job/testjob/`
- ジョブの設定  
  `http://192.168.10.11:8080/job/testjob/configure`
  - GitHub project : `http://gitbucket.sample.co.jp/root/test`
  - ソースコード管理  
      - Git
      - Repository URL: ssh://jenkinsbot@gitbucket.sample.co.jp/root/test.git
      - Credentials : 有効なSSH キーを設定  
      高度な設定を開く  
      - Refspec : `+refs/pull/*:refs/remotes/origin/pr/*`  
      - Branch Specifier (blank for 'any') : `${sha1}`
  - Build trigger
      - GitHub Pull Request Builder
      - Admin list = `root`
  - 保存する
  <img src="/images/continuous-integration-with-gitbucket-and-jenkins-03.png" class="image">  
  <img src="/images/continuous-integration-with-gitbucket-and-jenkins-04.png" class="image">  


## GitBucket のプルリクエストでテストを実行

ここまで出来たら GitBucket 上で pull request を作成すると、jenkins でブランチの自動マージが行われ、その後 job (テスト) が実行される。  

<img src="/images/continuous-integration-with-gitbucket-and-jenkins-05.png" class="image">  

## Test

どのようなテストを行っているか。  

Jenkins に、自動マージされたブランチのソースコードをデプロイ対象ホストの適当なディレクトリにデプロイさせる。  
もちろん、本来のデプロイのパスではない、影響のないパスに。  
その後、デプロイ対象ホストで unit テストを実行する。  

そうすることで pull request マージ後のソースコードがデプロイ対象ホストで正しく動くかを確認できる。  
自分は db のテストを中心に書いていて、各デプロイ先ホストで db テーブルが正しく作られているか等を確認している。  

## Merged ? or Closed without merge ?

後はテストの結果を見て、  

- 成功したらpull request をマージして Jenkins から自動デプロイする
- マージされずに close されたら Jenkins は何もしない

というふうに進めたい。  
しかし、そこまで出来ていない。  
なぜなら、GitBucket で pull request がマージされたか、マージされずに close されたかが Jenkins 側で判別できない為。(2015.06 時点)  

GitBucket で pull request がマージ、もしくは close されたタイミングで Jenkins に通知を送ることは可能で、GitBucket のレポジトリに以下の様な webhook url を設定すればできる。  

`http://192.168.10.11:8080/job/testjob/buildWithParameters`

通知はjson 形式で Jenkins にPOST送信され、その中にはレポジトリやブランチ、git のsha 1 等が含まている。  
しかし、肝心の pull request がマージされたのか、マージされず close されたのかが分かる情報が含まれていない。。  

その為、今はマージした後にブランチを手元に pull してから デプロイを行っている。  

GitBucket のソースコードを調べてみると、マージを判定する値の部分がコメントアウトされていた。  
いずれのアップデートで含まれるようになるのだろうか。。  

<https://github.com/takezoe/gitbucket/search?l=scala&q=merged&type=Code&utf8=%E2%9C%93>


## 感想

今まで、手元でたまに行っていたテストを pull request をきっかけに自動的に実行出来るようになったので、安心感が増した。  
これをきっかけにテストを今までよりちゃんと書こうと思う。

## 参考

[JenkinsプラグインのGitHub pull request builder pluginを使ってみる](http://d.hatena.ne.jp/oovu70/20130118/p1)
