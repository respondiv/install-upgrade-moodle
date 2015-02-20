
#### Install phpmyadmin

**update and install phpmyadmin**

```
sudo apt-get update
sudo apt-get install phpmyadmin
```

**Change port of phpmyadmin** 

```
# add code here

```

**Force SSL for phpmyadmin - if using ssl**

```
# edit /etc/phpmyadmin/config.inc.PhPMyAdmin
sudo nano /etc/phpmyadmin/config.inc.PhPMyAdmin

# add this lines on server(s) configuration
$cfg['ForceSSL'] = 'true';
```

Press `ctrl+x` and type `y` to save and exit

**Enable mcrypt**

`sudo php5enmod mcrypt`

If this doesn't work, then install mcrypt and enable it.

```
# Install mcrypt
sudo apt-get update
sudo apt-get install php5-mcrypt

# Create symlink to mods-avaliable
sudo ln -s /etc/php5/conf.d/mcrypt.ini /etc/php5/mods-available/mcrypt.ini

# enable it
sudo php5enmod mcrypt

# Restart Php-fpm
sudo service php5-fpm restart
```

**Test and Restart nginx**

`sudo nginx -t && sudo service nginx restart`

#### Check for PhPmyAdmin Error, if there is any. To check

**Go to `https://1.2.3.4:33344/db/pma` using Web Browser**
- Replace 1.2.3.4 with your VPS ip
- Replace :33344 with the new port that you have setup above
- Login to PhpMyAdmin using root credentials
  - Username: root
  - Password: MySQL root Password that you have setup above
- See if there is any Error, Normally there should be two
  - The configuration file now needs a secret passphrase (blowfish_secret)
  - The phpMyAdmin configuration storage is not completely configured

#### To fix "The phpMyAdmin configuration storage is not completely configured"

Copy all the SQL queries from here [create_tables.sql](https://github.com/respondiv/Set-up-Linux-Server/blob/master/create_tables.sql)

or, you can login Using SSH and copy all the queries from `/var/www/22222/htdocs/db/pma/examples/create_tables.sql`. To do so, run following command

`cat /var/www/22222/htdocs/db/pma/examples/create_tables.sql`

Copy the displayed SQL queries, then perfom following steps
- Go to phpmyadmin url `https://1.2.3.4:33344/db/pma`
  - replace ip and port to match yours
- Select "SQL" menu without choosing any database
- Paste the queries copied from create_tables.sql file and click "GO"
  - You should see the message "MySQL returned an empty result set (i.e. zero rows) in Green bar"

Now go back to SSH, we need to edit `config.inc.php` file

`sudo nano /var/www/22222/htdocs/db/pma/config.inc.php`

If it returns an empty file, then you need to copy content of `config.sample.inc.php` to `config.inc.php` 

`cp /var/www/22222/htdocs/db/pma/config.sample.inc.php /var/www/22222/htdocs/db/pma/config.inc.php`

Now edit `config.inc.php` by `sudo nano /var/www/22222/htdocs/db/pma/config.inc.php`

Make following Changes
- Add blowfish_secret, which is a random word(s)
  - this will fix the blowfish_secret error
- uncomment all storage database and tables config.

**Before Change**

```
......
......

/*
 * This is needed for cookie based authentication to encrypt password in
 * cookie
 */
$cfg['blowfish_secret'] = ''; /* YOU MUST FILL IN THIS FOR COOKIE AUTH! */

/*
 * Servers configuration
 */
$i = 0;

/*
 * First server
 */
$i++;
/* Authentication type */
$cfg['Servers'][$i]['auth_type'] = 'cookie';
/* Server parameters */
$cfg['Servers'][$i]['host'] = 'localhost';
$cfg['Servers'][$i]['connect_type'] = 'tcp';
$cfg['Servers'][$i]['compress'] = false;
$cfg['Servers'][$i]['AllowNoPassword'] = false;

/*
 * phpMyAdmin configuration storage settings.
 */

/* User used to manipulate with storage */
// $cfg['Servers'][$i]['controlhost'] = '';
// $cfg['Servers'][$i]['controlport'] = '';
// $cfg['Servers'][$i]['controluser'] = 'pma';
// $cfg['Servers'][$i]['controlpass'] = 'pmapass';

/* Storage database and tables */
// $cfg['Servers'][$i]['pmadb'] = 'phpmyadmin';
// $cfg['Servers'][$i]['bookmarktable'] = 'pma__bookmark';
// $cfg['Servers'][$i]['relation'] = 'pma__relation';
// $cfg['Servers'][$i]['table_info'] = 'pma__table_info';
// $cfg['Servers'][$i]['table_coords'] = 'pma__table_coords';
// $cfg['Servers'][$i]['pdf_pages'] = 'pma__pdf_pages';
// $cfg['Servers'][$i]['column_info'] = 'pma__column_info';
// $cfg['Servers'][$i]['history'] = 'pma__history';
// $cfg['Servers'][$i]['table_uiprefs'] = 'pma__table_uiprefs';
// $cfg['Servers'][$i]['tracking'] = 'pma__tracking';
// $cfg['Servers'][$i]['userconfig'] = 'pma__userconfig';
// $cfg['Servers'][$i]['recent'] = 'pma__recent';
// $cfg['Servers'][$i]['favorite'] = 'pma__favorite';
// $cfg['Servers'][$i]['users'] = 'pma__users';
// $cfg['Servers'][$i]['usergroups'] = 'pma__usergroups';
// $cfg['Servers'][$i]['navigationhiding'] = 'pma__navigationhiding';
// $cfg['Servers'][$i]['savedsearches'] = 'pma__savedsearches';
// $cfg['Servers'][$i]['central_columns'] = 'pma__central_columns';
/* Contrib / Swekey authentication */
// $cfg['Servers'][$i]['auth_swekey_config'] = '/etc/swekey-pma.conf';

......
......

```

**After Changes**

```
......
......

/*
 * This is needed for cookie based authentication to encrypt password in
 * cookie
 */
$cfg['blowfish_secret'] = 'MySuperSecretPasswordText'; /* YOU MUST FILL IN THIS FOR COOKIE AUTH! */

/*
 * Servers configuration
 */
$i = 0;

/*
 * First server
 */
$i++;
/* Authentication type */
$cfg['Servers'][$i]['auth_type'] = 'cookie';
/* Server parameters */
$cfg['Servers'][$i]['host'] = 'localhost';
$cfg['Servers'][$i]['connect_type'] = 'tcp';
$cfg['Servers'][$i]['compress'] = false;
$cfg['Servers'][$i]['AllowNoPassword'] = false;

/*
 * phpMyAdmin configuration storage settings.
 */

/* User used to manipulate with storage */
// $cfg['Servers'][$i]['controlhost'] = '';
// $cfg['Servers'][$i]['controlport'] = '';
// $cfg['Servers'][$i]['controluser'] = 'pma';
// $cfg['Servers'][$i]['controlpass'] = 'pmapass';

/* Storage database and tables */

/* Uncomment These Lines */

$cfg['Servers'][$i]['pmadb'] = 'phpmyadmin';
$cfg['Servers'][$i]['bookmarktable'] = 'pma__bookmark';
$cfg['Servers'][$i]['relation'] = 'pma__relation';
$cfg['Servers'][$i]['table_info'] = 'pma__table_info';
$cfg['Servers'][$i]['table_coords'] = 'pma__table_coords';
$cfg['Servers'][$i]['pdf_pages'] = 'pma__pdf_pages';
$cfg['Servers'][$i]['column_info'] = 'pma__column_info';
$cfg['Servers'][$i]['history'] = 'pma__history';
$cfg['Servers'][$i]['table_uiprefs'] = 'pma__table_uiprefs';
$cfg['Servers'][$i]['tracking'] = 'pma__tracking';
$cfg['Servers'][$i]['userconfig'] = 'pma__userconfig';
$cfg['Servers'][$i]['recent'] = 'pma__recent';
$cfg['Servers'][$i]['favorite'] = 'pma__favorite';
$cfg['Servers'][$i]['users'] = 'pma__users';
$cfg['Servers'][$i]['usergroups'] = 'pma__usergroups';
$cfg['Servers'][$i]['navigationhiding'] = 'pma__navigationhiding';
$cfg['Servers'][$i]['savedsearches'] = 'pma__savedsearches';
$cfg['Servers'][$i]['central_columns'] = 'pma__central_columns';

/* Stop Uncommenting here */

/* Contrib / Swekey authentication */
// $cfg['Servers'][$i]['auth_swekey_config'] = '/etc/swekey-pma.conf';

......
......

```



Press `ctrl+x` and type `y` to save and exit

Go to the Web Browser, Logout from current session of phpMyAdmin and login back. The errors should be fixed




