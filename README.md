# Project 3: Linux Server Configuration
Submission attempt to Misk Udacity FSDN two project. 
* **Author** Ibrahim AlSaud
* **Instructor** Elham Jaffar
* **Date** 12/28/2018

# update 12/30/2019
Project no longer functional; please contact owner for more info.

# required info
* IP address: ~~3.120.149.134~~
* URL: ~~http://3.120.149.134.xip.io/~~


# Getting started
* ~~The site is up and visiting (http://3.120.149.134.xip.io/).~~
* To connect via ssh type the following in your terminal:
```ssh grader@<server ip> -p 2200 -i path/to/privkey```


# software installed
* all python2.7 packages required for https://github.com/ialsaud/Catalog
* postgres
* apache2
* wsgi-mod


# important directories and files
## user access
created two users developer, grader
```
sudo adduser <username>
```
added those two users to '/etc/sudoers.d/90-cloud-init-users'
```
<username> ALL=(ALL) NOPASSWD:ALL
```
configured '/etc/ssh/sshd_config' (only changed the following)
```
port 2200
PermitRootLogin no # to disable root account login
```
restart ssh
```
sudo service ssh restart
```

## firewall setup
```
Status: active

To                         Action      From
--                         ------      ----
80/tcp                     ALLOW       Anywhere
2200                       ALLOW       Anywhere
123                        ALLOW       Anywhere
80/tcp (v6)                ALLOW       Anywhere (v6)
2200 (v6)                  ALLOW       Anywhere (v6)
123 (v6)                   ALLOW       Anywhere (v6)
```



## webserver setup
installed apache2, postgresql and wsgi_mod
```
sudo apt-get install apache2
sudo apt-get install libapache2-mod-wsgi
sudo apt-get install postgresql
```

### setting up postgresql database (as sqlite only support local databases)
made a user, catalog, in psql for our site to logon from
```
developer$ sudo su - postgres
postgres$ psql
postgres=# CREATE USER catalog WITH PASSWORD 'catalog';
postgres=# ALTER USER catalog CREATEDB;
```
create database, catalog, for our code to use.
```
postgres=# CREATE DATABASE catalog WITH OWNER catalog;
```
configure the permission for the database.
```
postgres=# \c catalog
catalog=# REVOKE ALL ON SCHEMA public FROM public;
catalog=# GRANT ALL ON SCHEMA public TO catalog; 
```

## code setup
clone catalog project into '/etc/www/'' creating '/etc/www/Catalog'
```
git clone https://github.com/ialsaud/Catalog
```

change all instances of the following to
```
engine = create_engine('sqlite:///catalog.db') # from
engine = create_engine('postgresql://catalog:catalog@localhost/catalog') # to
```

### created virtual environment for the code.
ran ```sudo virtualenv venv``` to create virtual environment, which creates the following
```
/var/www/Catalog/venv
                     /bin
                     /local
                     /include
                     /lib
```
installed packages
```
sudo venv/bin/pip install -r requirements.txt
```

## running the site
* execute database code
```
sudo python database_setup.py
sudo python seeder.py
```
* created '/var/www/Catalog/application.wsgi' to run our 'application.py'
* created '/etc/apache2/sites-avalible/catalog.conf'
* ran ```sudo a2ensite catalog.conf```
* restarted apache ```sudo apache2ctl restart```
* monitored logs using ```tail -f /var/log/apache2/error.log```






## references

* disabling root
https://www.a2hosting.com/kb/getting-started-guide/accessing-your-account/disabling-ssh-logins-for-root

* setting up wsgi_mod 
https://hk.saowen.com/a/0a0048ca7141440d0553425e8df46b16cdf4c13f50df4c5888256393d34bb1b9
https://stackoverflow.com/questions/42050982/flask-wsgi-no-module-named-flask

* setting up postgresql database 
https://docs.sqlalchemy.org/en/latest/core/engines.html
https://github.com/harushimo/linux-server-configuration
https://askubuntu.com/questions/413585/postgres-password-authentication-fails