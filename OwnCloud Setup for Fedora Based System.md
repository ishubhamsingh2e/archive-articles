
![[assets/Pasted image 20230221144056.png]]

## Installing Dependence

### Apache (HTTP server)

```sh
sudo dnf install httpd
sudo systemctl enable httpd.service
sudo systemctl start httpd.service
```

add HTTP server to firewall exclusion 

```sh
firewall-cmd --permanent --zone=public --add-service=http
firewall-cmd --permanent --zone=public --add-service=https
firewall-cmd --reload
```

### MariaDB

```sh
sudo dnf install mariadb-server
sudo systemctl enable mariadb.service
sudo systemctl start mariadb.service
```

#### Secure your database installation

pressing `y` for every thing is ok

```sh
/usr/bin/mysql_secure_installation
```

```
 mysql_secure_installation

NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL
      MariaDB SERVERS IN PRODUCTION USE!  PLEASE READ EACH STEP
      CAREFULLY!

In order to log into MariaDB to secure it, we'll need the
current password for the root user.  If you've just installed
MariaDB, and you haven't set the root password yet, the password
will be blank, so you should just press enter here.

Enter current password for root (enter for none): **<ENTER>**
OK, successfully used password, moving on...

Setting the root password ensures that nobody can log into
the MariaDB root user without the proper authorization.

Set root password? [Y/n] **<ENTER>**
New password: **Your_Password_Here**
Re-enter new password: **Your_Password_Here**

Password updated successfully!

Reloading privilege tables...
 ... Success!

By default, a MariaDB installation has an anonymous user,
allowing anyone to log into MariaDB without having to have
a user account created for them.  This is intended only for
testing, and to make the installation go a bit smoother.  You
should remove them before moving into a production environment.

Remove anonymous users? [Y/n] **<ENTER>**
 ... Success!

Normally, root should only be allowed to connect from
'localhost'.  This ensures that someone cannot guess at the
root password from the network.

Disallow root login remotely? [Y/n] **<ENTER>**
 ... Success!

By default, MariaDB comes with a database named 'test' that
anyone can access.  This is also intended only for testing, and
should be removed before moving into a production environment.

Remove test database and access to it? [Y/n] **<ENTER>**

 - Dropping test database...
 ... Success!

 - Removing privileges on test database...
 ... Success!

Reloading the privilege tables will ensure that all changes
made so far will take effect immediately.

Reload privilege tables now? [Y/n] **<ENTER>**
 ... Success!

Cleaning up...

All done!  If you've completed all of the above steps, your
MariaDB installation should now be secure.

Thanks for using MariaDB!
```

### Install PHP

make sure to check the required version for your OwnCloud version, can be checker using this `php -v`

```sh
sudo dnf install php php-common php-mysqlnd php-xml php-json php-gd php-mbstring
```

restart HTTP server

```sh
sudo systemctl restart httpd
```

## Get OwnCloud

latest OwnCloud can be found on OwnCloud web page [here](https://owncloud.com/download-server/), copy the tar download link

```sh
cd /var/www
wget <link>
```

I'm using `v10.8.0`, so my download link is `#https://download.owncloud.org/community/owncloud-10.8.0.tar.bz2`

extract files and set the required permission

```sh
tar xjf owncloud-10.8.0.tar.bz2
```

```sh
chown -R apache.apache owncloud
chmod -R 755 owncloud
```

remove the `.tar` file

```sh
rm -f owncloud-10.8.0.tar.bz2
```

## Setup For Database

make sure you are login as root

```sh
mysql -u root -p
```

enter your root password, and you should be in the database command line interface

```sql
CREATE DATABASE owncloud;
GRANT ALL ON owncloud.* to 'root'@'localhost' IDENTIFIED BY '
<password>';
FLUSH PRIVILEGES;
quit
```

now you should be able to open up the webpage at `localhost/owncloud`

to access own cloud from the web, you have to add the domain name or the server public IP in the configuration PHP

path for the configuration file is `/var/www/owncloud/config/config.php`

under trusted domain add your domain/IP

```php
  'trusted_domains' =>
  array (
    0 => 'localhost',
    1 => 'www.example.com',
    2 => '<your public ip>'
  ),
```

if the configuration is correct, your OwnCloud should be online at `<ip/domin>/owncloud`
