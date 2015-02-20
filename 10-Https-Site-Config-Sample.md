#### Https Site Config  -- After installing SSL Certificate

**Create a server block config file if its not yet created, repeat this for multiple domain or subdomain**

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
  
  # sends all traffic from http to https (only when ssl enabled)
  return 301 https://$server_name$request_uri;
}

server {
  listen 443 ssl default_server spdy;
  listen [::]:443 ssl default_server spdy;  
  
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
  
  # ssl settings begin
  ssl on;
  ssl_certificate /etc/nginx/example.com-ssl/example.com.combined.crt;
  ssl_certificate_key /etc/nginx/example.com-ssl/example.com.key;
  
  #enables all versions of TLS, but not SSLv2 or 3 which are weak and now deprecated.
  ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
  
  # Add-on: OCSP Stapling - recommend by yoast.com
  ssl_stapling on;
  ssl_stapling_verify on;
  resolver 8.8.8.8 8.8.4.4 valid=300s;
  resolver_timeout 10s;
  
  # This forces every request after this one to be over HTTPS
  add_header Strict-Transport-Security "max-age=31536000";
  
  # forward the page to serve via spdy if the server request non-spdy page request
  add_header Alternate-Protocol 443:npn-spdy/3;

 # ssl settings ends
  
  # SECURITY
  
  # Ignore other host headers
  if ($host !~* ^(example.com|www.example.com)$ ) {
  return 444;
  }
  
  # Only allow GET , POST, and HEAD
  if ($request_method !~ ^(GET|POST|HEAD)$ ) {
  return 444;
  }
  
}

```

**Enable your Server Blocks, if its not enabled**
  
`sudo ln -s /etc/nginx/sites-available/example.com /etc/nginx/sites-enabled/`

**Test and Restart nginx**

`sudo nginx -t && sudo /etc/init.d/nginx restart`

