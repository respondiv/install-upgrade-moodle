
#### Update / Upgrade Moodle

**Go to the moodle directory**

`cd /var/www/yoursite.com/htdocs/moodle`

**Update the moodle using following commadn**

```
sudo git pull https://github.com/moodle/moodle.git
git fetch
git merge
git tag
git checkout v3.0.3   
(copy your version number above)
```

**Copy the changes from moodle folder to the web root**

`rsync -ra /var/www/yoursite.com/htdocs/moodle/* /var/www/yoursite.com/htdocs/`


#### Go to your browser and visit your moodle site `http://edu.yoursite.com` then login and follow the upgrade database instruction

