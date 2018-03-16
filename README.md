# Install Nginx \(Mainline\) + PHP 7.2 + Mariadb \(Stable\) on Ubuntu 16.04 LTS

#### Step 1: Install Nginx

Setup the  Nginx \(Mainline\) Repository

```
sudo su
echo "deb http://nginx.org/packages/mainline/ubuntu/ xenial nginx" | sudo tee -a /etc/apt/sources.list
echo "deb-src http://nginx.org/packages/mainline/ubuntu/ xenial nginx" | sudo tee -a /etc/apt/sources.list
wget -qO - https://nginx.org/keys/nginx_signing.key | sudo apt-key add -
sudo apt update
```

Installing the Nginx Package

```
sudo apt install nginx
```

Setting up the Firewall

```
sudo ufw allow https
sudo ufw allow http
sudo ufw allow ssh
```

Enable the Nginx service at boot

```
sudo systemctl start nginx
sudo systemctl enable nginx
```

#### Step 2: Install PHP 7.2 + Necessary Modules

Add [Ondřej Surý’s ppa](https://launchpad.net/~ondrej/+archive/ubuntu/php) using the following commands:

```
sudo apt install software-properties-common
sudo add-apt-repository ppa:ondrej/php
sudo apt update
```

Install PHP 7.2 + Additional Necessary Modules:

```
sudo apt install php7.2-fpm php7.2-mysql
sudo apt install php7.2-curl php7.2-gd php7.2-mbstring php7.2-xmlrpc php7.2-xml php7.2-zip
```

Enable PHP systemd service:

```
sudo systemctl start php7.2-fpm
sudo systemctl enable php7.2-fpm
```

Modify `/etc/nginx/conf.d/default.conf`  as follows:

```
server {
    listen 80 default_server;
    listen [::]:80 default_server;
 
    root /usr/share/nginx/html;
    index index.php index.html index.htm index.nginx-debian.html;
 
    server_name localhost;
 
    location / {
        try_files $uri $uri/ =404;
    }
 
    location ~ .php$ {
        fastcgi_split_path_info ^(.+?\.php)(|/.*)$;
        include fastcgi_params;
        fastcgi_param HTTP_PROXY "";
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $fastcgi_path_info;
        fastcgi_param QUERY_STRING $query_string;
        fastcgi_intercept_errors on;
        fastcgi_pass unix:/var/run/php/php7.2-fpm.sock;
    }
 
    location ~ /\.ht {
        deny all;
    }
}
```

Edit `/etc/php/7.2/fpm/php.ini` as follows :

```
cgi.fix_pathinfo=0
```

Create a file `/etc/php/7.2/fpm/pool.d/www.ini` and add the following lines to it:

```
listen.owner = www-data
listen.group = www-data
listen.mode = 0660
```

Edit `/etc/nginx/nginx.conf` as follows:

```
user  www-data;
```

Restart Services:

```
sudo systemctl restart nginx php*
```

#### Step 3 : Install Mariadb

Setup Repository & Import Keys:

```
sudo apt-get install software-properties-common
sudo apt-key adv --recv-keys --keyserver hkp://keyserver.ubuntu.com:80 0xF1656F24C74CD1D8
sudo add-apt-repository 'deb [arch=amd64,i386,ppc64el] http://mirror.netcologne.de/mariadb/repo/10.2/ubuntu xenial main'
```

Installing Mariadb:

```
sudo apt update
sudo apt install mariadb-server
sudo mysql_secure_installation
```

```
root@titan ~ # mysql_secure_installation 

NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MariaDB
      SERVERS IN PRODUCTION USE!  PLEASE READ EACH STEP CAREFULLY!

In order to log into MariaDB to secure it, we'll need the current
password for the root user.  If you've just installed MariaDB, and
you haven't set the root password yet, the password will be blank,
so you should just press enter here.

Enter current password for root (enter for none): 
OK, successfully used password, moving on...

Setting the root password ensures that nobody can log into the MariaDB
root user without the proper authorisation.

You already have a root password set, so you can safely answer 'n'.

Change the root password? [Y/n] n
 ... skipping.

By default, a MariaDB installation has an anonymous user, allowing anyone
to log into MariaDB without having to have a user account created for
them.  This is intended only for testing, and to make the installation
go a bit smoother.  You should remove them before moving into a
production environment.

Remove anonymous users? [Y/n] y
 ... Success!

Normally, root should only be allowed to connect from 'localhost'.  This
ensures that someone cannot guess at the root password from the network.

Disallow root login remotely? [Y/n] y
 ... Success!

By default, MariaDB comes with a database named 'test' that anyone can
access.  This is also intended only for testing, and should be removed
before moving into a production environment.

Remove test database and access to it? [Y/n] y
 - Dropping test database...
 ... Success!
 - Removing privileges on test database...
 ... Success!

Reloading the privilege tables will ensure that all changes made so far
will take effect immediately.

Reload privilege tables now? [Y/n] y
 ... Success!

Cleaning up...

All done!  If you've completed all of the above steps, your MariaDB
installation should now be secure.

Thanks for using MariaDB!

```



