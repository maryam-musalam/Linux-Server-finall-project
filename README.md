# Project: Linux Server Configuration

This project for Udacity full stack web development

## what this project all about ?

In this project we will host our project “item catalog” to the server, using amazon Lightsail.

## How to start with Amazon Lightsail ?

1- you need to create an account in amazon if you don’t already have one.<br>
2- log into https://lightsail.aws.amazon.com with your account that you have created.<br>
3 - click on create instance button.<br>
4- choose instance location, for me I picked Frankfurt, Zone A (eu-central-1a).<br>
5- select Linux/Unix platform.<br>
6- select OS only blueprint.<br>
7- select Ubuntu 16.04 LTS.<br>
8- choose your instance plane, go with the less price for this project.<br>
9-identify your instance, I called it “electronic_project”.<br>
10- press create instance.<br>
11- wait seconds till your instance be ready.<br>
12- press the title after it turn into blue to go to your instance that you just created.<br>
13 - from the nav bar select networking.<br>

14- in the networking page down to firewall section add the following then save:
A) custom - TCP - 2200
B) custom - UDP - 123

15- go to connect from the nav bar and press Connect using SSH.
16- in the terminal that opened for you enter the following:

A) sudo apt-get update <br>
B) sudo apt-get upgrade - Press Y - Keep the local version currently installed - Enter <br>
C) sudo nano /etc/ssh/sshd_config and add Port 2200 under Port 22 without deleting 22 port for now (Press ^W then ^x and say yes)<br>
D) sudo ufw status <br>

- you should see : status: inactive <br>
E) sudo ufw default deny incoming <br>
F) sudo ufw default allow outgoing <br>
G) sudo ufw allow ssh <br>
H) sudo ufw allow 2200/tcp <br>
I) sudo ufw allow 80/tcp <br>
J) sudo ufw allow 123/UDP <br>
K) sudo ufw enable <br>
L) press y to enable the ufw changes <br>
M) then from the instance page press Reboot Button and wait seconds. <br>

## How to check that our new port 2200 is working :

1- click connect nav and scroll down the page and press : Account page link. <br>
2- press download to download your private_key <br>
3- move your :LightsailDefaultKey-eu-central-1.pem to your .ssh folder in your PC. <br>
For me I did on my terminal : <br>

```
open ~/.ssh
```

Then I place it there/

4- now in your terminal type the following and then yes:

```
 ssh -i ~/.ssh/private_key.pem ubuntu@Public_ip -p 2200
```

\*you will find your public IP in your connect page

For me it was:

```
 ssh -i ~/.ssh/LightsailDefaultKey-eu-central-1.pem ubuntu@52.59.252.140 -p 2200
```

if you face this error after you press yes then do the following :

1- cd ~/.ssh <br>
2- sudo chmod 600 LightsailDefaultKey-eu-central-1.pem <br>
3- type your computer password <br>
4- cd ../ <br>
5- run the line again : ssh -i ~/.ssh/LightsailDefaultKey-eu-central-1.pem ubuntu@52.59.252.140 -p 2200 <br>

Now you should see your line in the terminal start with something like this : ubuntu@ip-172-26-6-115:~$ , if you see it then port 2200 works fine, you can go to networking nav and delete port 22 , as well as deleting it from configuration file same way as we added port 2200. Now at this point you will not be able to access the terminal from the “lightsial” but from the local terminal.

\*dont forget to deny 22 port as well

## create new user (grader) :

In the terminal ubuntu@ip-172-26-6-115:~$ do the following :

```
sudo adduser grader
```

I gave the grader password : 123 , I didnt enter any other info, then pressed yes/ y.

### give the new user(grader) access:

```
sudo nano /etc/sudoers
```

This will create new file to you called grader , pass the following into it under User privilege specification :

```
 grader ALL=(ALL:ALL)
```

Now to switch to user grader type the following:

```
sudo su - grader
```

And you will see your terminal changed to : grader@ip-172-26-6-115:~$

## create ssh-keygen

Open new terminal and follow the next steps:

```
ssh-keygen
```

Then give it the name with the location you want it save it to ex: /Users/mariam/.ssh/grader_user, and press enter twice to leave passphrase empty for now.

Now in ~/.ssh you will have two files : grader_user and grader_user.pub

next, in your user terminal : grader@ip-172-26-6-115:~$ do the following :

```
mkdir .ssh
touch .ssh/authorized_keys
```

Then in the other terminal run the following line to get the key :

```
cat .ssh/grader_user.pub  
```

Switch back to other terminal(grader@ip-172-26-6-115:~$ ) :

```
sudo nano .ssh/authorized_keys
```

Then pass the key and save it (^x - yes)

Then run the following :

```
chmod 700 .ssh
chmod 644 .ssh/authorized_keys
```

Now you will be able to log into user using terminal like this:

```
ssh -i ~/.ssh/grader_user grader@52.59.252.140 -p 2200
```

## Configure the local timezone to UTC :

```
sudo dpkg-reconfigure tzdata
```

Select none of the above then chose UTC

## install the needed setup :

1- Install Apache2

```
 sudo apt-get install apache2
```

You can test if it installed by run in your public ip in browser : http://52.59.252.140

2- install python

```
sudo apt-get install python
sudo apt-get install python-setuptools
```

3- install libapache2-mod-wsgi

```
sudo apt-get install libapache2-mod-wsgi
sudo apt-get install libapache2-mod-wsgi python-dev
```

Then run :

```
sudo service apache2 restart
```

4- install postgresql

```
sudo apt-get install postgresql
```

5- install git:
sudo apt-get install git

Then do the following:

```
cd /var/www/
sudo mkdir catalog
sudo chown -R grader:grader catalog
cd catalog
```

Then now clone the repo:

```
git clone https://github.com/maryam-musalam/catalog_items_for_project4.git catalog
```

Then make catalog.wsgi using:

```
sudo nano catalog.wsgi
```

And pass the following inside

```
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog/")

from catalog import app as application
application.secret_key = 'super_secret_key'
```

Make sure to replace your private key from your .pub folder that you got from ssh-keygen

Now you need to rename your .py folder to **init**.py :
```
cd catalog
sudo mv project.py **init**.py
```
Then I will edit client_secrets.json after I changed the Authorized JavaScript origins and Authorized redirect URIs to the new public ip adress:

```
sudo nano client_secrets.json
```

And pass the new credentials.

Now go back to /var/www/catalog and follow the rest :

```
cd ../
sudo apt-get install python-pip
sudo pip install virtualenv
sudo virtualenv venv
source venv/bin/activate
sudo chmod -R 777 venv
```

After running the above code you should see (venv) at the begging of your line line this (venv) grader@ip-172-26-6-115:/var/www/catalog$ .

The follow the rest :

```
sudo pip install flask
pip install Flask-SQLAlchemy
sudo pip install httplib2
sudo pip install oauth2client
pip install psycopg2
sudo pip install psycopg2-binary
sudo pip install requests
```

Now we need to edit our **init**.py to the new directory:

```
cd catalog
sudo nano __init__.py
```

Anywhere in the file where Python tries to open client_secrets.json or fb_client_secrets.json must be changed to its complete path ex: /var/www/catalog/catalog/client_secrets.json

Here:

```
CLIENT_ID = json.loads(
    open('/var/www/catalog/catalog/client_secrets.json', 'r').read())['web']['client_id']
APPLICATION_NAME = "Electronic App"
```

And here :

```
 oauth_flow = flow_from_clientsecrets('/var/www/catalog/catalog/client_secrets.json', scope='')
        oauth_flow.redirect_uri = 'postmessage'
```

## run the site:

First we need to run :

```
cd ../
sudo nano /etc/apache2/sites-available/catalog.conf
```

Then past this on it with you hostname and your public ip:

```
<VirtualHost *:80>
    ServerName [Public IP]
    ServerAlias [Hostname]
    ServerAdmin admin@35.167.27.204
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

Enable to virtual host: $ sudo service apache2 restart , then $ sudo a2ensite catalog.conf and DISABLE the default host $ sudo a2dissite 000-default.conf otherwise your site will not load with the hostname.

Now time to set the database :

```
$ sudo apt-get install libpq-dev python-dev
$ sudo apt-get install postgresql postgresql-contrib
$ sudo su - postgres
$ psql
```

And then as we asked in the project to make user catalog :

```
postgres=# CREATE USER catalog WITH PASSWORD [password];
postgres=# ALTER USER catalog CREATEDB;
postgres=# CREATE DATABASE catalog with OWNER catalog;
postgres=# \c catalog
catalog=# REVOKE ALL ON SCHEMA public FROM public;
catalog=# GRANT ALL ON SCHEMA public TO catalog;
catalog=# \q
$ exit
```

note: I picked password: password
Finally get back to grader and change create engine from :

```
engine = create_engine('sqlite:///electronic.db?check_same_thread=False’)
```

To :

```
engine = create_engine('postgresql://catalog:password@localhost/catalog' )
```

In every .db file that you have.

## run your databases:

```
python db.py
python electronicCategories.py
```

## to visit the site:

http://52.59.252.140.xip.io

Reference:
https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/managing-users.html
https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/create-deploy-python-flask.html
https://github.com/mulligan121/Udacity-Linux-Configuration
