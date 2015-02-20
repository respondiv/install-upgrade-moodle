We have created a web rooot directory `/var/www/example.com/htdocs` to host website and have setup-up proper permission.

Now upload the website files, including the database dump (database backup file) if you have one, to the website root `/var/www/example.com/htdocs` via SFTP using the `new-user` credentials  we created.

To create web root directory, re-set existing user's password, or to create a new-user and setup proper permission check [05-Add-Edit-Users.md]()

#### Create a database, database user and import our database

You can create database, database user, and import / export database by [accessing phpmyadmin](). 

If you want to use the command shell, use following commands

```
mysql -u root -p -e "create database new_db; GRANT ALL PRIVILEGES ON new_db.* TO new_db_user@localhost IDENTIFIED BY 'new_db_user_pass'"
```

Here

- root -> root user name
- -p -> will prompt to enter root user's password
- new_db -> new database name
- new_db_user -> username for the new_db database
- new_db_user_pass -> password for the new_db_user

**After the database has been created, use following command to import databse from the backup**

```
# syntax
mysql db_name < /path/to/backup/file/backup_db.sql.gz

e.g
mysql db_name < /var/www/example.com/htdocs/backup_db.sql.gz
```

**Do not forget to update the database name, database user name and password on the config file of your CMS. For WordPress use following command**

```
# open the config file
sudo nano /var/www/example.com/htdocs/wp-config.php

# update the database name, database user name and database user's password
```

Press `ctrl+x` and type `y` to save and exit 

**Change the ownership of the website files/folder back to nginx**

This is specially needed for CMS platform like WordPress

`sudo chown www-data:www-data -R /var/www/`