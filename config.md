# Web server setup configuration

### APT
----------

#### nginx, fpm repo

nano /etc/apt/sources.list

    deb http://packages.dotdeb.org stable all
	deb http://packages.dotdeb.org squeeze-php54 all
	deb http://nginx.org/packages/debian/ sqeeze nginx

wget http://www.dotdeb.org/dotdeb.gpg

cat dotdeb.gpg | sudo apt-key add -

rm dotdeb.gpg

apt-get update

### NGINX
----------

nginx [engine x] is an HTTP and reverse proxy server, as well as a mail proxy server

#### nginx install

aptitude install nginx

#### gzip tune

    gzip_min_length 1100;
    gzip_vary on;
    gzip_proxied any;
    gzip_buffers 16 8k;
    gzip_types text/plain text/css application/json application/x-javascript
        text/xml application/xml application/rss+xml text/javascript
        image/svg+xml application/x-font-ttf font/opentype
        application/vnd.ms-fontobject;

#### virtual host

cd /etc/nginx/sites-available

cp default www

cd /etc/nginx/sites-enabled

rm default

ln -s /etc/nginx/sites-available/www .

nano /etc/nginx/sites-available/www

    try_files $uri $uri/ =404;

#### nginx.conf

    server_tokens off;
    
    client_max_body_size  4096k;
    client_header_timeout 10;
    client_body_timeout   10;
    keepalive_timeout     10 10;
    send_timeout          10;

### FPM & ETC
----------

#### install

aptitude install php5-fpm php5-suhosin php5-gd php-apc php5-mcrypt php5-cli php5-curl php5-xdebug memcached php5-memcache

#### apc

nano /etc/php5/conf.d/apc.ini

    apc.shm_size = 128

#### php.ini

nano /etc/php5/fpm/php.ini

    session.save_handler = memcache
	session.save_path = unix:/tmp/memcached.sock
	html_errors = On
	short_open_tag = On

#### www conf

nano /etc/php5/fpm/pool.d/www.conf

    listen = /var/run/php5-fpm.soc
	listen.owner = www-data
	listen.group = www-data

#### memcached

nano /etc/memcached.conf

	# -p 11211
	# -l 127.0.0.1
	
	# Listen on a Unix socket
	-s /tmp/memcached.sock
	-a 666

#### PHP socket

cd /etc/nginx/conf.d/

nano php-sock.conf

	upstream php5-fpm-sock {
    		server unix:/var/run/php5-fpm.soc;
	}

nano /etc/nginx/fastcgi_params

	fastcgi_buffer_size 128k;
	fastcgi_buffers 4 256k;
	fastcgi_busy_buffers_size 256k;

nano /etc/nginx/sites-available/www

	location ~ \.php$ {
		try_files $uri =404;
		include fastcgi_params;
		fastcgi_pass php5-fpm-sock;
		fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
		fastcgi_intercept_errors on;
	}

### MariaDB
----------

MariaDB is a fully MySQL compatible successor of the original MySQL database server. It's more optimised, and has more features then the original. It's being developed by the initial core team.

#### gpg key

apt-key adv --recv-keys --keyserver keyserver.ubuntu.com 0xcbcb082a1bb943db

#### repositories

nano /etc/apt/sources.list

	# MariaDB 10.0 repository list - created 2013-05-02 09:19 UTC
	# http://mariadb.org/mariadb/repositories/
	deb http://ftp.igh.cnrs.fr/pub/mariadb/repo/10.0/debian squeeze main
	deb-src http://ftp.igh.cnrs.fr/pub/mariadb/repo/10.0/debian squeeze main

#### install

aptitude update

aptitude install mariadb-server php5-mysql

#### settings

nano /etc/mysql/my.cnf

    Under "basic settings": skip-networking
    
service mysql restart

#### FPM settings

nano /etc/php5/fpm/php.ini

    Locate "mysql.default_socket", set to:
	    /var/run/mysqld/mysqld.sock

service php5-fpm restart

#### security-ish

mysql -u root -p

    rename user root@localhost to USERNAME@localhost;