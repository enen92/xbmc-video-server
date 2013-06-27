XBMC Video Server
=================

This is a standalone web interface for XBMC which allows users to browse the library and download or stream the movies and TV shows in it.

Requirements
------------

* Web server
* PHP 5.3
* allow_url_fopen = On in php.ini

Installation
------------

The following steps assume that your shell user is able to use the `sudo` command, that you're running a Debian/Ubuntu-based operating system, that you're going to use Apache 2 as web server and that its document root is located in /var/www.

Run the following commands, one by one, in the exact order shown here:

```
sudo su 
apt-get install libapache2-mod-php5 php5-gd php5-cli php5-sqlite git curl sqlite3
a2enmod rewrite headers
service apache2 restart
cd /var/www
git clone git://github.com/Jalle19/xbmc-video-server.git
cd xbmc-video-server
curl -sS https://getcomposer.org/installer | php
php composer.phar install
sqlite3 src/protected/data/xbmc-video-server.db < src/protected/data/schema.sqlite.sql
chmod 777 src/images/image-cache/
chmod 777 src/protected/data
chmod 777 src/protected/data/*.db
chmod 777 src/protected/runtime/
chmod 777 src/assets/
```

Initial setup
-------------

Once the installation is complete, use your web browser to browse to /xbmc-video-server on your web server (if you did the installation steps on your local machine the URL would be http://localhost/xbmc-video-server). You will be presented with a login form. Log in with username "admin" and password "admin" (you'll be able to change this later).

Once logged in, you will be presented with a Settings form, which you must complete before being able to use the application. Here you should specify XBMC's hostname and port as well as the username and password for XBMC's web server.

Proxy Location
--------------

The "Proxy Location" setting is a bit more exotic. Without it, all requests to the XBMC API (including the URLs to your movies and TV shows) are in the form of http://user:pass@hostname:port/API_PATH, which means in order to use the application over the Internet you'd have to forward the right port to the machine running XBMC. It also means you'd be exposing the XBMC API credentials to anyone using your application. What the "Proxy Location" setting does is replace the http://user:pass@hostname:port/ part with the specified location. To make this work you have to configure your web server to provide a reverse proxy on that location. Here's an example for Apache 2 (the configuration lines must be inside your VirtualHost definition):

```
	# Needed to make certain URLs work
	AllowEncodedSlashes On
	
	<Location /xbmc-reverse-proxy>
		ProxyPass http://host:port
		ProxyPassReverse http://host:port
		RequestHeader set Authorization "Basic eGJtYzp4Ym1j"
	</Location>
```

In the example above, "/xbmc-reverse-proxy" should match the "Proxy Location" in the application settings. The "eGJtYzp4Ym1j" part is username:password encoded with Base64 (in the example the credentials are the default xbmc:xbmc).

**You should use a randomly generated long string as location (see Security implications)**

User management
---------------

Once you've configured the application you should be able to browse your library. You can now click the "Users" link in the main menu to configure new users. There are two user roles; User and Administrator. The difference is that a standard user cannot see (and cannot access by other means) the Settings and Users pages.

Security implications
---------------------

This application is insecure by virtue of design. Since there is only one set of credentials for XBMCs API and the only way to authenticate from a media player (such as VLC) is by passing the credentials in the URL, it is impossible to protect XBMC from malicious users. You can avoid exposing the actual API credentials to your users by configuring a proxy location, but on the other hand that exposes the whole API on an authentication-less URL. Thus, if you use a proxy location, you should specify a non-guessable one so that outsiders can't accidentally gain access.

Since the application is not meant for large-scale use (and in order to simplify development), the user passwords are stored in cleartext in the database, although this will most likely change in the future.

Developers
----------

The application comes with pre-compiled CSS files. If you wish to re-compile the files on the fly you need to preload the "less" component and optionally change the path to node and lessc. All of this is done in `src/protected/config/main.php`.