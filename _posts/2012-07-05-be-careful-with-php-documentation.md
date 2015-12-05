---
layout: post
excerpt: Recent PHP versions change the way to use it and the documentation page can fool people working in 5.3 or below.
image: /images/old/PHP__htmlentities___Manual.png
---

[htmlentities\(\)](http://php.net/manual/en/function.htmlentities.php) is a well known PHP function used to convert strings in a HTML-safe representation. Recent PHP versions change the way to use it and the documentation page can fool people working in 5.3 or below.

# A classic escaping

One of the developpers [I am coaching](/what-i-do) told me he had an unexpected behaviour in his [view](http://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller). Eventually, he end up with this code, (written in UTF-8), running on his PHP5.3 environment

{% highlight php startinline=1 %}
$string = "I like chocolate éclairs."; // bit.ly/extvJY

echo "as is  :", $string, "\n";
echo "h()    :", htmlentities($string), "\n";
{% endhighlight %}

He would have expected something like "I like chocolate &eacute;clairs." and got "I like chocolate &Atilde;&copy;clairs.".

You may recognise typical characters - &Atilde;&copy; - involved in conversion of iso->utf8, when the source is already utf8.

When you look at the description of the doc, it says:

![htmlentities function description](/images/old/9bce0d2d91adcef54229575cdaf5a108130266a9.png)

My developper told me that according to the [documentation page](http://php.net/manual/en/function.htmlentities.php), the function was expecting a UTF-8 string by default, which he provided, so everything should be good. Unfortunatly, it is actually more complicated than that.

# PHP changed with the 5.4 release, including its documentation.

![htmlentities function changelog](/images/old/2bd84b931919a12a7f8ff9004985c56128d7f3f2.png)

If you scroll down a few pages, you'll see that the PHP version 5.4 has led quite a lot of changes, including the constants or the default settings.

<div style="float:right;">
![htmlentities function changelog](/images/old/7999e0773b669022911749465dfee90faa0aa942.png)
</div>

Thus, the default encoding is ISO-8859-1 for versions prior to version 5.4.0 of PHP and UTF-8 from version 5.4.0. Unaware of that, the developper - who was working with PHP 5.3  - remained at the function description and did not specify any encoding (it has always been an optional parameter).

Similarly, the file ext/standard/html.h, which defines constants for the encoding is different for [PHP 5.3](https://github.com/php/php-src/blob/PHP-5.3.13/ext/standard/html.h) and [PHP 5.4](https://github.com/php/php-src/blob/PHP-5.4.4/ext/standard/html.h): ENT\_* constants were added in PHP 5.4.

Here is what the developper and I ended with after looking more carrefully the documentation.

{% highlight php startinline=1 %}
$string = "I like chocolate éclairs."; // bit.ly/extvJY

echo "as is     :", $string, "\n";
echo "h() bad :", htmlentities($string), "\n"; // bad in 5.3 if $string is in utf8

// following is good in both 5.3 and 5.4 if $string is in utf8
echo "h() ok  :", htmlentities($string, ENT_COMPAT, 'UTF-8'), "\n"; 
{% endhighlight %}

# Conclusion

* htmlentities() escapes strings in an HTML-safe representation with a different behavior depending on the version of PHP
* 5.4 modifies a lot of things and has an impact in the documentation: reading the description is not sufficient. *Read everything*. Having a local documentation suited for your PHP version is recommanded.

# Bonus notes

* Mind that htmlentities() does not recognise all Unicode-Symbols.
* The [html\_entity\_decode\(\)](http://fr.php.net/manual/en/function.html-entity-decode.php) function is the opposite of htmlentities(). Yep, with underscores in the middle of function name.
