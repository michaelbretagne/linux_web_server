# Linux Web Server

This application was created as my submission for the project 7 of Udacity's Full Stack NanoDegree program.

## Project Description:
Installation of a Linux server and prepare it to host web applications

## Development Environment Information
Public IP Address: 52.37.68.228

## Steps accomplished

1.  Launch your Virtual Machine and log in.

 - Create a new development environment

 - Get and download the private key from [Udacity account](https://www.udacity.com/account#!/development_environment)
 - Move the private key file into the folder ~/.ssh
 `mv ~/Downloads/udacity_key.rsa ~/.ssh/`

 - Open terminal and set a file permission<br>
 `chmod 600 ~/.ssh/udacity_key.rsa`

 - SSH into server using the private key
 `ssh -i ~/.ssh/udacity_key.rsa root@52.37.68.228`<br><br>

2. Create a new user and grant this user sudo permissions.
 - Create a new user name grader<br>
 `adduser grader`<br>

 - Create a grader sudoers file<br>
 `nano /etc/sudoers.d/grader`<br>
 
 - Add the following line and save<br>
 `grader ALL=(ALL) ALL`<br><br>
 
3. Update all currently installed packages<br>
 - Switch to the grader user<br>
 `su - grader`<br>

 - Update and upgrade packages<br>
 `sudo apt-get update`<br>
 `sudo apt-get upgrade`<br>
 `sudo apt-get autoremove`<br>

 Configure ssh authentification<br>
 - Exit from ssh<br>
 `exit`<br>

 - Create ssh key. Name it authkey into ~/.ssh/linux_server file<br>
 `ssh-keygen`<br>

 - Copy the new key<br>
 `cat ~/.ssh/linux_server/authkey.pub`<br>

 - SSH into server and switch to greader<br>
 `ssh -i ~/.ssh/udacity_key.rsa root@52.37.68.228`<br>
 `su - grader`<br>

 - Create a new file .ssh<br>
 `mkdir .ssh`<br>

 - Paste the new key into authorized_keys file<br>
 `touch .ssh/authorized_keys`<br>
 `nano .ssh/authorized_keys`<br>

 - Set permission<br>
 `sudo chmod 700 .ssh`<br>
 `sudo chmod 644 .ssh/authorized_keys`<br>

 - Exit from ssh<br>
 `exit`<br>

 - Connect as grader into server using the authentification key<br>
 `ssh -i ~/.ssh/authkey grader@52.37.68.228`<br><br>
 
4. Configure the local timezone to UTC.
 - Open the timezone selection dialog<br>
 `sudo dpkg-reconfigure tzdata`<br>
 Select `None of the above`<br>
 Select `UTC`<br><br>
 
5. Change the SSH port from 22 to 2200 and secure server by disabling root login.
 - Open ssh config file<br>
 `sudo nano /etc/ssh/sshd_config`<br>

 - Edit file
 change `port 22` to `port 2200`<br>
 change `PermitRootLogin without-passwod` to `PermitRootLogin no`<br>

 - Reload SSH<br>
`sudo service ssh restart`<br><br>
 
6. Configure the Uncomplicated Firewall (UFW).<br>
 - Block all incoming traffic and allow all outgoing traffic<br>
 `sudo ufw default deny incoming`<br>
 `sudo ufw default allow outgoing`<br>

 - Allow SSH, HTTP and NTP connections<br>
 `sudo ufw allow 2200`<br>
 `sudo ufw allow 123`<br>
 `sudo ufw allow 80`<br>

 - Enable firewall<br>
 `sudo ufw enable`<br><br>
 
 7. Install and configure Apache to serve a Python mod_wsgi application.<br>
 - Install Apache web server<br>
 `sudo apt-get install apache2`<br>

 - Install mod_wsgi<br>
 `sudo apt-get install python-setuptools libapache2-mod-wsgi`<br>

 - Restart the Apache server<br>
 `sudo service apache2 restart`<br>
 
 Configure a Flask Application on the server<br>
 - Create new directories for the app<br>
 `sudo mkdir /var/www/FlaskApp`<br>
 `sudo mkdir /var/www/FlaskApp/catalog`<br>

 - Move into catalog directory<br>
 `cd /var/www/FlaskApp/catalog`<br>

 - Create two subdirectories named static and templates<br>
 `sudo mkdir static templates`<br>

 - Create a new file that contain the Flask application logic<br>
 `sudo nano __init__.py`<br>

 - Paste in the following code for testing purpose<br>

 ```from flask import Flask <br>
 app = Flask(__name__)  
 @app.route("/")  
 def hello():  
 return "It works!!"  
 if __name__ == "__main__":  
 app.run()```

 - Install a virtual environment
 `sudo apt-get install python-pip`
 `sudo pip install virtualenv`

 - Create a new virtual environment and activate it
 `sudo virtualenv venv`
 `source venv/bin/activate`

 - Install Flask in (venv)
 `pip install Flask`

 - Run app
 `python __init__.py`

 - Deactivate the environment
 `deactivate`
 
 Configure and Enable a New Virtual Host
 - Create app congig file
 `sudo nano /etc/apache2/sites-available/catalog.conf`

 - Add the following code into catalog.conf
 ```<VirtualHost *:80>
    ServerName 52.37.68.228
    ServerAdmin admin@52.37.68.228
    WSGIScriptAlias / /var/www/FlaskApp/flaskapp.wsgi
    <Directory /var/www/FlaskApp/catalog/>
            Order allow,deny
            Allow from all
    </Directory>
    Alias /static /var/www/FlasApp/catalog/static
    <Directory /var/www/FlaskApp/catalog/static/>
            Order allow,deny
            Allow from all
    </Directory>
    ErrorLog ${APACHE_LOG_DIR}/error.log
    LogLevel warn
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>```

- Enable the virtual host
`sudo a2ensite catalog`

- Create the .wsgi File
`sudo nano /var/www/FlaskApp/flaskapp.wsgi`

- Add the following code into flaskapp.wsgi
 ```#!/usr/bin/python
 import sys
 import logging
 logging.basicConfig(stream=sys.stderr)
 sys.path.insert(0,"/var/www/FlaskApp/")

 from catalog import app as application
 application.secret_key = 'Add your secret key'```


 - Restart Apache<br>
 `sudo service apache2 restart<br>` 
 
 8. Install and configure PostgreSQL<br>
 - Activate the virtual environment (venv)<br>
 `cd /var/www/FlaskApp/catalog`<br>
 `source venv/bin/activate`<br>

 - Install PostgreSQL and module required for the app<br>
 `sudo apt-get install postgresql postgresql-contrib`<br>
 `pip install httplib2`<br>
 `pip install requests`<br>
 `sudo pip install oauth2client`<br>
 `sudo pip install sqlalchemy`<br>
 `sudo apt-get install python-psycopg2`<br>

 - Switch to the new user postgres created when installing PostgreSQL<br>
 `sudo -i -u postgres`<br>

 - Launch PostgreSQL<br>
 `psql`<br>

 - Create user named catalog<br>
 `CREATE USER catalog WITH PASSWORD 'password_db';`<br>

 - Allow the user to create tables<br>
 `ALTER USER catalog CREATEDB;<br>`

 - Create database<br>
 `CREATE DATABASE catalog WITH OWNER catalog;`<br>

 - Switch to the user catalog and revoke all right<br>
 `\c catalog`<br>
 `REVOKE ALL ON SCHEMA public FROM public;`<br>

 - Grant access to the user catalog<br>
 `GRANT ALL ON SCHEMA public TO catalog;`<br>

 - Exit PSQl<br>
 `\q`<br>
 `exit`<br>
 
 9. Install git, clone and set up [Catalog App project](https://github.com/michaelbretagne/beer_catalog).
 - Install Git<br>
 `sudo apt-get install git<br>`

 - Clone [Catalog App project](https://github.com/michaelbretagne/beer_catalog)<br>
 `cd /var/www/FlaskApp/`<br>
 `git clone https://github.com/michaelbretagne/beer_catalog.git`<br>

 - Block acces to the GitHub repository<br>
 `sudo nano .htaccess`<br>

 - Add the following code into .htaccess<br>
 `RedirectMatch 404 /\.git`<br>

 - Rename app<br>
 `mv beer_catalog catalog`<br>

 - Rename the main python file<br>
 `mv catalog/application.py catalog/__init__.py<br>`
 
 Modify catalog app to allow OAuth login and postgreSQL to work properly<br>
 - Modify code to match the following code into datasetup.py, brewery_populate.py and __init__.py<br>
 `engine = create_engine("postgresql://catalog:password_db@localhost/catalog")`<br>

 - In the [Google Developer Console](https://console.developers.google.com/project), modify Authorized redirect URIs credential informations under API Manager<br>
 `http://ec2-52-37-68-228.us-west-2.compute.amazonaws.com/oauth2callback`<br>

 - In the [Facebook Developer site](https://developers.facebook.com/apps/), modify OAuth redirect URIs setting under Facebook Login<br>
 `http://ec2-52-37-68-228.us-west-2.compute.amazonaws.com/oauth2callback\`<br>

 - Add the client secrets and edit the filepaths<br>
 Into /var/www/FlaskApp/catalog/__init__.py change<br>
 `CLIENT_ID = json.loads(open('client_secrets.json', 'r').read())['web']['client_id']`<br>
    to<br>
 `CLIENT_ID = json.loads(
    open('/var/www/FlaskApp/catalog/client_secrets.json', 'r').read())['web']['client_id']`<br>
    and also change<br>
 `app_id = json.loads(open('fb_client_secrets.json','r').read())['web']['app_id']`<br>
 `app_secret = json.loads(open('fb_client_secrets.json', 'r').read())['web']['app_secret']`<br>
 to<br>
 `app_id = json.loads(open('/var/www/FlaskApp/catalog/fb_client_secrets.json','r').read())['web']['app_id']`<br>
 `app_secret = json.loads(open('/var/www/FlaskApp/catalog/fb_client_secrets.json', 'r').read())['web']['app_secret']`<br>