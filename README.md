# Ubuntu server setup
A set of cheatsheets intended to get you up and running with a fresh Ubuntu server to a production ready server.
We wil go over the initial server setup, install nginx, PHP, PostgreSQL, PostGIS, MySQL, Redis, Nodejs etc. and configure our server to able to deploy a Laravel and Nodejs applications.

This guide is tested on Ubuntu server v18

## Table of contents
- [Autentication and Users](#authentication-and-users)
- [Configure firewall rules](#configure-firewall-rules)
- [Install Nginx](#install-nginx)
- [Setup Nginx server blocks](#setup-nginx-server-blocks)
- [SSL Certificates with Letsencrypt](#ssl-certificates-with-letsencrypt)
- [Install Composer](#install-composer)
- [Install PHP](#install-php)
- [Install and Configure PostgreSQL](#install-and-configure-postgresql)
- [Allow remote access to PostgreSQL](#allow-remote-access-to-postgresql)
- [Install and enable PostGIS](#install-and-enable-postgis)
- [Install MySQL](#install-mysql)
- [Allow remote access to MySQL](#allow-remote-access-to-mysql)
- [Install Redis](#install-redis)
- [Install Nodejs](#install-nodejs)
- [Useful links](#useful-links)
***

## Authentication and Users
This section shows how to connect to a server for the first time.
Before logging onto the server, generate **ssh** keys on your local machine if yo haven't already.

- Generate ssh keys with: `$ ssh-keygen -t rsa -b 4096 -C "your_email@example.com"`
- Login with default user (Digital ocean): `$ ssh root@server_ip`
- Add new user: `$ sudo adduser ultrasamad`. Replace *ultrasamad* with name of user
- Create a copy **.ssh** directory for the new user: `$ rsync --archive --chown=root:root ~/.ssh /home/ultrasamad`
- Grant administrative privileges to user: `$ usermod -aG sudo ultrasamad`
- Disable root login by setting `PermitRootLogin no` directive in `/etc/ssh/sshd_config` and restart SSH process with `sudo systemctl restart sshd`
***
## Configure Firewall rules
- Enable firewall: `sudo ufw enable`
- Allow ssh connection: `$ sudo ufw allow openSSH`
- Check status: `$ sudo ufw status`
- View list of registered applications: `$ sudo ufw app list`
***
## Install Nginx
- Update packages repositories: `$ sudo apt update`
- Install Nginx: `$ sudo apt install nginx`
- Allow Firewall rules for **http** & **https**: `$ sudo ufw allow Nginx Full`
- Verify rule was added: `$ sudo ufw status`
- Restart Nginx: `$ sudo service nginx restart`

## Setup Nginx server blocks
- Create new directory for your site: `$ sudo mkdir -p /var/www/example.com`
- Give directory ownership to current user: `$ sudo chown -R $USER:$USER /var/ww/example.com`
- Chage permissions of web root: `$ sudo chmod 755 -R /var/www` 
- Create test file: `$ sudo vim /var/www/example.com/index.html`
- Cope over default server block to new site: `$ cp /etc/nginx/sites-available/default /etc/nginx/sites-available/example.com`
- Update `server_name` in new site file: `server_name example.com www.example.com;`
- Enable new site server block: `$ sudo ln -s /etc/nginx/sites-available/example.com /etc/nginx/sites-enabled`
- Uncomment `server_names_hash_bucket_size 64` in */etc/nginx/nginx.conf* file
- Verify syntax of Nginx configurations: `$ sudo nginx -t`
- Restart Nginx: `$ sudo service nginx restart`
- Vist your new site at example.com or www.example.com
***

## SSL Certificates with Letsencrypt
- Add certbot repository: `$ sudo add-apt-repository ppa:certbot/certbot`
- Install certbot: `$ sudo apt install python-certbot-ngix`
- Verify the server_name is set to your owned domain in the **/etc/ngix/sites-available/example.com** file
- Make sure firewall rules for http & https is allowed
- Obtain SSL Certificate for your domain: `$ sudo certbot --ngix -d example.com -d www.example.com`. Add any other subdomains you will like to enable ssl for.
- Verify certbot auto renewal: `$ sudo certbot renew --dry-run`
- Adding new subdomains: `$ sudo certbot certbotonly --ngix -d blog.example.com, projects.example.com`
***

## Install Composer
- `$ cd ~ && sudo curl -sS https://getcomposer.org/installer | sudo php`
- Add installation to path: `$ sudo ln -s /usr/local/bin/composer /suer/bin/composer`
- Confirm installation with: `$ composer --version`

## Install PHP
- Add a repository for latest **PHP** version: `$ sudo add-apt-repository ppa:ondrej/php`
- Update repo packages list: `$ sudo apt update`
- Check the version of **PHP** available: `$ sudo apt-cache show php`
- Install **PHP** and all necessary packages: \
```
$ sudo apt install php7.4-fpm php7.4-common php7.4-mysql php7.4-pgsql php7.4-xml php7.4-xmlrpc php7.4-curl php7.4-gd php7.4-imagick php7.4-cli php7.4-dev php7.4-imap php7.4-mbstring php7.4-soap php7.4-zip php7.4-bcmath zip unzip -y
```
- Check **PHP** version: `$ php --version`
- Check status of *php7.4-fpm*: `$ sudo service php7.4-fpm status`
- Configure *php7.4-fpm* gateway. \
Add `fastcgi_pass unix:/run/php/php7.4-fpm.sock;` to the location server block in `/etc/ngix/sites-available/example.com`
- Reload Nginx: `$ sudo service nginx reload`
- Restart *php7.4-fpm*: `$ sudo service php7.4-fpm restart`

## Install and Configure PostgreSQL
- Remove any old installation: `$ sudo apt remove postgresql && sudo apt-get --purge remove postgresql\*`
- Reboot server: `$ sudo reboot`
- Create the repository configuration file:\
```
$ sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
```
- Import the repository signing key:\
 ```
 $ sudo wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
 ```
 - Update repository package list: `$ sudo apt update`
 - Install PostSQL: `$ sudo apt install postgresql-12`
 - Check port of postgres installation: `$ sudo netstat -plunt | grep postgres`
 - Confirm installation: `$ sudo psql --version`
 - Login with default user: `$ sudo -u postgres psql`
 - Exit out of *psql* shell: `$ exit`
 - Create mew role called **demo**: `$ sudo -u postgres createuser --interactive`
 - Create database for the new user: `$ sudo createdb demo`
 - Login with new user: `$ sudo -u demo psql`
 - Change passowrd of the user: `$ sudo -u demo psql -c "ALTER USER demo PASSWORD 'secret123'"`
 
 ## Allow remote access to PostgreSQL
 - Allow Firewall rule: `$ sudo ufw allow 5432/tcp`
 - Update *listen_address* in */etc/postgresql/12/main/postgresql.conf* to `listen_addresses = '*'`
 - Add allowed IP entry to */etc/postgresql/12/main/pg_hba.conf*: `host all all 0.0.0.0/0 md5`
 - Restart PostgreSQL: `$ sudo service postgresql restart`

 ## Install and enable PostGIS
 - Install: `$ sudo apt install postgis postgresql-12-postgis-3`
 - Enable PostGIS on current database: `# CREATE EXTENSION postgis;`
 - Verify PostGIS installation: `# SELECT Postgis_Version();` 
 ***

 ## Install MySQL
 - Download config: `$ sudo wget https://repo.mysql.com/mysql-apt-config_0.8.13-1_all.deb`
 - Extract archive: `$ sudo dpkg -i mysql-apt-config_0.8.13-1_all.deb`
 - Update repository packages list: `$ sudo apt update`
 - Install MySQL server: `$ sudo apt install mysql-server`
 - Enable secure installation: `$ sudo mysql_secure_installation utility`
 - Enable firewal rule for MySQL: `$ sudo ufw enable mysql`
 - Allow MySQL firewall rule: `$ sudo ufw allow mysql`
 - Login as root user: `$ mysql -uroot -p`
 - Check authentication methods used by users: `SELECT user,authentication_string,plugin,host FROM mysql.user;`
 - Change password of user: `# ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'secret123';`\
 OR \
 `# UPDATE mysql.user SET authentication_string = PASSWORD('password') WHERE user = 'root';`
 - Flush privileges: `# FLUSH PRIVILEGES;`
 - Create new user: `# CREATE USER 'ultrasamad'@'localhost' IDENTIFIED BY 'password';`


 ## Allow remote access to MySQL
 - Update *bind_address* in */etc/mysql/mysql.conf.d/mysqld.cnf* `bind-address = 0.0.0.0`
 - Change the host of the connecting user from localhost to **%**: \
 `UPDATE mysql.user SET host='%' WHERE user='username' AND host='localhost';`
  ***

## Install Nodejs
- Enable Node source repository: `curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -`
- Install nodejs: `$ sudo apt install nodejs`
- Verify installation: `$ node --version`
- Install PM2: `$ sudo npm install pm2@latest -g`


## Install Redis
- Update repository packages list: `$ sudo apt update`
- Install redis server: `$ sudo apt install redis-server`
- Update **supervised** in */etc/redis/redis.conf* to `supervised systemd`
- Invoke redis cli to confirm installation: `$ redis-cli`
- Type **ping** and you should get back **pong**

## Useful links
- [Digital ocean community tutorials](https://www.digitalocean.com/community/tags/ubuntu-18-04)
- [How to install PostgreSQL on Ubuntu](https://linuxize.com/post/how-to-install-postgresql-on-ubuntu-18-04/)
- [How to install PostGIS on Ubuntu](https://computingforgeeks.com/how-to-install-postgis-on-ubuntu-debian/)
- [How to install latest version of PostgreSQL on Ubuntu](https://www.howtoforge.com/how-to-install-configure-and-use-latest-postgresql-on-ubuntu/)