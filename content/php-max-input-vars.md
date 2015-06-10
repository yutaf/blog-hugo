+++
date = "2015-05-19T14:55:52+09:00"
draft = false
title = "php max_input_vars makes you lose form values"
tags = ["php"]

+++

<!--more-->

ie(11) でのみ form で送られるべき値が送られてこないということが起きた。  
最初は js のバグかと思っていたが、php の設定が問題だった。  

[【php】max_input_varsの影響でフォームの値を全部受け取れないことがある   at softelメモ](https://www.softel.co.jp/blogs/tech/archives/3591)

```
問題
フォームから送信した値が、サーバー側（php）で取得できない。

プログラムは動いてるみたいなんだけど、あるはずの $_REQUEST[‘hoge’] などがないみたい。

どうなってるのこれ？

答え
以下の条件に当てはまるようなら、phpのmax_input_varsの設定値の影響かもしれない。

...

max_input_vars integer

入力変数を最大で何個まで受け付けるかを指定します 
(この制限は、スーパーグローバル $_GET、$_POST そして $_COOKIE にそれぞれ個別に適用されます)。
```

対象サーバーの `max_input_vars` は 150 だったので、300 まで上げた。  
解決した。  

それにしても ie(11) だけで起こった理由がわからない。  
他のブラウザよりも多くのフォーム値を送信しているということ？

同じ実装ではないが、次のような記事を見つけた。

[inputタグのonclickでform.submit()を実行しているため、IEだけ二重にリクエストが送信される時の対処法](http://hack.aipo.com/archives/5998/)

この対処法は自分の場合は有効でなかった。  
とりあえず、`max_input_vars` の値には気をつける。  
