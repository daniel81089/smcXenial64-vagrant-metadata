## Planned

- systemctl status systemd-networkd systemd-timedated systemd-timesyncd
#### probably better to use ntp, but either would need time to test thoroughly on a test system.

## V2017.09.30

### install system
#### using ubuntu-16.04.3-server-amd64.iso
#### disabled USB and Audio
### update the system
- sudo apt-get update && sudo apt-get upgrade
### set up (compile) latest virtual box guest additions
- sudo apt-get install linux-headers-$(uname -r) build-essential dkms
- sudo mount /dev/cdrom /media/cdrom
- sudo sh /media/cdrom/VBoxLinuxAdditions.run
- sudo umount /media/cdrom
- sudo apt-get clean
### set up ssh for vagrant
- mkdir /home/vagrant/.ssh
- cd /home/vagrant/.ssh/
- curl -O https://raw.githubusercontent.com/hashicorp/vagrant/master/keys/vagrant.pub
- mv vagrant.pub authorized_keys
- chmod 700 /home/vagrant/.ssh
- chmod 600 /home/vagrant/.ssh/authorized_keys
- chown -R vagrant /home/vagrant/.ssh
- sudo apt-get install openssh-server
- sudo apt-get install ssh
#### AuthorizedKeysFile %h/.ssh/authorized_keys
#### PasswordAuthentication no
- sudo nano /etc/ssh/sshd_config
- sudo systemctl restart ssh
### set up and configure lemp stack
- sudo apt-get install nginx
- sudo apt-get install mysql-server
- sudo mysql_secure_installation
- sudo apt-get install php-fpm php-mysql
#### cgi.fix_pathinfo = 0
#### post_max_size = 32M
- sudo mkdir /srv/config && sudo mkdir /srv/config/backup
- sudo cp /etc/php/7.0/fpm/php.ini /srv/config/backup/php.ini
- sudo nano /etc/php/7.0/fpm/php.ini
- sudo systemctl restart php7.0-fpm
#### configure nginx default site
#### root /srv/www/default/htdocs/public;
#### index index.html index.php;
#### try_files $uri $uri/ /index.php;
#### include snippets/fastcgi-php.conf;
#### fastcgi_pass unix:/run/php/php7.0-fpm.sock;
#### location ~ /\\.ht {
#### deny all;
- sudo mkdir /srv/config/backup/sites-available
- sudo cp /etc/nginx/sites-available/default /srv/config/backup/sites-available/default
- sudo nano /etc/nginx/sites-available/default
#### configure nginx
#### sendfile off;
- sudo cp /etc/nginx/nginx.conf /srv/config/backup/nginx.conf
- sudo nano /etc/nginx/nginx.conf
- sudo mkdir -p /srv/www/default/htdocs/public
- sudo chown -R www-data:www-data /srv/www
- sudo chmod -R 770 /srv/www
- sudo nginx -t
- sudo systemctl restart nginx
- sudo usermod -a -G vagrant www-data
- sudo usermod -a -G www-dat vagrant
- sudo shutdown -r now
#### <?php phpinfo();
- nano /srv/www/default/htdocs/public/index.php
### set up ufw
- sudo ufw app list
- sudo ufw allow OpenSSH
- sudo ufw allow "Nginx HTTP"
- sudo ufw enable
- sudo ufw status
### set up composer and laravel-friendly environment
- sudo apt-get install php7.0-mbstring php7.0-xml composer unzip
- mysql -u root -p
#### CREATE DATABASE defaultDB DEFAULT CHARACTER SET utf8 COLLATE utf8_unicode_ci;
#### GRANT ALL ON defaultDB.* TO defaultUser@localhost IDENTIFIED BY "vagrant";
#### FLUSH PRIVILEGES;
#### EXIT;
### set up node.js (node)
- sudo apt-get install build-essential libssl-dev
- curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.4/install.sh | bash
- source ~/.bashrc
- nvm install 8.5.0
- node -v
### set up git and clone laravel to the nginx directory for "default"
- sudo apt-get install git
- cd /srv/www/default/htdocs
- git clone https://github.com/laravel/laravel.git ./
- composer install
- cp .env.example .env
- php artisan key:generate
- npm install
- npm run dev
### finish and restart the machine
#### vagrant ALL=(ALL) NOPASSWD: ALL
- sudo nano /etc/sudoers.d/vagrant
- sudo cp -R www wwwOrig
- sudo shutdown -r now
