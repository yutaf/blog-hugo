+++
date = "2015-02-17T12:32:23+09:00"
title = "phpbrew memo"

+++

ローカル開発環境に phpenv + php-build を使っていたが、[phpbrew](https://github.com/phpbrew/phpbrew) のほうが簡単そうだったので移行した。

#### environment

* osx 10.9.5
* Apache/2.2.29
* mysql 5.6.15

#### メリット

* apache の php モジュールをバージョン毎に保存してくれる。
* ちゃんと動く pcre ライブラリが入る。  
今回、apache とともに php をソースからビルドすることも試したが、ビルド時に両方で指定した brew インストールの pcre ライブラリが apache の php モジュールには正しくリンクされず、違うバージョン(apache 同梱のもの?)になっていた。  
結果、preg_replace が動作しなかった。  
その他にも `--enable-intl` など linux に比べて osx では php のビルドがまともにいかないことが多いが、phpbrew はその点を補ってくれる。  

#### デメリット

* 使用するのに php5.3 以上が必要
* php のバージョン切り替えで挙動が不安定な時がある
* configure option を variants という独自の仕組みで指定する。(通常の指定のみでもビルドできるが、variants を使ったほうが良い部分がある)

## Requirement

<https://github.com/phpbrew/phpbrew/wiki/Requirement>

phpbrew を利用するには php が必要。  
osx はデフォルトで php がインストールされているので、osx ユーザーにはいいと思う。

## インストール

```
$ curl -L -O https://github.com/phpbrew/phpbrew/raw/master/phpbrew
$ chmod +x phpbrew
$ mv phpbrew /usr/local/bin/
```

## configure option の設定と php のインストール

variants という仕組みを利用して configure option を設定し、インストールする。

```
$ phpbrew install 5.3.10 +pdo +mysql +pgsql +apxs2=/usr/bin/apxs2
```

通常の configure option の記述をする場合

```
$ phpbrew install 5.3.10 +mysql +sqlite -- \
    --enable-ftp --apxs2=/opt/local/apache2/bin/apxs
```

#### ファイルによる configure option の設定

yamlファイルで独自の variants を設定できる。

<https://github.com/phpbrew/phpbrew/wiki/Setting-up-Configuration>

config.yaml

```yaml:config.yaml
variants:
  dev:
    bcmath:
    mbstring:
    intl:
    icu:
      - --with-icu-dir=/usr/local/opt/icu4c
    gettext:
      - --with-gettext=/usr/local/opt/gettext
    pcre:
    readline:
    xml:
      - --with-libxml-dir=/usr/local/opt/libxml2
    soap:
    zlib:
      - --with-zlib=/usr/local/opt/zlib
      - --with-zlib-dir=/usr/local/opt/zlib
    gd:
      - --with-gd
      - --with-jpeg-dir=/usr/local/opt/jpeg
      - --with-png-dir=/usr/local/opt/libpng
      - --with-freetype-dir=/usr/local/opt/freetype
      - --enable-gd-native-ttf
      - --enable-gd-jis-conv
    openssl:
    mcrypt:
    curl:
    mysql:
    pdo:
    my-exif:
      - --enable-exif
    my-config-file-path:
      - --with-config-file-path=/Users/yutaf/Sync/www/php.ini
extensions:
  dev:
    xdebug: stable
```

yamlファイルを作成後、以下のコマンドを実行する。

```
$ phpbrew init -c=/path/to/config.yaml
```

ファイルに記述されている `+dev` variants を使用できるようになる。

```
$ phpbrew -d install 5.4.36 +dev
```

#### 最終的に辿り着いたインストール方法

```
$ phpbrew -d install 5.4.36 +neutral +apxs2=/opt/apache2.2.29/bin/apxs +dev
```
`+neutral` を指定しないと `--disable-all` 等のオプションが自動的に設定される。  
`--disable-all` は phpのデフォルトで有効な json や xml モジュール等が無効になるので、これらの関数が使用できなくなる。  

`'+apxs2'` は apache の php モジュールをバージョン毎に管理する為に必須(後述)。

## php のバージョン切り替え

```
$ phpbrew switch php-5.4.36
```

## xdebug インストール

上の yaml ファイルに記述してあるので、以下でインストールできる。

```
$ phpbrew ext install +dev
```

## apache のphpモジュール切り替え

variants の `+apxs2` を設定すると各バージョンごとにモジュールを保存し、
httpd.conf に `LoadModule php5_module ...` の記述がされる。

<https://github.com/phpbrew/phpbrew/wiki/Cookbook#apache2-support>

ただし、その際 conf と modules フォルダのパーミッションを 777 に変更するよう言われる。

```
$ phpbrew install 5.3.29 +apxs2=/opt/apache2.2.29/bin/apxs
$ phpbrew install 5.4.36 +apxs2=/opt/apache2.2.29/bin/apxs
```

* 作成されるモジュール

```
/opt/apache2.2.29/modules/libphp5.3.29.so
/opt/apache2.2.29/modules/libphp5.4.36.so

```

* httpd.conf

```apacheconf:/opt/apache2.2.29/conf/httpd.conf
...

LoadModule rewrite_module modules/mod_rewrite.so
LoadModule php5_module        modules/libphp5.4.36.so
LoadModule php5_module        modules/libphp5.3.29.so

...
```

##### バージョン切り替えの方法

httpd.conf で使用するバージョンのモジュール以外をコメントアウトして apache を再起動する

```apacheconf:/opt/apache2.2.29/conf/httpd.conf
...

LoadModule rewrite_module modules/mod_rewrite.so
LoadModule php5_module        modules/libphp5.4.36.so
#LoadModule php5_module        modules/libphp5.3.29.so

...
```

```
$ sudo apachectl graceful
```
## 感想

気持ち悪い所も多いが、osx でphpをビルドするならこれが一番楽な気がした。
