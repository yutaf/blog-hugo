+++
date = "2016-01-28T16:24:41+09:00"
draft = false
title = "Domain Status: clientHold"
tags = ["dns", "route53"]

+++

<!--more-->

Route 53 でdnsの設定をした後、何度 `dig` や `nslookup` をやっても反映されないのでおかしいと思ったが、ドメインステータスに問題があるらしい。

```
$ whois everydaymusic.net | grep Status
   Status: clientHold https://www.icann.org/epp#clientHold
   Status: clientTransferProhibited https://www.icann.org/epp#clientTransferProhibited
Domain Status: clientTransferProhibited https://icann.org/epp#clientTransferProhibited
Domain Status: clientHold https://icann.org/epp#clientHold
```

`Domain Status: clientHold`

[参考にしていたページ](http://dev.classmethod.jp/cloud/aws/onamae-to-route53/)だとここが

```
Domain Status: ok
```

と表示されている。  
調べた所、以下のように書いてあった。

<https://muumuu-domain.com/?mode=faq&state=answer&id=000717>

```
Ｑ．【clientHoldとは？】ドメインが急に利用できなくなりました。
取得されているドメインのご契約やネームサーバ設定に問題がないのに、
ホームページの表示や、メールアドレスがご利用いただけなくなった場合、
上位機関による『利用制限(clientHold)』が行われている可能性があります。

・・・

利用制限の解除は、ドメインのWHOIS情報・ムームーIDにご登録の
メールアドレスをご確認いただき、上位機関より送信される、
『 ドメイン情報認証 』へのご対応が必要です。

・・・

■WHOIS情報に「弊社の情報を代理公開する」を設定されている場合

コントロールパネルへログイン後、左方メニュー項目の
「[その他]個人設定 > ユーザー登録情報」から、
最下部の『 ご連絡先メールアドレス 』が
受信可能なものであるかをご確認ください。

受信可能なメールアドレスでしたら、そのアドレス宛に、
以下のメールが送信されている可能性がございます。

-----------------------------
件名：【重要】[お名前.com] ドメイン 情報認証のお願い
または
件名：【重要】[お名前.com] ドメイン利用制限設定　完了通知
または
件名：【重要】[ムームードメイン] ドメイン 情報認証のお願い

送信元メールアドレス：[verification-noreply@onamae.com]
-----------------------------

恐れ入りますが、受信メールBOXや迷惑メールBOXなどに、
届いていないかご確認いただき、メールを確認されましたら、
メール本文内に記載された「URL」へアクセスをお願いいたします。
```

メールボックス調べたら確かに届いていた。  
ドメインはムームードメインで契約したが、このメールはお名前.com から届いていた。  
たぶん、広告かなんかだと思って全く読んでなかった。  

```
From: verification-noreply@onamae.com
Subject: 【重要】[お名前.com] ドメイン 情報認証のお願い
────────────────────────────────── 
■メールアドレスの有効性認証■
────────────────────────────────── 

本メールは、先日お送りしているドメイン登録者（Registrant）情報のメール
アドレス有効性認証をご対応いただけていない方に送信しております。

対応期日までに認証が行われない場合、ドメインの利用が制限されますので、
お早めにご対応いただきますようお願いいたします。

ドメイン登録者情報のメールアドレスとして情報が正しい場合は、以下URLへ
アクセスしてください。

対応期日：2015年07月18日 15:25
https://www.onamae.com/domain/verification?authId=26534&token=pq4CWa3gNp7vbA_9ATkiuRbKt&lang=ja

※メールアドレス以外にも、Whois情報が不正確なドメインは、登録抹消や
　 使用停止の対象となることがございますのでご注意ください。
※各種情報の確認・修正等はご利用のドメイン管理会社へご相談くださいます
　ようお願いいたします。
```

有効期限半年ぐらい過ぎてたけど、メール内のリンク先をクリックしたらドメイン利用制限は無事解除された。

```
$ whois everydaymusic.net | grep Status
   Status: clientTransferProhibited https://www.icann.org/epp#clientTransferProhibited
Domain Status: clientTransferProhibited https://icann.org/epp#clientTransferProhibited
```

`Domain Status: ok` ではないけど、大丈夫なのかな。  

ちなみに、ムームードメインからもこんなメールが来ていた。  

```
From: ムームードメイン news@service.muumuu-domain.com mag.pepabo.com
Subject: 【重要】ドメイン情報認証開始および利用規約改定のお知らせ【ムームードメイン】

このメールはムームードメインのすべてのユーザー様へお送りしています。
------------------------------------------------------------------

平素よりムームードメインをご利用いただき、誠にありがとうございます。

この度、特定のお手続きを行った際に、ICANNのWHOIS情報正確性確認方針に基づき、
ムームードメインの上位レジストラである「お名前.com」（GMOインターネット株式会社）より、
お客様のメールアドレスの有効性を確認するメール（ドメイン情報認証）が
送信されることとなりました。

ドメイン情報認証のメールを受信されましたら、2週間以内に、
メールアドレスの有効性確認のお手続きをお願いいたします。

ドメイン情報認証のお手続きが期限内に行われない場合、
ドメインの利用制限が行われ、該当のドメインを利用したホームページの閲覧や、
メールの送受信ができなくなりますので、必ずお手続きをお願いいたします。

ドメイン情報認証の対象となるお手続きおよび詳細は以下のとおりです。


■ドメイン情報認証が行われるお手続き

　・新規ドメイン取得（一括取得を含む）
　・WHOIS情報のうち、登録者情報（Registrant）の変更
　・ムームードメインへのドメイン移管

※認証はメールアドレスひとつにつき1回です。
　一度認証を行ったメールアドレスについてはその後の認証は行われません。

■ドメイン情報認証メール送信元メールアドレス

　verification-noreply@onamae.com

■ドメイン情報認証メールタイトル

　「【重要】[お名前.com] ドメイン 情報認証のお願い」

■送信先メールアドレス

　○WHOIS情報を弊社情報で代理公開している方
　　・お客様のご連絡先メールアドレスへ送信されます

　○WHOIS情報をお客様の情報で公開している方
　　・登録者情報（Registrant）に記載されているメールアドレスへ送信されます

■ドメイン情報認証方法
　上記タイトルで送信されるメールに記載されているURLをクリックし、
　メールアドレスの有効性認証を行ってください。

■ドメイン情報認証開始日時
　2015年6月30日午前12時以降のお手続き

■ドメイン情報認証の対象となるドメイン

.com/.net/.org/.biz/.asia/.info/.mobi/.xxx/.ceo/.wiki/.yokohama/.xyz/.bar/
.rest/.christmas/.blackfriday/.click/.fishing/.rodeo/.vodka/.hosting/.property/
.diet/.help/.audio/.hiphop/.juegos/.host/.press/.website/.black/.onl/.rich/
.global/.ryukyu/.surf/.club/.nagoya/.tokyo/.sexy/.tattoo/.link/.guitars/.pics/
.gift/.photo/.pink/.red/.blue/.kim/.ink/.okinawa/.cooking/.horse/.bid/.trade/
.webcam/.market/.beer/.uno/.cricket/.science/.casa/.work/.yoga/.vegas/.dance/
.social/.pub/.moda/.consulting/.kaufen/.actor/.rocks/.haus/.party/.fashion/
.garden/.fit/.flowers/.kiwi/.buzz/.country/.wedding/.space/

※日本語ドメインを含みます。
※上記リストはムームードメインで今後取り扱い予定のドメインを含んでいます。


また、ドメイン情報認証の開始に伴い、
『ムームードメイン利用規約』の一部を改定いたします。

━━━━━━━━━━━━
■改定日
━━━━━━━━━━━━
2015年6月30日（火）

━━━━━━━━━━━━
■改定内容
━━━━━━━━━━━━
改定の概要は以下の通りです。

・ムームードメイン利用規約第5条に
　ドメイン情報認証に関する規定を追加（第5条第3項および第4項の新規追加）

利用規約の全文につきましては、下記ページをご確認ください。

▽ムームードメイン利用規約
https://muumuu-domain.com/?mode=kiyaku


これからも、ムームードメインをどうぞよろしくお願いいたします。

------------------------------------------------------------------
◆ご不明な点がございましたらお問い合わせフォームよりご連絡ください
https://muumuu-domain.com/?mode=inq
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
ムームードメイン https://muumuu-domain.com
GMOペパボ株式会社 http://pepabo.com/company/overview/
```

はっきり言って、こういうメールは広告かスパムかだと思って自分のメールフィルターから全く外れてしまっている。。
