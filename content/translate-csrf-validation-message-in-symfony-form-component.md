+++
date = "2016-06-24T13:33:48+09:00"
draft = false
title = "Translate csrf validation message in symfony form component"
tags = ["i18n", "php", "symfony"]

+++

<!--more-->

I was stuck for a while.  
You need to pass translator instance(also domain as needed) as an argument when you instantiate CsrfExtension.  

php

```
use Symfony\Component\Form\Extension\Csrf\CsrfExtension;
use Symfony\Component\Form\Forms;

...

$csrfExtension = new CsrfExtension($csrfTokenManager, $translator, 'validators');

$formFactory = Forms::createFormFactoryBuilder()
  ->addExtensions([$csrfExtension])
  ->getFormFactory();

...
```

　  

I wrote about internationalization of symfony validation component in [older post]({{< ref "internationalize-symfony-validation-component-outside-of-symfony-framework.md" >}}),  
however it seems each form extensions should be passed the Translator instance individually.  

```
use Symfony\Component\Form\Extension\Csrf\CsrfExtension;
use Symfony\Component\Form\Extension\Validator\ValidatorExtension;
use Symfony\Component\Form\Forms;
use Symfony\Component\Security\Csrf\CsrfTokenManager;
use Symfony\Component\Translation\Translator;
use Symfony\Component\Validator\Validation;

...

$validator = Validation::createValidatorBuilder()
  ->setTranslator($translator)
  ->setTranslationDomain('validators')
  ->getValidator();

// extensions
$validatorExtension = new ValidatorExtension($validator);
$csrfExtension = new CsrfExtension($csrfTokenManager, $translator, 'validators');

$formFactory = Forms::createFormFactoryBuilder()
  ->addExtensions([$csrfExtension, $validatorExtension])
  ->getFormFactory();
```
