#### Converting InnoDB tables to Barracuda

** Tools for Conversion **

```
Go to the web root
cd /var/www/yoursite.com/htdocs

To view tables requiring conversion, use the list option
sudo php admin/cli/mysql_compressed_rows.php --list

To proceed with the conversion, run the command using the fix option
sudo php admin/cli/mysql_compressed_rows.php --fix

```
** That's it, good to go

**if above commands gives error writing to database error follow these steps, if not ignore them

sudo php admin/cli/mysql_compressed_rows.php --showsql

copy the displayed SQl Statement

then, run mysql using root (or admin account)

mysql -u root

