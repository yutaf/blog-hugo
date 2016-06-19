+++
date = "2016-06-19T13:35:28+09:00"
draft = false
title = "Internationalize symfony validation component outside of symfony framework"
tags = ["i18n", "php", "twig", "symfony"]

+++

<!--more-->

It took me several hours to make it, Symfony validation component Internationalization.  
The point is instantiation of `translator` and `validator`.  


composer.json

```
{
  "require" : {
    "twig/twig": "~1.0",
    "symfony/translation": "^3.1",
    "symfony/twig-bridge": "^3.1",
    "symfony/config": "^3.1",
    "symfony/form": "^3.1",
    "symfony/validator": "^3.1",
  },
}
```

php

```
use Symfony\Component\Translation\Translator;
use Symfony\Component\Translation\MessageSelector;
use Symfony\Component\Translation\Loader\XliffFileLoader
use Symfony\Component\Validator\Validation;
use Symfony\Component\Form\Extension\Validator\ValidatorExtension;

$translator = new Translator($locale, new MessageSelector());

$vendorDir = realpath(__DIR__.'/../vendor');
$vendorFormDir = $vendorDir.'/symfony/form';
$vendorValidatorDir = $vendorDir.'/symfony/validator';

$translator->addLoader('xlf', new XliffFileLoader());
// there are built-in translations for the core error messages
$translator->addResource('xlf', $vendorFormDir.'/Resources/translations/validators.en.xlf', 'en', 'validators');
$translator->addResource('xlf', $vendorValidatorDir.'/Resources/translations/validators.en.xlf', 'en', 'validators');
$translator->addResource('xlf', $vendorFormDir.'/Resources/translations/validators.ja.xlf', 'ja', 'validators');
$translator->addResource('xlf', $vendorValidatorDir.'/Resources/translations/validators.ja.xlf', 'ja', 'validators');

$validator = Validation::createValidatorBuilder()
  ->setTranslator($translator)
  ->setTranslationDomain('validators')
  ->getValidator();

$formFactory = Forms::createFormFactoryBuilder()
  ->addExtensions([new ValidatorExtension($validator)])
  ->getFormFactory();

$form = $formFactory->get()->createBuilder()
  ->add('email', TextType::class, array(
        'constraints' => array(
          new NotBlank(),
          new Length(array('min' => 4)),
          ),
        ))
  ->add('password', PasswordType::class, array(
        'constraints' => array(
          new NotBlank(),
          new Length(array('min' => 6)),
          ),
        ))
  ->getForm();

...
```

It is important that you specify `validators` as the domain, 4th argument of `addResource` method.  
And it should correspond to the validator's domain, which is specified with `setTranslationDomain` method.  
You have to pass translator instance to the validator, too.  

Please see the documentation if you want further explanation for domain.
<http://symfony.com/doc/current/components/translation/introduction.html#using-message-domains>  

That's it!  
You already have form with internationalized validation messages.  

References:  
<http://symfony.com/doc/current/components/form/introduction.html#translation>
<http://stackoverflow.com/questions/16063531/translation-in-validator-component>
