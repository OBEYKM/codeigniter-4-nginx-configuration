# DevJungle: Configuration Codeigniter 4 with Nginx

![Devjungle logo](DevJungle.webp)

##Welcome to **DevJungle**, where we **Code \~ Evolve \~ Thrive**! 🚀

This guide will walk you through setting up an codeigniter framework with nginx on Linux-based OS (Ubuntu). Follow along step-by-step to get your server running smoothly!

> **Note:** Feel free to fork this project and improve it—your contributions are welcome! 😊

---

> Note: if you dont have nginx installed please [click here](https://github.com/OBEYKM/ngnx-ubuntu-server-setup), which is another documentation for installing nginx !


___________________________________________________________________________________________________________________________



## 1️⃣ Install PHP, and MySQL (optional)

```sh

sudo apt update
sudo apt install php-fpm php-mysql unzip curl -y


```

PHP-FPM (FastCGI Process Manager) is an alternative PHP execution method that is optimized for high-performance web applications. It runs as a separate service and processes PHP scripts more efficiently than traditional PHP execution methods.

### Why Use PHP-FPM with Nginx?
+ Nginx does not have a built-in PHP processor.
+ PHP-FPM handles multiple requests faster than running PHP as an Apache module.
+ It reduces memory usage compared to Apache’s mod_php.

### PHP-FPM Works with Nginx?
When a request is made to a PHP file:

+ ginx receives the request (e.g., index.php).
+ Nginx forwards the request to PHP-FPM.
+ PHP-FPM processes the script and returns the result to Nginx.
+ Nginx delivers the response to the user.

### Verify PHP installation:

```sh
php -v
```


#### Error you might find (missing PHP extension)

if you see similiar to this error message in your terminal: 
> PHP Warning:  PHP Startup: Unable to load dynamic library 'curl' (tried: /usr/lib/php/20240924/curl (/usr/lib/php/20240924/curl: cannot open shared object file: No such file or directory), /usr/lib/php/20240924/curl.so (/usr/lib/php/20240924/curl.so: cannot open shared object file: No such file or directory)) in Unknown on line 0
PHP Warning:  PHP Startup: Unable to load dynamic library 'intl' (tried: /usr/lib/php/20240924/intl (/usr/lib/php/20240924/intl: cannot open shared object file: No such file or directory), /usr/lib/php/20240924/intl.so (/usr/lib/php/20240924/intl.so: cannot open shared object file: No such file or directory)) in Unknown on line 0
PHP Warning:  PHP Startup: Unable to load dynamic library 'mysqli' (tried: /usr/lib/php/20240924/mysqli (/usr/lib/php/20240924/mysqli: cannot open shared object file: No such file or directory), /usr/lib/php/20240924/mysqli.so (/usr/lib/php/20240924/mysqli.so: undefined symbol: mysqlnd_global_stats)) in Unknown on line 0
PHP Warning:  PHP Startup: Unable to load dynamic library 'openssl' (tried: /usr/lib/php/20240924/openssl (/usr/lib/php/20240924/openssl: cannot open shared object file: No such file or directory), /usr/lib/php/20240924/openssl.so (/usr/lib/php/20240924/openssl.so: cannot open shared object file: No such file or directory)) in Unknown on line 0
PHP Warning:  PHP Startup: Unable to load dynamic library 'sqlite3' (tried: /usr/lib/php/20240924/sqlite3 (/usr/lib/php/20240924/sqlite3: cannot open shared object file: No such file or directory), /usr/lib/php/20240924/sqlite3.so (/usr/lib/php/20240924/sqlite3.so: cannot open shared object file: No such file or directory)) in Unknown on line 0
PHP Warning:  PHP Startup: Unable to load dynamic library 'zip' (tried: /usr/lib/php/20240924/zip (/usr/lib/php/20240924/zip: cannot open shared object file: No such file or directory), /usr/lib/php/20240924/zip.so (/usr/lib/php/20240924/zip.so: cannot open shared object file: No such file or directory)) in Unknown on line 0

***These errors indicate that PHP is trying to load extensions eg: (curl, intl, mysqli, openssl, sqlite3, and zip), but they are either not installed or misconfigured. Here's how to fix them:***

##### Install the Missing PHP Extensions
these extension ( curl and intl ) are requiered from codeigniter 4, so that it can work properly

```sh
sudo apt update
sudo apt install php8.4-curl php8.4-intl -y


```

>note: subtitute version number that you installed in your system , above code shows for version 8.4

##### Verify Installed Extensions

```sh
php -m | grep -E 'curl|intl'

```

##### Restart PHP-FPM and Nginx

```sh
sudo systemctl restart php<version>-fpm

```
>note subsitute <version> with your php version instaled

#### Error you might find (Duplicate Extension Entries)

>php -m | grep -E 'curl|intl'

PHP Warning:  Module "curl" is already loaded in Unknown on line 0
PHP Warning:  Module "intl" is already loaded in Unknown on line 0
PHP Warning:  Module "zip" is already loaded in Unknown on line 0
PHP 8.4.4 (cli) (built: Feb 15 2025 08:51:15) (NTS)
Copyright (c) The PHP Group
Zend Engine v4.4.4, Copyright (c) Zend Technologies
    with Zend OPcache v8.4.4, Copyright (c), by Zend Technologies
PHP Warning:  Module "curl" is already loaded in Unknown on line 0
PHP Warning:  Module "intl" is already loaded in Unknown on line 0
PHP Warning:  Module "zip" is already loaded in Unknown on line 0
curl
intl


##### Find all php.ini files being used Run:
```sh
php --ini

```
Loaded Configuration File: /etc/php/8.4/cli/php.ini
Additional .ini files parsed: /etc/php/8.4/cli/conf.d/*.ini

##### Check php.ini for duplicate entries Open the PHP configuration file:

```sh
sudo nano /etc/php/<version>/cli/php.ini

```
Search for these lines (press CTRL+W and type curl, intl, zip to find them):

```sh
extension=curl
extension=intl
extension=zip

```
***If these lines appear multiple times, comment out the duplicates by adding ; at the beginning.***


##### Check conf.d directory for duplicate entries Run:

```sh
ls /etc/php/<version>/cli/conf.d/ | grep -E 'curl|intl'

```

If you see multiple .ini files trying to load the same extension (e.g., 20-curl.ini and 30-curl.ini), you may need to remove one:

```sh
sudo rm /etc/php/8.4/cli/conf.d/30-curl.ini
sudo rm /etc/php/8.4/cli/conf.d/30-intl.ini
sudo rm /etc/php/8.4/cli/conf.d/30-zip.ini

```


##### Restart PHP and Nginx

```sh
sudo systemctl restart php8.4-fpm
sudo systemctl restart nginx

```

#### Issue: Missing PHP ext-dom Extension
Your CodeIgniter 4 installation failed because PHPUnit requires ext-dom, but it's missing from your system.


##### Install php-xml (Provides ext-dom)

```sh
sudo apt update
sudo apt install php<version>-xml -y

```
###### Verify Installation

```sh
php -m | grep dom

```

#### Missing PHP mbstring Extension
Your CodeIgniter 4 installation failed because PHPUnit requires ext-mbstring, but it's missing from your system.

```sh
sudo apt update
sudo apt install php8.4-mbstring -y

```
> Restart PHP-FPM and Nginx   
sudo systemctl restart php<version>-fpm
sudo systemctl restart nginx
