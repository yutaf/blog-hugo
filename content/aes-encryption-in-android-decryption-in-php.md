+++
date = "2018-03-13T10:44:36+09:00"
title = "Aes encryption in android, decryption in php"
draft = false
tags = ["android", "php", "encryption"]

+++

<!--more-->

## Notes

- PHP Mcrypt is deprecated as of PHP 7.1.0, removed as of PHP 7.2.0
<http://php.net/manual/en/intro.mcrypt.php>
- I could not decrypt in alternative way([openssl_decrypt](http://php.net/manual/en/function.openssl-decrypt.php)) instead of Mcrypt.

## References
- <https://developer.android.com/reference/android/util/Base64.html#URL_SAFE>
- <https://github.com/stevenholder/PHP-Java-AES-Encrypt>

```
class Encryption {

    ...

    /**
     * Decrypt AES encrypted data by Android
     *
     * @param $sStr
     * @param $sKey
     * @return bool|string
     */
    public static function decryptAesAndroid($sStr, $sKey) {
      // Replace strings escaped by Android's Base64.URL_SAFE
      // https://developer.android.com/reference/android/util/Base64.html#URL_SAFE
      $sStr = str_replace(array('-', '_'), array('+', '/'), $sStr);

      // Borrowed from https://github.com/stevenholder/PHP-Java-AES-Encrypt
      $decrypted= mcrypt_decrypt(
        MCRYPT_RIJNDAEL_128,
        $sKey,
        base64_decode($sStr),
        MCRYPT_MODE_ECB
      );
      $dec_s = strlen($decrypted);
      $padding = ord($decrypted[$dec_s-1]);
      $decrypted = substr($decrypted, 0, -$padding);
      return $decrypted;
    }
}
```
