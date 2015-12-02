---
layout: post
excerpt: This article shows how to bootstrap a non web based Symfony app. 
---

Most of us use Symfony as a PHP Web Development Framework, but its components can be used in non-web environments,
for instance a standalone command line. This article shows how to bootstrap such applications without unneeded libraries (such as EventDispatcher or HttpKernel)
and how to package them as a PHAR file for easy redistribution. If you're unfamiliar with Symfony components, head over [http://symfony.com/en/components](http://symfony.com/en/components).

Let's say we have to create a small mailer - a dummy one without error handling or proper modeling - to send new year whishes through Gmail SMTP. We'll need :

+ some [CLI](https://en.wikipedia.org/wiki/Command-line_interface) interactions, handled by [symfony/console](http://symfony.com/doc/current/components/console/).
+ A template solution, provided by [twig](http://twig.sensiolabs.org).
+ MIME emailing, done by [swiftmailer](http://swiftmailer.org).
+ A HTML mail template and we'll use the [Ink](http://zurb.com/ink/) responsive one done by the fine folks of [Zurb](http://zurb.com/).
+ A list of recipients in a CSV file with emails addresses and first names. I already prepared one available [here](https://raw2.github.com/0livier/Season-s-Greetings/master/recipients.csv).
+ A Gmail account and its password.

The dependencies will be downloaded using [composer](https://getcomposer.org). If you're not familiar with it, I urge you to look at this handy tool which can even be used to download non PHP dependencies. You'll be able to see the code in the [following github repository](https://github.com/0livier/Season-s-Greetings).

## Project initialization
Let's start by creating the project - we'll assume we're using 5.3+ in a \*nix environment - and the subdirectories where we'll put our code in.
We'll abide by [PSR-0](http://www.php-fig.org/psr/psr-0/), [PSR-1](http://www.php-fig.org/psr/psr-1/) and [PSR-2] (http://www.php-fig.org/psr/psr-2/) standards.

```bash
mkdir ny-mail
cd ny-mail
mkdir -p app src/Greetings/{Command,Parser,Transport}
wget http://getcomposer.org/composer.phar
wget https://raw2.github.com/0livier/Season-s-Greetings/master/recipients.csv
```

Create the following composer.json then install dependencies thanks to php composer.phar install. Additionally, replace the name with some twig so our mail
template will be look like we're going to send personalized messages.

<script src="http://gist-it.appspot.com/github/0livier/Season-s-Greetings/blob/master/composer.json?footer=minimal"></script>

```bash
php composer.phar install
sed -i "s/Hi, Susan Calvin/Happy new year 2014 {{ name }}/" vendor/zurb/ink-basic/basic.html
```

## Show me the code
As we need to load recipients from a CVS file, let's create a really simple parser in src/Greetings/Parser/Csv.php.

<script src="http://gist-it.appspot.com/github/0livier/Season-s-Greetings/blob/master/src/Greetings/Parser/Csv.php?footer=minimal"></script>

The creation and sending of the message is done using Swiftmailer, wrapped in the following class to put in src/Greetings/Transport/Gmail.php.

<script src="http://gist-it.appspot.com/github/0livier/Season-s-Greetings/blob/master/src/Greetings/Transport/Gmail.php?footer=minimal"></script>

These classes will be used in src/Greetings/Command/GreetingsCommand.php. Since we only have one command, telling its name is not necessary
so we'll [remove that need](http://symfony.com/doc/master/components/console/single_command_tool.html) by extending Symfony\\Component\\Console\\Application
in src/Greetings/GreetingsApplication which be itself called from the script app/console as shown below.

<script src="http://gist-it.appspot.com/github/0livier/Season-s-Greetings/blob/master/src/Greetings/Command/GreetingsCommand.php?footer=minimal"></script>

<script src="http://gist-it.appspot.com/github/0livier/Season-s-Greetings/blob/master/src/Greetings/GreetingsApplication.php?footer=minimal"></script>

<script src="http://gist-it.appspot.com/github/0livier/Season-s-Greetings/blob/master/app/console?footer=minimal"></script>

Voila ! We now have a working mailer that can be used as shown below:

```bash
php app/console vendor/zurb/ink-basic/basic.html recipients.csv --username foo@bar.com -p CorrectHorseBatteryStaple
```

## PHAR Packaging

PHAR allows us to package a set of files in a single one, which is very convenient if you want to
distribute an library or an command line application. If your build process is more complex than just aggregating files,
you'll definitely want to look at composer's [PHAR compiler](https://github.com/composer/composer/blob/master/src/Composer/Compiler.php)
to create your own. However, if the build is easy you can rely on [Box](http://box-project.org/) utility, by [Kevin Herrera](https://github.com/kherge),
that will just ask for a simple configuration file.
We'll go for that solution for our CLI application, starting by installing Box at the top directory of our project. You may see a warning
like « Notice: The "phar.readonly" setting needs to be off to create Phars. » but we'll get around that later.

```bash
curl -LSs http://box-project.org/installer.php | php
```

Now let's create a file box.json - still at the top directory - which will tell Box what files to pick or not, that we have an entry point, name of resulting phar.

```javascript
{
    "directories": ["src", "vendor"],
    "blacklist": ["vendor/zurb/ink-basic/basic.html"],
    "stub": true,
    "main": "app/console",
    "output": "greetings.phar"
}
```

Just run

```bash
php box.phar build
```

and you should have a greetings.phar. If you had the warning mentioned earlier, you may need to do

```bash
php -d phar.readonly=0 box.phar build
```

You can now redistribute your command which will be called like that :

```bash
php greetings.phar template.html recipients.csv --username foo@bar.com -p CorrectHorseBatteryStaple
```

We got there ! If you see things to enhance or have questions on what I described here, feel free to comment below or [contact me](http://yet.another.linux-nerd.com/contact).
