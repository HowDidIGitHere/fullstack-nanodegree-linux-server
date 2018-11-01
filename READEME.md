# Linux Server Configuration

  The contents below illustrate how to configure a linux server that runs an already created item catalog application.

## Server Information

  IP Address: `54.202.123.18`

  SSH Port: `2200`

  URL: `http://ec2-54-202-123-18.us-west-2.compute.amazonaws.com`

# Configurations

  Everything from this point shows the configurations needed to initialize a functioning linux server.
  This server utilizes the following:

  &nbsp;&nbsp;&nbsp;
  Amazon Lightsail or DigitalOcean
  
## Uncomplicated Firewall Configuration

  Block all incoming connections:
  
  `$ sudo ufw default deny incoming`

  Allow all outgoing connections:
  
  `$ sudo ufw default allow outgoing`

  Allow incoming connections for SSH on port 22:
  
  `$ sudo ufw allow 22`

  Allow incoming connections for SSH on port 2200:
  
  `$ sudo ufw allow 2200/tcp`

  Allow incoming connections for HTTP on port 80:
  
  `$ sudo ufw allow www`

  Allow incoming connections for NTP on port 123:
  
  `$ sudo ufw allow ntp`

  Before enabling the firewall, make sure the configuration doesn't have any errors:
  
  `$ sudo ufw show added`

  Enable the firewall:
  
  `$ sudo ufw enable`

  Check the status of the firewall to make sure everything is how you want it:
  
  `$ sudo ufw status`

## SSH Port Change

  Edit the sshd_config file located in path `~/etc/ssh/` and change `Port 22` to `Port 2200`:
  
  `$ sudo nano /etc/ssh/sshd_config`
  
  Now block connections for SSH on port 22:
  
  `$ sudo ufw deny 22`
  
  And restart SSH service:
  
  `$ sudo service ssh restart`
  
## Update and upgrade any pre-installed packages 

  `$ sudo apt-get update`
  
  `$ sudo apt-get upgrade`
  
  If there is a message that says `System restart required` run the following command:
  
  `$ sudo reboot`
  
## Automate package updating

  Install the unattended-upgrades package:

  `$ sudo apt install unattended-upgrades`
  
  Enable automatic updates by editting the `20auto-upgrades` file located in the path `~/etc/apt/apt.conf.d/` to say:
  
  ```
  APT::Periodic::Update-Package-Lists "1";
  APT::Periodic::Download-Upgradeable-Packages "1";
  APT::Periodic::AutocleanInterval "15";
  APT::Periodic::Unattended-Upgrade "1";
  ```
  
  This will auto-update packages every 15 days.

## Add new user grader for Udacity Nanodegree Graders

  Add a new user named grader:
  
  `$ sudo useradd -m -s grader`
  
  Once the new user has been created, add the new grader to the sudo group:
  
  `$ sudo usermod -aG sudo grader`
  
  And edit the sudoers file to say:
  
  `grader ALL=(ALL) NOPASSWD:ALL`
  
  Now set up SSH keys for grader:
  
  ```
  $ sudo mkdir /home/grader/.ssh
  $ sudo chown grader:grader /home/grader/.ssh
  $ sudo chmod 700 /home/grader/.ssh
  $ sudo cp /root/.ssh/authorized_keys /home/grader/.ssh/
  $ sudo chown grader:grader /home/grader/.ssh/authorized_keys
  $ sudo chmod 644 /home/grader/.ssh/authorized_keys
  ```
  
  Edit the `/home/grader/.ssh/authorized_keys` file and replace the contents with the locally generated ssh key.
  
  You can then login from your local machine with the command:
  
  `$ ssh grader@54.202.123.18 -p 2200 -i ~/.ssh/grader`
  
## Disable root login

  Edit the `sshd_config` file in the `/etc/ssh/` path:
  
  `$ sudo nano /etc/ssh/sshd_config`
  
  Change the text that says `PermitRootLogin without-password` to `PermitRootLogin no`.
  
  In the same file, also uncomment the line that says `PasswordAuthentication no`.
  
  Once these changes are made, exit nano and restart ssh:
  
  `$ sudo service ssh restart`
  
  Now you should sign in as the grader.
  
## Install Apache2 and mod_wsgi

  Install Apache2:
  
  `$ sudo apt-get install apache2`
  
  Install mod_wsgi:
  
  `$ sudo apt-get install libapache2-mod-wsgi`

## Install Postgresql and Create a Database

  Install Postgresql:

  `$ sudo apt-get install postgresql postgresql-contrib`
  
  Create a psql user named catalog:
  
  `$ sudo -u postgres createuser -P catalog`
  
  You will need to create a password for this new user.
  
  Remember this password as you will need it for later.
  
  Now create a new database for the user also named catalog:
  
  `$ sudo -u postgres createdb -O catalog catalog`

## Install Flask and SQLAlchemy

  Install all the following packages:

  ```
  $ sudo apt-get install python-psycopg2 python-flask
  $ sudo apt-get install python-sqlalchemy python-pip
  $ sudo pip install --upgrade pip
  $ sudo pip install Flask
  $ sudo pip install sqlalchemy
  $ sudo pip install random
  $ sudo pip install oauth2client
  $ sudo pip install requests
  $ sudo pip install httplib2
  ```
  
  If necessary, use `pip3` if the application is written in python3. 

## Install Git and Clone the Item Catalog Application

  Install git:

  `$ sudo apt-get install git`
  
  Use git to clone the already create application:
  
  ```
  $ cd /var/www/
  $ sudo mkdir fullstack-nanodegree-vm
  $ sudo chown www-data:www-data fullstack-nanodegree-vm
  $ sudo -u www-data git clone https://github.com/HowDidIGitHere/fullstack-nanodegree-vm.git fullstack-nanodegree-vm
  ```
  
  Make .git files inaccessable by writing the code `RedirectMatch 404 /\.git` in a `.htaccess` file:
  
  `$ sudo nano .htaccsess`

## Prepare the Item Catalog Application for deployment

  Create a new folder at the end of the application path and rename the application.py file:
  
  ```
  $ cd /var/www/fullstack-nanodegree-vm/vagrant/catalog
  $ sudo mkdir catalog
  $ sudo mv client_secrets.json database_setup.py application.py itemcatalogwithusers.db populate_database.py README.md static templates /var/www/fullstack-nanodegree-vm/vagrant/catalog/catalog
  $ cd catalog/
  $ sudo mv application.py __init__.py
  ```
  
  Change all mentions of `engine = create_engine('sqlite:///itemcatalogwithusers.db')` to `engine = create_engine('postgresql://catalog:DATABASE-PASSWORD@localhost/catalog')`.
  
  And also chane all mentions of `client_secrets.json` to `/var/www/fullstack-nanodegree-vm/vagrant/catalog/catalog/client_secrets.json`.
  
  Edit __init__.py:
  
  `$ sudo nano __init__.py`
  
  Edit database_setup.py:
  
  `$ sudo nano database_setup.py`
  
  Edit populate_database.py:
  
  `$ sudo nano populate_database.py`
  
  Create a wsgi file named catalog.wsgi at the path `/var/www/fullstack-nanodegree-vm/vagrant/catalog/`:
  
  `$ sudo touch catalog.wsgi`
  
  Add the following code in catalog.wsgi:
  
  ```
  #!/usr/bin/env/ python
  import sys
  import logging
  logging.basicConfig(stream=sys.stderr)
  sys.path.insert(0, "/var/www/fullstack-nanodegree-vm/vagrant/catalog/")
  sys.path.append("/var/www/fullstack-nanodegree-vm/vagrant/catalog/catalog/")
  from catalog import app as application
  application.secret_key = 'CREATOR-MADE-SECRET-KEY'
  ```
  
  Create a virtual host conf file:
  
  `$ sudo nano /etc/apache2/sites-available/catalog.conf`
  
  Inside this file add the following code:
  
  ```
  <VirtualHost *:80>
        ServerName http://54.202.123.18.xip.io
        ServerAdmin YOUR-ADMIN-EMAIL
        WSGIScriptAlias / /var/www/fullstack-nanodegree-vm/vagrant/catalog/catalog.wsgi
        <Directory /var/www/fullstack-nanodegree-vm/vagrant/catalog/catalog/>
                Require all granted
        </Directory>
        Alias /static /var/www/fullstack-nanodegree-vm/vagrant/catalog/catalog/static
        <Directory /var/www/fullstack-nanodegree-vm/vagrant/catalog/catalog/static/>
                Require all granted
        </Directory>
        ErrorLog ${APACHE_LOG_DIR}/error.log
        LogLevel warn
        CustomLog ${APACHE_LOG_DIR}/access.log combined
  </VirtualHost>
  ```
  
  Disable the default virtual host:
  
  `$ sudo a2dissite 000-default.conf`
  
  Enable the custom virtual host:
  
  `$ sudo a2ensite catalog.conf`
  
  Restart Apache2:
  
  `$ sudo service apache2 restart`

## Update Google Oauth client secrets

  Edit the `client_secrets.json` file and change the javascript_origins to:
  
  `"javascript_origins":["http://54.202.123.18", "http://ec2-54-202-123-18.us-west-2.compute.amazonaws.com"]`
  
  Go to `https://console.developers.google.com` and change the Authorized JavaScript Origins to match the changes made in the client_secrets.json file.

## Populate the database

  ```
  $ python database_create.py
  $ python populate_database.py
  ```
  
## Run __init__.py

  Run the application:

  `$ python /var/www/fullstack-nanodegree-vm/vagrant/catalog/catalog/__init__.py`
  
  Restart Apache2:
  
  `$ sudo service apache2 restart`
  
# After all these commands are made, the web application is now live and functioning.

## Special Thanks to:

  Steven Wooding https://github.com/SteveWooding/fullstack-nanodegree-linux-server-config
  
  
  
  Stackoverflow
  
  
  
  Udacity Knowledge
