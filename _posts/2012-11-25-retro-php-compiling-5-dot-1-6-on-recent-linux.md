---
layout: post
excerpt: An unforseen need required me to put a six years old PHP 5.1 application back online for
         a few weeks in my customer Intranet. Between rewriting or adapt a significant portion of the
         code to make it work on 5.3+ or getting back a system with a running
         5.1 PHP with its Oracle dependencies, business and team constrainsts
         made us patch the 5.1 code tree for recent libs.
---

An unforseen need required me to put a six years old PHP 5.1 application back online for
a few weeks in my customer Intranet. Between rewriting or adapt a significant portion of the
code to make it work on 5.3+ or getting back a system with a running
5.1 PHP with its Oracle dependencies, business and team constrainsts
usually give little choice:

* Nearly no budget was available,
* Most of the coding team were already booked on others projects,
* Most of them were reluctant to work on old and crapy code,
* The website is internal only,
* There's significant incompatibilities between 5.1 and 5.3 ([5.1 to 5.2][5.1 to 5.2], and [5.2 to 5.3][5.2 to 5.3]).
* The UI is not complex and has nearly no Javascript.

Compilation for a temporary site seemed to be the solution with the lowest duration and risks, however I had to adjust PHP 5.1 a bit to make it work with our recent Centos systems. Here the parts I had to modify :

## The Curl extension

CURLOPT\_FTPASCII was a deprecated constant, alias of CURLOPT\_TRANSFERTEXT and has now been removed from recent [Curl include][Curl include]. The constant must be removed from PHP as sown in the following patch.

```diff
--- /root/php-5.1.6/ext/curl/interface.c.orig 2012-11-15 11:21:54.829741804 +0100
+++ /root/php-5.1.6/ext/curl/interface.c      2012-11-15 11:22:29.026246504 +0100
@@ -266,7 +266,7 @@
        REGISTER_CURL_CONSTANT(CURLOPT_FTPAPPEND);
        REGISTER_CURL_CONSTANT(CURLOPT_NETRC);
        REGISTER_CURL_CONSTANT(CURLOPT_FOLLOWLOCATION);
-       REGISTER_CURL_CONSTANT(CURLOPT_FTPASCII);
+       //      REGISTER_CURL_CONSTANT(CURLOPT_FTPASCII);
        REGISTER_CURL_CONSTANT(CURLOPT_PUT);
 #if CURLOPT_MUTE != 0
        REGISTER_CURL_CONSTANT(CURLOPT_MUTE);
@@ -306,7 +306,7 @@
        REGISTER_CURL_CONSTANT(CURLOPT_FILETIME);
        REGISTER_CURL_CONSTANT(CURLOPT_WRITEFUNCTION);
        REGISTER_CURL_CONSTANT(CURLOPT_READFUNCTION);
-       REGISTER_CURL_CONSTANT(CURLOPT_PASSWDFUNCTION);
+       //      REGISTER_CURL_CONSTANT(CURLOPT_PASSWDFUNCTION);
        REGISTER_CURL_CONSTANT(CURLOPT_HEADERFUNCTION);
        REGISTER_CURL_CONSTANT(CURLOPT_MAXREDIRS);
        REGISTER_CURL_CONSTANT(CURLOPT_MAXCONNECTS);
```

## The Openssl extension
openssl/lhash.h implements a dynamic hash tables systems, but nowaday versions gives the ability to specity types of tables: the macro changed and the PHP extension needs to be adjusted accordingly.

```diff
--- openssl.c.orig      2012-11-15 13:54:21.228464238 +0100
+++ openssl.c   2012-11-15 13:55:21.010589034 +0100
@@ -203,8 +203,8 @@
 static char default_ssl_conf_filename[MAXPATHLEN];
 
 struct php_x509_request {
-       LHASH * global_config;  /* Global SSL config */
-       LHASH * req_config;             /* SSL config for this request */
+       LHASH_OF(CONF_VALUE) * global_config;   /* Global SSL config */
+       LHASH_OF(CONF_VALUE) * req_config;              /* SSL config for this request */
        const EVP_MD * md_alg;
        const EVP_MD * digest;
        char    * section_name,
@@ -364,7 +364,7 @@
                const char * section_label,
                const char * config_filename,
                const char * section,
-               LHASH * config TSRMLS_DC)
+               LHASH_OF(CONF_VALUE) * config TSRMLS_DC)
 {
        X509V3_CTX ctx;
```

## The OCI extension
If you want to add support of recent Oracle client, I strongly suggest you to ignore the 5.1 ext/oci extensions. It's way easier to grab the current [PECL][PECL] one: PHP needs to be compiled without OCI support, and once it's properly installed we use the pecl command to install a working OCI.

```bash
bin/pecl install oci8
```

[5.1 to 5.2]: http://www.php.net/manual/en/migration52.incompatible.php
[5.2 to 5.3]: http://www.php.net/manual/en/migration53.incompatible.php
[Curl include]: https://github.com/bagder/curl/blob/master/include/curl/curl.h
[PECL]: http://pecl.php.net/
