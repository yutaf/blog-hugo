+++
date = "2016-06-24T11:16:11+09:00"
draft = false
title = "Password confirmation form with symfony"
tags = ["php", "twig", "symfony"]

+++

<!--more-->

I was so impressed it goes very easy.  
*RepeatedType Field* helps us to create password & password confirmation fields.  
<http://symfony.com/doc/current/reference/forms/types/repeated.html>

php  

```
...

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
  ->add('password', RepeatedType::class, array(
        'type' => PasswordType::class,
        'required' => true,
        'constraints' => array(
          new NotBlank(),
          new Length(array('min' => 6)),
          ),
        'first_options'  => array('label' => 'label.password'),
        'second_options' => array('label' => 'label.passwordConfirmation'),
        ))
  ->getForm();
```

Then you will have password confirmation fields with their matching validation.  
