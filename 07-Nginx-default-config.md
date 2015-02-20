#### Default Nginx Config for Optimized Website

**check # of cpu, the # of worker_processes value (below) depends upon # of CPU**

`sudo grep processor /proc/cpuinfo | wc -l`

**Edit Nginx default config**

`sudo nano /etc/nginx/nginx.conf`

Make necessary changes so it looks like following

```
user www-data;
worker_processes auto;
worker_rlimit_nofile 100000;
pid /run/nginx.pid;

events {
  # add epoll
  use epoll;
  worker_connections 4096;
  multi_accept on;
}

http {
  
  server_tokens off;
  reset_timedout_connection on;
  add_header rd-Fastcgi-Cache $upstream_cache_status;
  
  # Limit Request
  limit_req_zone $binary_remote_addr zone=one:10m rate=1r/s;
  
  # must add
  limit_req_zone $binary_remote_addr zone=php:10m rate=2r/s;
  limit_req_status 429;
  limit_conn_status 429;
  
  # Proxy Settings
  # set_real_ip_from proxy-server-ip;
  # real_ip_header X-Forwarded-For;
  
  # FastCGI cache settings
  fastcgi_read_timeout 300;
  fastcgi_cache_path /var/run/nginx-cache levels=1:2 keys_zone=WORDPRESS:50m inactive=60m;
  fastcgi_cache_key "$scheme$request_method$host$request_uri";
  fastcgi_cache_use_stale error timeout invalid_header updating http_500 http_503;
  fastcgi_cache_valid 200 301 302 404 1h;
  fastcgi_buffers 16 16k;
  fastcgi_buffer_size 32k;
  fastcgi_param SERVER_NAME $http_host;
  fastcgi_ignore_headers Cache-Control Expires Set-Cookie;

  
  # Change this to the desired max upload limit
  client_max_body_size 100m;
  
  # SSL Settings
  ssl_session_cache shared:SSL:20m;
  ssl_session_timeout 10m;
  ssl_prefer_server_ciphers on;
  ssl_ciphers "ECDHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-SHA384:ECDHE-RSA-AES128-SHA256:ECDHE-RSA-AES256-SHA:ECDHE-RSA-AES128-SHA:DHE-RSA-AES256-SHA256:DHE-RSA-AES128-SHA256:DHE-RSA-AES256-SHA:DHE-RSA-AES128-SHA:ECDHE-RSA-DES-CBC3-SHA:EDH-RSA-DES-CBC3-SHA:AES256-GCM-SHA384:AES128-GCM-SHA256:AES256-SHA256:AES128-SHA256:AES256-SHA:AES128-SHA:DES-CBC3-SHA:HIGH:!aNULL:!eNULL:!EXPORT:!DES:!MD5:!PSK:!RC4";
  
  
  ##
  # Basic Settings
  ##
  sendfile on;
  tcp_nopush on;
  tcp_nodelay on;
  keepalive_timeout 30;
  types_hash_max_size 2048;
  # server_tokens off;

  server_names_hash_bucket_size 64;
  server_name_in_redirect off;
  
  include /etc/nginx/mime.types;
  default_type application/octet-stream;
  
  ##
  # Logging Settings
  ##
  access_log /var/log/nginx/access.log;
  error_log /var/log/nginx/error.log;
  log_format respondiv_cache '$remote_addr $upstream_response_time $upstream_cache_status [$time_local] '
  '$http_host "$request" $status $body_bytes_sent '
  '"$http_referer" "$http_user_agent"';
  
  ##
  # Gzip Settings
  ##
  gzip on;
  gzip_disable "msie6";
  gzip_vary on;
  gzip_proxied any;
  gzip_comp_level 6;
  gzip_buffers 16 8k;
  gzip_http_version 1.1;
  gzip_types text/plain text/css application/json application/x-javascript text/xml application/xml application/xml+rss text/javascript;
  
  ##
  # Virtual Host Configs
  ##
  
  include /etc/nginx/conf.d/*.conf;
  include /etc/nginx/sites-enabled/*;

}
```

Press `ctrl+x` and type `y` to save and exit

**Test and Restart nginx**

`sudo nginx -t && sudo /etc/init.d/nginx restart`

#### Create few extra config file to use later on multiple website

**Create upstream config file**

`sudo nano /etc/nginx/conf.d/upstream.conf`

Paste the following

```
# Common upstream settings
upstream php {
# server unix:/run/php5-fpm.sock;
server 127.0.0.1:9000;
}
upstream debug {
# Debug Pool
server 127.0.0.1:9001;
}
```

Press `ctrl+x` and type `y` to save and exit

**Create Location config file**

`sudo nano /etc/nginx/common/locations.conf`

Paste the following 

```
# NGINX CONFIGURATION FOR COMMON LOCATION
# Basic locations files
location = /favicon.ico {
  access_log off;
  log_not_found off;
  expires max;
}
location = /robots.txt {
  # Some WordPress plugin gererate robots.txt file
  # Refer #340 issue
  try_files $uri $uri/ /index.php?$args;
  access_log off;
  log_not_found off;
}
# Cache static files
location ~* \.(ogg|ogv|svg|svgz|eot|otf|woff|mp4|ttf|css|rss|atom|js|jpg|jpeg|gif|png|ico|zip|tgz|gz|rar|bz2|doc|xls|exe|ppt|tar|mid|midi|wav|bmp|rtf|swf)$ {
  add_header "Access-Control-Allow-Origin" "*";
  access_log off;
  log_not_found off;
  expires max;
}
# Security settings for better privacy
# Deny hidden files
location ~ /\. {
  deny all;
  access_log off;
  log_not_found off;
}
# Deny backup extensions & log files
location ~* ^.+\.(bak|log|old|orig|original|php#|php~|php_bak|save|swo|swp|sql)$ {
  deny all;
  access_log off;
  log_not_found off;
}
# Return 403 forbidden for readme.(txt|html) or license.(txt|html)
if ($request_uri ~* "^.+(readme|license)\.(txt|html)$") {
  return 403;
}
# Status pages
location /nginx_status {
  stub_status on;
  access_log off;
  include common/acl.conf;
}
location ~ ^/(status|ping) {
  include fastcgi_params;
  fastcgi_pass php;
  include common/acl.conf;
}
```

Press `ctrl+x` and type `y` to save and exit

**Create Wp-Common config file for common WordPress config**

`sudo nano /etc/nginx/common/wpcommon.conf`

Paste the following 

```
# WordPress COMMON SETTINGS
# Limit access to avoid brute force attack
location = /wp-login.php {
  limit_req zone=one burst=1 nodelay;
  include fastcgi_params;
  fastcgi_pass php;
}
# Disable wp-config.txt
location = /wp-config.txt {
  deny all;
  access_log off;
  log_not_found off;
}
# Disallow php in upload folder
location /wp-content/uploads/ {
  location ~ \.php$ {
    #Prevent Direct Access Of PHP Files From Web Browsers
    deny all;
  }
}
# Yoast sitemap
location ~ ([^/]*)sitemap(.*)\.x(m|s)l$ {
  rewrite ^/sitemap\.xml$ /sitemap_index.xml permanent;
  rewrite ^/([a-z]+)?-?sitemap\.xsl$ /index.php?xsl=$1 last;
  # Rules for yoast sitemap with wp|wpsubdir|wpsubdomain
  rewrite ^.*/sitemap_index\.xml$ /index.php?sitemap=1 last;
  rewrite ^.*/([^/]+?)-sitemap([0-9]+)?\.xml$ /index.php?sitemap=$1&sitemap_n=$2 last;
  # Following lines are options. Needed for WordPress seo addons
  rewrite ^/news_sitemap\.xml$ /index.php?sitemap=wpseo_news last;
  rewrite ^/locations\.kml$ /index.php?sitemap=wpseo_local_kml last;
  rewrite ^/geo_sitemap\.xml$ /index.php?sitemap=wpseo_local last;
  rewrite ^/video-sitemap\.xsl$ /index.php?xsl=video last;
  access_log off;
}
```

Press `ctrl+x` and type `y` to save and exit

**Create Fast CGI config file to work with WordPress**

`sudo nano /etc/nginx/common/wpfc.conf`

Paste the following 

```
# WPFC NGINX CONFIGURATION
set $skip_cache 0;
# POST requests and URL with a query string should always go to php
if ($request_method = POST) {
  set $skip_cache 1;
}
if ($query_string != "") {
  set $skip_cache 1;
}
# Don't cache URL containing the following segments
if ($request_uri ~* "(/wp-admin/|/xmlrpc.php|wp-.*.php|index.php|/feed/|sitemap(_index)?.xml|[a-z0-9_-]+-sitemap([0-9]+)?.xml)") {
  set $skip_cache 1;
}
# Don't use the cache for logged in users or recent commenter
if ($http_cookie ~* "comment_author|wordpress_[a-f0-9]+|wp-postpass|wordpress_no_cache|wordpress_logged_in") {
  set $skip_cache 1;
}
# Use cached or actual file if they exists, Otherwise pass request to WordPress
location / {
  try_files $uri $uri/ /index.php?$args;
}
location ~ ^/wp-content/cache/minify/(.+\.(css|js))$ {
  try_files $uri /wp-content/plugins/w3-total-cache/pub/minify.php?file=$1;
}
location ~ \.php$ {
  try_files $uri =404;
  include fastcgi_params;
  fastcgi_pass php;
  fastcgi_cache_bypass $skip_cache;
  fastcgi_no_cache $skip_cache;
  fastcgi_cache WORDPRESS;
}
location ~ /purge(/.*) {
  fastcgi_cache_purge WORDPRESS "$scheme$request_method$host$1";
}
```

Press `ctrl+x` and type `y` to save and exit

**Create W3TC config file for WordPress site**

`sudo nano /etc/nginx/common/w3tc.conf`

Paste the following 

```
# W3TC NGINX CONFIGURATION
# DO NOT MODIFY, ALL CHNAGES LOST AFTER UPDATE EasyEngine (ee)
set $cache_uri $request_uri;
# POST requests and URL with a query string should always go to php
if ($request_method = POST) {
  set $cache_uri 'null cache';
}
if ($query_string != "") {
  set $cache_uri 'null cache';
}
# Don't cache URL containing the following segments
if ($request_uri ~* "(/wp-admin/|/xmlrpc.php|wp-.*.php|index.php|/feed/|sitemap(_index)?.xml|[a-z0-9_-]+-sitemap([0-9]+)?.xml)") {
  set $cache_uri 'null cache';
}
# Don't use the cache for logged in users or recent commenter
if ($http_cookie ~* "comment_author|wordpress_[a-f0-9]+|wp-postpass|wordpress_logged_in") {
  set $cache_uri 'null cache';
}
# Use cached or actual file if they exists, Otherwise pass request to WordPress
location / {
  try_files /wp-content/cache/page_enhanced/${host}${cache_uri}_index.html $uri $uri/ /index.php?$args;
}
location ~ ^/wp-content/cache/minify/(.+\.(css|js))$ {
  try_files $uri /wp-content/plugins/w3-total-cache/pub/minify.php?file=$1;
}
location ~ \.php$ {
  try_files $uri =404;
  include fastcgi_params;
  fastcgi_pass php;
}
```

Press `ctrl+x` and type `y` to save and exit

**Create common php config file**

`sudo nano /etc/nginx/common/php.conf`

Paste the following 

```
# PHP NGINX CONFIGURATION
location / {
  try_files $uri $uri/ /index.php?$args;
}
location ~ \.php$ {
  try_files $uri =404;
  include fastcgi_params;
  fastcgi_pass php;
}
```

Press `ctrl+x` and type `y` to save and exit

**Create HTTP Authetication config file**

`sudo nano /etc/nginx/common/acl.conf`

Paste the following 

```
# HTTP authentication || IP address
satisfy any;
auth_basic "Restricted Area";
# need to create htpasswd file at `/etc/nginx/htpasswd-rd`
auth_basic_user_file htpasswd-rd;
# Allowed IP Address List
allow 127.0.0.1;
deny all;
```

Press `ctrl+x` and type `y` to save and exit

*we need to create `htpasswd-rd` file, to create this file perform following steps*

run this command

`openssl passwd`

it will ask you to renter a password and re-enter to confirm it. then it will display that password in encrypted format e.g `O5az.RSPzd.HE`. Copy this 

Create a file `/etc/nginx/htpasswd-rd` and add following

```
#syntax 
# username:encrypted-password-generated-by-openssl

username:O5az.RSPzd.H
```

Press `ctrl+x` and type `y` to save and exit