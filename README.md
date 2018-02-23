# Linux-Server-Configuration-UDACITY
By Areej adam , in fulfillment of Udacity's Full-Stack Web Developer Nanodegree


# Server details
IP address: 18.197.88.191

SSH port: 2200

URL: http://18.197.88.191

# Configuration changes
__________________________

### Step 1 : Launch your Virtual Machine with your Udacity account
```sudo ssh -vvv -i ~/Downloads/areej-key.pem ubuntu@18.197.88.191```


## 2. Add a new user and give grader permission
Add user ```grader``` with command: ```sudo adduser grader``` password ```grader```

- `touch /etc/sudoers.d/grader`
- `vim /etc/sudoers.d/grader`, write `grader ALL=(ALL:ALL) ALL` under `root ALL=(ALL:ALL) ALL` , save and quit (ctr+x , y , enter)

## 3. Update all currently installed packages
* Update the package `sudo apt-get update`

*  upgrade the installed packages ` sudo apt-get upgrade`

## 4. Set-up SSH keys for user grader
As root user do:`ssh-keygen`

```
mkdir /home/grader/.ssh
chown grader:grader /home/grader/.ssh
chmod 700 /home/grader/.ssh
cp /root/.ssh/authorized_keys /home/grader/.ssh/
chown grader:grader /home/grader/.ssh/authorized_keys
chmod 644 /home/grader/.ssh/authorized_keys
```
## 5. reload SSH using `service ssh restart`

## 6. Change the SSH port from 22 to 2200
by login as root 
    * `nano /etc/ssh/sshd_config`
    * change port from `22` to `2200`
    * change `PermitRootLogin without-password` to `PermitRootLogin no` . it is disable root login.
    * change `PasswordAuthentication` from no to yes.
    * add `AllowUsers grader` .
* restart the SSH service :
    `sudo service ssh restart`
    
## 7. Configure local Timezone to UTC
-Configure the time zone `sudo dpkg-reconfigure tzdata`

# 8.Add SSH port 2200
-  ```sudo nano /etc/ssh/sshd_config```

- and add: ```port 2200```

- Then reload :```sudo service ssh force-reload```


# 9.Configuration Uncomplicated Firewall (UFW)
allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)

```
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow ssh
sudo ufw allow 2200/tcp (Note: Changed SSH above to port 2200)
sudo ufw allow www
sudo ufw allow 123/udp
sudo ufw enable
```

# 10 . Installation :

Install Apache:
```sudo apt-get install apache2```

```sudo apt-get install libapache2-mod-wsgi```

Restart Apache 
`sudo service apache2 restart`

# 11. Install and configure the PostgreSQL database system
```
sudo apt-get update
sudo apt-get install postgresql postgresql-contrib
Create a PostgreSQL role named catalog and a database named catalog
sudo -u postgres -i
postgres:~$ createuser catalog
postgres:~$ createdb catalog
postgres:~$ psql catalog
postgres=# ALTER DATABASE catalog OWNER TO catalog;
postgres=# ALTER USER catalog WITH PASSWORD 'password';
postgres=# \q
postgres:~$ exit
```

# 12. Install Flask, SQLAlchemy, etc
Issue the following commands:
```
sudo apt-get install python-psycopg2 python-flask
sudo apt-get install python-sqlalchemy python-pip
sudo pip install oauth2client
sudo pip install requests
sudo pip install httplib2
sudo pip install flask-seasurf
Install Git version control software
sudo apt-get install git
```
# 13. Clone the repository that contains project
```
cd /var/www
sudo mkdir catalog
sudo chown -R grader:grader catalog
cd catalog
sudo git clone https://github.com/areejadam/Udacity-catalog-fullstack.git catalog
```

# 15. Configure the Catalog app
` cd /var/www/catalog/catalog `
* Change application.py to __init__.py
`mv application  __init__.py`

update the create_engine line:
	`python engine = create_engine('postgresql://catalog:password@localhost/catalog')`
* Update the create_engine line in __init__.py project.py and lotsofmenus.py too

# 16. Configure Google OAuth
* Google Authorization steps:
  * Go to [console.developer](https://console.developers.google.com/)
  * click on Credentails --> edit
  * add you hostname (http://ec2-18-197-88-191.us-west-2.compute.amazonaws.com  ) and public IP address (http://18.197.88.191) to Authorised   JavaScript origins.
  * add hostname (http://ec2-18-197-88-191-us-west-2.compute.amazonaws.com/oauth2callback) to Authorised redirect URIs.
  * update the client_secret.json file too(adding hostname and public IP address).

# 17. Configure Apache to serve the web application using WSGI
* Create the web application WSGI file.
```sudo nano /var/www/catalog/catalog.wsgi```

# 18. Add the following lines to the file, and save the file.
#!/usr/bin/python
```
import sys 
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/catalog/")
from catalog import app as application
application.secret_key = 'super_secret_key'
```
# 19. Update the Apache configuration file to serve the web application with WSGI.
```sudo nano /etc/apache2/sites-enabled/000-default.conf```

# 20 . Add the following line inside the <VirtualHost *:80> element, and save the file.
        ```
	ServerName 18.197.88.191
	ServerAdmin ubuntu@18.197.88.191
	WSGIScriptAlias / /var/www/catalog/catalog.wsgi
	<Directory /var/www/catalog/catalog>
		Order allow,deny
		Allow from all
	</Directory>
	Alias /static /var/www//catalog/catalog/static
	<Directory /var/www/catalog/catalog/static/>
		Order allow,deny
		Allow from all
	</Directory>
	ErrorLog ${APACHE_LOG_DIR}/error.log
	LogLevel warn
	CustomLog ${APACHE_LOG_DIR}/access.log combined```
 
###  21. Run the application:
* Create the datbase schema:
    `python database_setup.py`
    `python lotsofmenus.py`
* Restart Apache : `sudo service apache2 restart`
* in /var/www/catalog/catalog directory : execute -  `python __init__.py`
restart the apache2 server.

___________________________________________________________________________________________________________
# References
* Ask Ubuntu
* PosgreSQL Docs
* Apache Docs
* How To Install and Use PostgreSQL on Ubuntu 14.04
* Stackoverflow and the Readme's of other FSND students on Github.
* Special Thanks to anumsh for a very helpful README
