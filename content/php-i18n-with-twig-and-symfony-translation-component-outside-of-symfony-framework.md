+++
date = "2016-06-05T14:47:30+09:00"
draft = false
title = "php i18n with twig and symfony translation component outside of symfony framework"
tags = ["i18n", "php", "twig", "symfony"]

+++

<!--more-->

composer.json

```
{
  "require" : {
    "twig/twig": "~1.0",
    "symfony/translation": "^3.1",
    "symfony/twig-bridge": "^3.1",
    "symfony/config": "^3.1",
    "symfony/yaml": "^3.1"
  },
}

```

php

```
$loader = new Twig_Loader_Filesystem(__DIR__.'/../templates');
$twig = new Twig_Environment($loader);

$translator = new Symfony\Component\Translation\Translator('en_GB', new \Symfony\Component\Translation\MessageSelector());
$translator->setFallbackLocales(['ja_JP']);
$translator->addLoader('yaml', new Symfony\Component\Translation\Loader\YamlFileLoader());
$translator->addResource('yaml',  __DIR__.'/../locales/en.yml', 'en');

$twig->addExtension(new \Symfony\Bridge\Twig\Extension\TranslationExtension($translator));
$data = ['somethig' => 'abcd'];
$twig->render("hello.html", $data);
```

yml

```
foo: FOO
```

html

```
<!DOCTYPE html>
<html>
<body>
<p>{{ 'foo'|trans }}</p>
</body>
</html>
```

output

```
FOO
```

References:  
<https://gist.github.com/2bard/4329452>  
<http://silex.sensiolabs.org/doc/providers/twig.html>  
<http://silex.sensiolabs.org/doc/providers/translation.html>  
