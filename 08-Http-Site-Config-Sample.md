#### Http Site Config - Without SSL

**Create a server block config file, repeat this for multiple domain or subdomain**

`sudo cp /etc/nginx/sites-available/default /etc/nginx/sites-available/example.com`

**Edit Server Block Config File**

`sudo nano /etc/nginx/sites-available/example.com` 

Modify the file as follows

```
server {
  listen 80 default_server;
  listen [::]:80 default_server;
  
  # Make site accessible from http://example.com/
  server_name example.com www.example.com;
  
  access_log /var/log/nginx/example.com.access.log respondiv_cache;
  error_log /var/log/nginx/example.com.error.log;
  
  # website root folder
  root /var/www/example.com/htdocs;
  index index.php index.htm index.html;
  
  # include fast-cgi cache config file, if enabling fast-cgi cache as primary
  #(with w3tc cahce use for database and object cache -> memcache)
  include common/wpfc.conf;

  # include w3tc cache config file, if using w3tc cache as primary caching
  # without Fast-cgi cache
  include common/w3tc.conf;

  # include common wordpress config file if using WordPRess
  include common/wpcommon.conf;

  # include common php config file, if website is custom created using php
  # and mysql
  include common/php.conf;

  # include location config file, this is must
  include common/locations.conf;
  
  # Specify a character set
  charset utf-8;
  
  # SECURITY
  
  # Ignore other host headers
  if ($host !~* ^(example.com|www.example.com)$ ) {
  return 444;
  }
  
  # Only allow GET , POST, and HEAD
  if ($request_method !~ ^(GET|POST|HEAD)$ ) {
  return 444;
  }
  
  #hide phpmyadmin folder or any other folder
  #location ~ ^/phpmyadmin {
  # Allow from static ip (this is a current server ip)
  #allow 104.236.4.61;
  #deny all;
  #return 404;
  #}
  
  # prevent brute force
  #location = /wp-login.php { #use this for single url
  #location ~ ^/wp-admin { #use this for whole folder
  # fastcgi_pass unix:/var/run/php5-fpm.sock;
  # fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
  # include fastcgi_params;
  # limit_req zone=php burst=1 nodelay;
  #}

}

```

**Enable your Server Blocks**
  
`sudo ln -s /etc/nginx/sites-available/example.com /etc/nginx/sites-enabled/`

**Test and Restart nginx**

`sudo nginx -t && sudo /etc/init.d/nginx restart`


