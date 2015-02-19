#### Installing Nginx, PHP and MySQL

**Update Local Package**

`sudo apt-get update`

**Install Nginx, PHP and MySQL**

`sudo apt-get install nginx php5-cli php5-fpm php5-mysql mysql-server mysql-client

**Configure php** 

Edit `/etc/php5/fpm/php.ini`

```
# edit php.ini file
sudo nano /etc/php5/fpm/php.ini


# find this line
cgi.fix_pathinfo=1

# change the 1 to 0
cgi.fix_pathinfo=0

# To change Max file upload to 100MB, find and update following line
# Note post_max_size should be larger than upload_max_filesize

upload_max_filesize = 100M
post_max_size = 120M

```

Press `ctrl+x` and type `y` to save and exit

Edit `/etc/php5/fpm/pool.d/www.conf`

```
# edit /etc/php5/fpm/pool.d/www.conf
sudo nano /etc/php5/fpm/pool.d/www.conf


# find this line
listen = 127.0.0.1:9000

# change it to
listen = /var/run/php5-fpm.sock
```

**Configure MySQL**

Tell MySQL to generate the directory structure it needs to store its databases and information.

`sudo mysql_install_db`

**Locate mysql root password, if forgotten**

`sudo cat ~/.my.cnf`

**Secure mysql installation**

`sudo mysql_secure_installation`

Select the configure that best matches you, normally press `y` 5 times. You can update the MySQL root password here if you want to

**Update mysql root password on th econfig file**

`sudo nano ~/.my.cnf`


Press `ctrl+x` and type `y` to save and exit

**Reload php-fpm**

`sudo service php5-fpm restart`

**Test and Restart nginx**

`sudo nginx -t && sudo /etc/init.d/nginx restart`

**Reboot the system if needed**

`sudo reboot now`