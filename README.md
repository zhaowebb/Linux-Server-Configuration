# Linux-Server-Configuration

## User Info
* public IP: 34.205.84.130
* SSH port: 2200
* URL to hosted web application: 34.205.84.130

## Update and upgrade installed packages
* sudo apt-get update
* sudo apt-get upgrade

## Create User grader
* Run "sudo adduser grader" and enter password

## Give user grader sudo access
* Copy a new file for grader: sudo cp /etc/sudoers.d/ubuntu /etc/sudoers.d/grader
* Edit this file to give user access: sudo nano /etc/sudoers.d/grader
* Change "root" to "grader" and save

## Generate key-pair and disable password-base login:

* On local terminal run command: ssh-keygen and specify location for key-pair
* On server: Run command mkdir .ssh
* Run command: touch .ssh/authorized_keys
* Copy and paste public key to authorized_keys
* Run command to set access permission: chmod 700 .ssh
* Run command: chmod 644 .ssh/authorized_keys

## Configure firewall and ports
* Run command: sudo nano /etc/ssh/sshd_config, find the Port line and edit it to 2200.
* Run: sudo ufw default deny incoming
* sudo ufw default allow outgoing
* Allow incoming TCP packets on port 2200 (SSH): sudo ufw allow 2200/tcp
* Allow incoming TCP packets on port 80 (HTTP): sudo ufw allow www
* Allow incoming UDP packets on port 123 (NTP): sudo ufw allow ntp
* sudo ufw enable
* sudo service ssh restart

## Install and configure Apache
* Install Apache web server: sudo apt-get install apache2
* Install mod_wsgi for serving Python apps from Apache and the helper package python-setuptools: sudo apt-get install python-setuptools libapache2-mod-wsgi
* Enable mod_wsgi: sudo a2enmod wsgi

## Clone the Catalog app repository from Github
* cd /var/www
* clone the catalog repository from Github: git clone https://github.com/zhaowebb/catalog
* Make a catalog.wsgi file to serve the application over the mod_wsgi. That file should look like this:
```
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog/")

from project import app as application
```

## Install virtual environment and the project's dependencies
* Install pip, the tool for installing Python packages: sudo apt-get install python-pip
* Install virtualenv: sudo pip install virtualenv
* Activate the virtual environment: source venv/bin/activate
* Install all the other project's dependencies: sudo -H pip install httplib2 request oauth2client sqlalchemy python-psycopg2

## Configure and enable a new virtual host
* Create a virtual host conifg file: $ sudo nano /etc/apache2/sites-available/catalog.conf.
* Paste in the following lines of code:
```
<VirtualHost *:80>
    ServerName 34.205.84.130
    ServerAdmin admin@34.205.84.130
    WSGIDaemonProcess catalog python-path=/var/www/catalog:/var/www/catalog/venv/lib/python2.7/site-packages
    WSGIProcessGroup catalog
    WSGIScriptAlias / /var/www/catalog/catalog.wsgi
    <Directory /var/www/catalog/catalog/>
        Order allow,deny
        Allow from all
    </Directory>
    Alias /static /var/www/catalog/catalog/static
    <Directory /var/www/catalog/catalog/static/>
        Order allow,deny
        Allow from all
    </Directory>
    ErrorLog ${APACHE_LOG_DIR}/error.log
    LogLevel warn
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

## Install and configure PostgreSQL
* Install PostgreSQL: sudo apt-get install postgresql postgresql-contrib
* Create a new user called 'catalog' with his password: # CREATE USER catalog WITH PASSWORD 'somepassword'
* Create the 'catalog' database owned by catalog user: sudo -u postgres creatdb -O catalog catalog;
* Inside the Flask application, the database connection is now performed with: engine = create_engine('postgresql://catalog:somepassword@localhost/catalog')

## Run catalog app
* Setup the database with: python db_setup.py
* Populate database: python catalog_init.py, don't forget to change "engine = create_engine('postgresql://catalog:somepassword@localhost/catalog')" in that file
* Run nano project.py, In the line of CLIENT_ID, add route info of json file '/var/www/catalog/client_secrets.json', add it in gconnect function also.
* In console developer website, add "http://34.205.84.130" in javascript origin, and download client_secrets.json again
* Run command: python project.py
* enter your public IP address in browser
* Done!



