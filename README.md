##Udacity FSND: #Linux Server Configuration

This is the last eigth project in 
"Fullstack Nanodegree Web Developer" on Udacity. 

In this project i configured remote virtual server to serve Catalog project.
  
Site is available on [http://52.24.24.158](http://52.24.24.158)

###Tasks

1. Launch Virtual Machine with Udacity account. 
2. Create a new user named `grader`
3. Give the `grader` the permission to sudo
4. Update all currently installed packages
5. Change the SSH port from 22 to 2200
6. Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)
7. Configure the local timezone to UTC
8. Install and configure Apache to serve a Python mod_wsgi application
9. Install and configure PostgreSQL
10. Install git, clone and setup Catalog App project

### Launch Virtual machine

1. Download private key
2. Save private key to `~/.ssh/udacity_key.rsa`
3. Change mod of key file `chmod 600 ~/.ssh/udacity_key.rsa`
4. Connect to remote server via `ssh root@52.24.24.158 -i ~/.ssh/udacity_key.rsa`
 
Public IP Address: 52.24.24.158

### Create a new user named `grader` and the `grader` the permission to sudo 

1. `sudo adduser grader`
2. `touch /etc/sudoers.d/grader`
3. Paste `grader ALL=(ALL) NOPASSWD:ALL` to `/etc/sudoers.d/grader`
4. Add `Defaults    authenticate` to request user password when usging sudo
5. Add `AllowUsers grader` to `/etc/ssh/sshd_config` file
6. Also change `PasswordAuthentication` to `no`


### Update all currently installed packages

1. `apt-get update`
2. `apt-get upgrade`

### Set ssh login keys

1. Generate public key from private using `ssh-keygen -y -f ~/.ssh/udacity_key.rsa > ~/.ssh/udacity_key.pub` command
2. Deploy public key to `grader` user `.ssh` folder:
```
sudo mkdir /home/grader/.ssh
sudo chmod 700 /home/grader/.ssh
sudo chown grader:grader /home/grader/.ssh
sudo touch /home/grader/.ssh/authorized_keys
sudo chmod 644 /home/grader/.ssh/authorized_keys
sudo chown grader:grader /home/grader/.ssh/authorized_keys
```
3. Paste public key to `~/grader/.ssh/authorized_keys`
4. Reload SSH service using `service ssh restart`
5. Now you can connect to server via `grader` user using `ssh grader@52.24.24.158 -i ~/.ssh/udacity_key.rsa` command

### Change the SSH port from 22 to 2200

1. Backup SSH config `cp /etc/ssh/sshd_config /etc/ssh/sshd_config_backup`
2. Open it with `vi /etc/ssh/sshd_config` and change `22` to `2200`
3. Restrict `root` user login with `PermitRootLogin no`
4. Save and quit `:x`
5. Restart SSH service `service ssh restart`

### Configure the Uncomplicated Firewall

1. Deny all incomig connections with `sudo ufw default deny incoming`
2. Allow all outgoing `sudo ufw default allow outgoing`
3. Allow SSH `ufw allow ssh`
4. Allow HTTP `ufw allow http`
5. Allow NTP `ufw allow ntp`
6. `ufw enable`

### Configure the local timezone to UTC

1. Configure the time zone `dpkg-reconfigure tzdata`
2. Set it to UTC

### Install and configure Apache to serve a Python mod_wsgi app

1. Install apache webserver `apt-get install apache2`
2. Install WSGI `apt-get install libapache2-mod-wsgi`
3. Restart apache `sudo service apache2 restart`

### Install and configure PostgreSQL

1. Install PostgreSQL `sudo apt-get install postgresql`
2. Check if no remote connections are allowed `sudo vim /etc/postgresql/9.3/main/pg_hba.conf`
3. Login as user "postgres" `sudo su - postgres`
4. Get into postgreSQL shell `psql`
5. Create a new database named catalog  and create a new user named catalog in postgreSQL shell
```
postgres=# CREATE DATABASE catalog;
postgres=# CREATE USER catalog;
```
5. Set a password for user catalog
```
postgres=# ALTER ROLE catalog WITH PASSWORD 'password';
```
6. Give user "catalog" permission to "catalog" application database
```
postgres=# GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;
```
7. Quit postgreSQL `postgres=# \q`
8. Exit from user "postgres" 
```
exit
```

### Install git, clone and setup Catalog App project

1. Install Git using `sudo apt-get install git`
2. Move to `/var/www` directory `cd /var/www`
3. Clone catalog app from github repository `git clone https://github.com/mrukhlov/udacity_catalog.git`
4. Rename project folder to `CatalogApp`
5. Change `SQLite` connections to `PostgreSQL` with `create_engine('postgresql://catalog:password@localhost/catalog')`
6. Install `psycopg2`
7. Create database schema with `python database_setup.py`

### Create new virtual host

1. Create `CatalogApp.conf` with `vi /etc/apache2/sites-available/CatalogApp.conf`
2. Paste following code to configure virtual host
```
<VirtualHost *:80>
    ServerName 52.24.24.158
    WSGIScriptAlias / /var/www/CatalogApp/project.wsgi
    <Directory /var/www/Catalog/>
        Order allow,deny
        Allow from all
    </Directory>
    Alias /static /var/www/CatalogApp/static
    <Directory /var/www/CatalogApp/static/>
        Order allow,deny
        Allow from all
    </Directory>
    ErrorLog ${APACHE_LOG_DIR}/error.log
    LogLevel warn
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
3. Enable virtual host with `a2ensite CatalogApp`

### Create .wsgi file

1. Create `project.wsgi` file in the folder with CatalogApp
2. Add the following lines
```
import sys
sys.path.insert(0, '/var/www/CatalogApp')
from project import app as application
```

### Compile mod_wsgi with python 2.7 (for AWS)
1. `wget https://github.com/GrahamDumpleton/mod_wsgi/archive/4.4.21.tar.gz`
2. `tar -xzf 4.4.21.tar.gz`
3. `cd mod_wsgi-4.4.21`
4. `yum install httpd-devel`
5. `sudo yum install libtool`
6. `./configure --with-python=/usr/bin/python2.7`
7. `make; make install`

### Restart Apache
1. Restart Apache`service apache2 restart`

### References

[Database Roles and Privileges](https://www.postgresql.org/docs/9.0/static/user-manag.html)

[How To Secure PostgreSQL on an Ubuntu VPS](https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps)

[SQLAlchemy 1.1 Documentation: PostgreSQL](http://docs.sqlalchemy.org/en/latest/dialects/postgresql.html)

[mod_wsgi (Apache)](http://flask.pocoo.org/docs/0.12/deploying/mod_wsgi/)

[Creating Public keys](http://askubuntu.com/questions/53553/how-do-i-retrieve-the-public-key-from-a-ssh-private-key)

[Changing The SSH Port](https://www.liquidweb.com/kb/changing-the-ssh-port/)

[How do I permit specific users SSH access?](https://knowledgelayer.softlayer.com/learning/how-do-i-permit-specific-users-ssh-access)

[sshd_config](https://www.freebsd.org/cgi/man.cgi?sshd_config(5))

[Password Usage in Sudo](http://www.ducea.com/2006/06/18/linux-tips-password-usage-in-sudo-passwd-nopasswd/)