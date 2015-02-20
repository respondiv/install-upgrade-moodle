#### Remove `default_server` value from `/etc/nginx/sites-available/default` aka Default Website

**Open the file for editing**

`sudo nano /etc/nginx/sites-available/default`

Modify the file as follows

```
# Before editing
server {
  listen 80 default_server;
  listen [::]:80 default_server;
  ....
}

# After editing
server {
  listen 80;
  listen [::]:80;
  ....
}
```

#### Caching

- When enabling Fast-CGI with WordPRess
    + Ngins helper plugin [https://wordpress.org/plugins/nginx-helper/](https://wordpress.org/plugins/nginx-helper/)
    + Install [W3TC](https://wordpress.org/plugins/w3-total-cache/) plugin
        * Enable Database Cache -> Memcahced
        * Enable Object Cached -> Memcached
        * Disable Page Cache, Browser Cache and Minify
- When enabling W3TC Cache with WordPress
    + Enable Page Cache -> Disk Enhanced
    + Enable Database Cache -> Memcahced
    + Enable Object Cached -> Memcached
    + Disable Browser Cache and Minify

#### Find a file by name

```
# syntax: Find [path] -name "filename.extension"

# e.g this search the file ee.conf in entire harddisk
find / -name "ee.conf"

# e.g this search the file wp-config.php in /var/www/example.com folder
find /var/www/example.com -name "wp-config.php"
```

Details Docs link: [https://help.ubuntu.com/community/find](https://help.ubuntu.com/community/find)

#### Remove a file / folder

```
# syntax to remove a file
sudo rm path/to/file/filename.extension

# e.g
sudo rm /var/www/example.com/htdocs/file.php

# syntax to remove a folder and its content
sudo rm -r path/to/folder

# e.g
sudo rm -r /var/www/example.com/htdocs/folder1
```

#### zip / unzip all files and folder inside a directory

```
# to zip file / folder in current directory
zip -r new-zip-filename.zip path/to/the/folder*

# to zip file / folder in current directory and exclude few director
zip -r new-zip-filename.zip path/to/the/folder* -x "path/to/exclude/folder1" "path/to/exclude/folder2"

# to unzip in current directory
unzip filename.zip

# to unzip in different directory
unzip filename.zip -d path/to/destination_folder
```

#### Backup MySQL Database from command line

```
mysqldump --single-transaction -u mysql_user_name -p -mysql_db_name | gzip > filename.sql.gz
```

#### Repair and Optimize MySQL Database

```
# repair a database
mysqlcheck -r -u mysql_user_name -p -mysql_db_name

# Optimize a database
mysqlcheck -o -u mysql_user_name -p -mysql_db_name
```

#### Change from your sudo user to root user

```
# log into SSH using Sudo user's credential. Then type following command
su
```
