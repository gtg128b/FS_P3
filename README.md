
#Item Catalog WSGI: Project 3

## Web application that provides a list of items within a variety of categories
## Includes third party authentication via Google OAuth
## Hosted on AWS Linux server with Apache2

  The program achieves the following goals including CRUD ops, OAuth, and JSON endpoints

  1. Homepage displays all current categories with a list of the latest items
  2. Selecting a category displays the list of items in the category (READ)
  3. Selecting an item displays the item's details
  4. After logging in via Google OAuth and permitting Google user profile access:
	    * User can add a new item (CREATE)
	    * User can edit an item (UPDATE)
	    * User can delete an item (DELETE)
	    * Optional ability to add a Category provided; however, I chose not to allow Category edit or delete
  5. JSON endpoints are made available including Catalog, Category, Item

 The Project 3 goals were achieved:

  1.	Grader account with SSH key
  2.	Disable remote login
  3.	Grant sudo to grader
  4.	SSH 2200, HTTP 80, NTP 123 only
  5.	Authenticate SSH via RSA keys only
  6.	All system pkgs updated
  7.	SSH hosted on non-default port (2200)
  8.	Web server responds on port 80
  9.	Database (SQLLite) serves data
  10.	Serves Item Catalog as WSGI
  11.	README IP Addr, URL, sw summary, summary config, list of 3rd party resources

## Prerequisites

  * If you want to add Catagories or Items to the system, you must have a [Google account](https://accounts.google.com/signup/v2/webcreateaccount?continue=https%3A%2F%2Fwww.google.com%2F%3Fhl%3Den-US&hl=en&gmb=exp&biz=false&flowName=GlifWebSignIn&flowEntry=SignUp)
  * If you need to log into the server, you will login as grader but you need an SSH key and passphrase.  Located at: 18.222.73.127:80
  * A browser such as Edge, Firefox, or Chrome to find: http://18.222.73.127.xip.io/login

## Program prepwork

##UPDATE SYSTEM:
```
sudo apt-get update
sudo apt-get upgrade
sudo apt-get autoremove
sudo apt-get install finger
```

##ADD USERS
```
sudo adduser student
sudo adduser grader
```

##ADD SUDO
```
cd /home/ubuntu
sudo echo "student ALL=(ALL) NOPASSWD:ALL" > student
sudo cp student /etc/sudoers.d
sudo rm student
sudo echo "grader ALL=(ALL) NOPASSWD:ALL" > grader
sudo cp grader /etc/sudoers.d
sudo rm grader

cd /home/student
sudo mkdir .ssh
sudo touch .ssh/authorized_keys
sudo chown student:student .ssh
sudo chown student:student .ssh/authorized keys
sudo chmod 700 .ssh
sudo chmod 644 .ssh/authorized_keys
sudo nano .ssh/authorized_keys

(add key)

cd /home/grader
sudo mkdir .ssh
sudo touch .ssh/authorized_keys
sudo chown grader:grader .ssh
sudo chown grader:grader .ssh/authorized keys
sudo chmod 700 .ssh
sudo chmod 644 .ssh/authorized_keys
sudo nano .ssh/authorized_keys

(add key)
```

#DISALLOW REMOTE
```
sudo nano /etc/ssh/sshd_config
	(change allow password to no
	 make sure port is 2200)
```

#CONFIGURE CONNECTIVITY
```
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 2200/tcp
sudo ufw allow www
sudo ufw allow 123/tcp
sudo ufw enable

sudo ufw status

sudo service ssh restart
```

#SET TZ
```
sudo timedatectl set-timezone America/New_York
```

#INSTALL REQUIREMENTS
```
sudo apt-get -qqy install python3 python3-pip
sudo pip3 install flask packaging oauth2client passlib flask-httpauth
sudo pip3 install sqlalchemy flask-sqlalchemy bleach requests
sudo pip3 install authlib==0.11
sudo apt-get install libapache2-mod-wsgi-py3
sudo apt-get install apache2
```

#Item Catalog updates:
  1. Update client_secrets.json with new Domain and Redirect URI
  2. Add path for py files:
    ```
  	import sys
  	sys.path.append('/var/www/catalog/catalog')
    ```
  3. Add path for db file:
      ```
	    engine = create_engine(
  	    'sqlite:////var/www/catalog/catalog/catalog.db',
  	    connect_args={'check_same_thread': False},
  	    echo=True)
      ```
  4. Add new redirect URI for gconnect method
  5. Update redirect urls in templates for new Domain
  6. Rename application.py to __init__.py
  7. Re-run catalog_db_setup.py and add_data.py for clean start


#USE SCP TO UPLOAD FILES (GIT REPO purged) and set file permissions
```
scp -r -i ~/.ssh/linuxCourse -P 2200 catalog ubuntu@18.222.73.127:/home/ubuntu
ssh ubuntu@18.222.73.127 -p 2200 -i ~/.ssh/linuxCourse
sudo cp -r /home/ubuntu/catalog /var/www/html/catalog
sudo chown -R www-data:www-data /var/www/catalog
#sudo chown -R student:student /var/www/catalog
sudo chmod -R 700 /var/www/catalog
```

#APACHE CONFIGURATION
```
sudoedit /etc/apache2/sites-available/catalog.conf

	## /etc/apache2/sites-available/catalog.conf
	<VirtualHost [asterisk]:80>
	 ServerName 18.222.73.127.xip.io
	 WSGIDaemonProcess catalog user=student group=student threads=5
	 WSGIScriptAlias / /var/www/catalog/catalog.wsgi
	<Directory /var/www/catalog/catalog/>
	 WSGIProcessGroup catalog
	 WSGIApplicationGroup %{GLOBAL}
	 WSGIScriptReloading On
	 Require all granted
	</Directory>
	</VirtualHost>
```

# Disable Default conf
```
sudo a2dissite 000-default.conf
```

# Enable catalog.conf
```
sudo a2ensite catalog.conf
```

#WSGI file catalog.wsgi
```
nano /var/www/catalog/catalog.wsgi

	#!/usr/bin/python3
	import sys
	import logging
	logging.basicConfig(stream=sys.stderr)
	if sys.version_info[0]<3:       # require python3
		raise Exception("Python3 required! Current (wrong) version: '%s'" % sys.version_info)
	sys.path.insert(0, '/var/www/catalog/')
	from catalog import app as application
	application.secret_key = 'wherever you go there you are'


sudo service apache2 restart
```

#UPDATE GOOGLE AUTH Domain and Redirect URI
Add domain:
 	http://18.222.73.127.xip.io
Add redirect URI:
	http://18.222.73.127.xip.io:80 	
	http://18.222.73.127.xip.io:80/gconnect 	
	http://18.222.73.127.xip.io:80/login


## Program execution

    * Visit the web site, http://18.222.73.127.xip.io:80


## Known issues

  I've seen problems logging out of the application if/when user is logged in for an extended period.

## Authors

  **Philip Ellis**

## Credits & Updates

  * Code was carved from the FullStack Nanodegree Training
  * Insights for OAuth were used from Ellis E [comment](https://knowledge.udacity.com/questions/56880)
  * Example data was harvested from Wikipedia.org

## References
  WSGI:
  http://terokarvinen.com/2016/deploy-flask-python3-on-apache2-ubuntu
  https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
  https://www.thegeekstuff.com/2011/07/apache-virtual-host/
  GOOGLE:
  https://console.developers.google.com
