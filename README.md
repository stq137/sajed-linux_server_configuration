# Ubuntu-Linux Server Configuration

  
  

## Host name, IP

  

Host Name: http://ec2-3-126-249-3.eu-central-1.compute.amazonaws.com/

  

IP Address: 3.126.249.3

  

## Amazon Lightsail Server Set Up

  

1.  [Visit Amazon Lightsail](https://signin.aws.amazon.com/signin?redirect_uri=https%3A%2F%2Flightsail.aws.amazon.com%2Fls%2Fwebapp%3Fstate%3DhashArgs%2523%26isauthcode%3Dtrue&client_id=arn%3Aaws%3Aiam%3A%3A015428540659%3Auser%2Fparksidewebapp&forceMobileApp=0) and create a new AWS account:

  
  

2. After you log in, click 'Create Instance';

  

3. Select Platform (Linux/Unix) then select blueprint (OS Only) then(ubuntu 16.04 LTS)

  

4. Choose your payment then Scroll then name your instance (i leave it as default (ubuntu-1)) and click 'Create'

  

5. The instance needs about 2 mins to set up. After it is set up, you will see 'running'.

  

6. Click the status card and navigate to account

  

7. Download your private key from 'Account page' which is a .pem file.

  

8. Click the 'Networking' tab and under 'Firewall' section find 'Add another' at the bottom. Add port 123 (udp) and 2200(tcp).

  

## Server Configuration

  

1. Open terminal window on your local machine (we will refere to this window as first window) then save the downloaded `.pem` public key file into .ssh folder which is based in the home directory ~/.ssh:
`$ mv ~/Downloads/LightsailDefaultKey-eu-central-1.pem ~/.ssh/`

2. Secure public key while also making it accessible `$ chmod 600 ~/.ssh/LightsailDefaultKey-eu-central-1.pem`

  

3. Use this key to log into our Amazon Lightsail Server: `$ ssh -i ~/.ssh/LightsailDefaultKey-eu-central-1.pem ubuntu@3.126.249.3`

  

4. After you are connected to your instanse switch to 'root' user `$ sudo su -`

  

5. Then type `$ sudo adduser grader` to create another user 'grader' and give it a password.

  

6. Create a new file in the sudoers directory: `$ sudo nano /etc/sudoers.d/grader`. And give grader the super permisssion by pasting this code inside the editor `grader ALL=(ALL:ALL) ALL`.

In nano save with (control o, then hit the return key on your keyboard, then control x)

  

7. Run the following commands to update all packages and install finger package:

-  `$ sudo apt-get update`

-  `$ sudo apt-get upgrade`

-  `$ sudo apt-get install finger`

  

8. Open a new terminal window on your local machie (we will refere to this widow as second window) with (Command+N) and input `$ ssh-keygen -f ~/.ssh/grader_key.rsa` ,with 'ssh-keygen' you make a private and public key pair and you named this key 'grader'.

**note**: the system will ask you to enter passphrase, we will press enter and leave it empty.

  

9. Stay on the same(the second) Terminal window, input `$ cat ~/.ssh/grader_key.rsa.pub` to read the public key. Copy the public key.

  

10. Return to the first terminal window where you are logged into Amazon Lightsail as the root user, move to grader's folder by `$ cd /home/grader`

  

11. Create a .ssh directory (first terminal): `$ mkdir .ssh`

  

12. Create a file to store the public key(still in first terminal): `$ touch .ssh/authorized_keys`

  

13. Edit the authorized_keys file `$ nano .ssh/authorized_keys`, now past the public key you copied in step 9.

  

14. Change the permission: `$ sudo chmod 700 /home/grader/.ssh` then `$ sudo chmod 644 /home/grader/.ssh/authorized_keys`

  

15. Change the owner from root to grader: `$ sudo chown -R grader:grader /home/grader/.ssh`

  

16. Restart the ssh service: `$ sudo service ssh restart`

  

17.  `contrl c` then `control d` to disconnect from Amazon Lightsail server.

  

18. Log into the server as grader(that is log in via the second terminal window): `$ ssh -i ~/.ssh/grader_key.rsa grader@3.126.249.3`

  

19. Enforce the key-based authentication: `$ sudo nano /etc/ssh/sshd_config`. Find the *PasswordAuthentication* line and change text to `no`. After this, restart ssh again: `$ sudo service ssh restart`

  

20. Change the ssh port from 22 to 2200: `$ sudo nano /etc/ssh/sshd_config` Find the *Port* line and change `22` to `2200`. Restart ssh: `$ sudo service ssh restart`

  

21. Disconnect the server by `control c` then `control d` then log back through port 2200: `$ ssh -i ~/.ssh/grader_key.rsa -p 2200 grader@3.126.249.3` now loggin in via port 2200

  

22. Disable ssh login for *root* user to prevent attackers from making a fondering attept with root: `$ sudo nano /etc/ssh/sshd_config`. Find the *PermitRootLogin* line and edit to `no`. Restart ssh `$ sudo service ssh restart`

  

23. Configure Uncomplicated Firewall:

-  `$ sudo ufw allow 2200/tcp`

-  `$ sudo ufw allow 80/tcp`

-  `$ sudo ufw allow 123/udp`

-  `$ sudo ufw enable`

  

24. Configure the local timezone to UTC:

- Changed EC2 instance time zone to UTC:

	`sudo dpkg-reconfigure tzdata`

- choose 'None of the above', then UTC

- And set time sync with NTP:

	`sudo apt-get install ntp`

- Then added additional servers to **/etc/ntp.conf** file:

```
server ntp.ubuntu.com
server pool.ntp.org
```

- And reloaded the NTP service:

	`sudo service ntp reload`

  

## Deploy Catalog Application

  

Ensure you are logged in as grader. Should at anypoint a ubuntu password is requested simply ^d and use `sudo` to re-execute that command.

  

1. Install required packages

-  `$ sudo apt-get install apache2`

-  `$ sudo apt-get install libapache2-mod-wsgi python-dev`

-  `$ sudo apt-get install git`

  

2. Enable mod_wsgi (mod_wsgi package implements an Apache module which can host any Python web application which supports the Python WSGI specification.)`$ sudo a2enmod wsgi` and start the web server by `$ sudo service apache2 start` or `$ sudo service apache2 restart`

3. Enter your public IP address in your browser now and the apache2 default page should be loaded.

  

4. Create catalog folder to keep app and make grader owner and group of the folder

-  `$ cd /var/www`

-  `$ sudo mkdir catalog`

-  `$ sudo chown -R grader:grader catalog`

-  `$ cd catalog`

  

5. Clone the project from Github and make it web inaccessible:

-  `$ git clone https://github.com/stq137/catalog.git catalog` (so folder path to app will become `var/www/catalog/catalog`)

- To make the reposotory inaccessible:

	`$ cd nano /var/www/catalog/` then `$ sudo nano .htaccess`

- paste the following:

	`RedirectMatch 404 /\.git`

  

(The Web Server Gateway Interface (WSGI) is a specification for simple and universal interface between web servers and web applications or frameworks for the Python programming language.)

  

6. Create a .wsgi file in `/var/www/catalog/`: `$sudo nano catalog.wsgi` and add the following into this file

```
#!/usr/bin/python

import sys

import logging

logging.basicConfig(stream=sys.stderr)

sys.path.insert(0, "/var/www/catalog/")

from catalog import app as application

application.secret_key = 'super_secret_key'
```

  

7. In /var/www/catalog/catalog Rename the `application.py` to `__init__.py` as follows `mv application.py __init__.py`

  

(The venv module provides support for creating lightweight “virtual environments” with each virtual environment having its own Python binary. It allows the app have its own independent set of installed Python packages in its site directories.)

  

8. Install virtual environment

-  `$ sudo apt-get install python-pip`

**note:** you may need to do `sudo pip install --upgrade pip` to avoid possiple later errors.

-  `$ sudo pip install virtualenv`

-  `$ sudo virtualenv venv`

-  `$ source venv/bin/activate`

note:You should see a `(venv)` appears before your username in the command line.

-  `$ sudo chmod -R 777 venv`

  

- Your Project Folder structure now should looks like this:

```
/var/www/catalog

|-- catalog.wsgi

|__ /catalog

	|-- __init__.py

	|-- client_secrets.json

	|-- database_setup.py

	|-- /static

	|-- /templates

	|-- /venv

 ```

  

9. Install the Flask and other packages needed for this application

-  `$ sudo pip install Flask`

-  `$ sudo pip install httplib2 oauth2client sqlalchemy psycopg2 sqlaclemy_utils requests`

note: You may need to re-install outside the venv if some modules are missing such as `ImportError: No module named sqlalchemy`...sometimes there are some minimal compatibility issues between venv and some module installation, and if you still face issues with downloading any library it's a good prctice to search the web for solution.

  
  

10. Use the `nano __init__.py` command to change the `client_secrets.json` line to `/var/www/catalog/catalog/client_secrets.json` as follows `CLIENT_ID = json.loads(

open('/var/www/catalog/catalog/client_secrets.json', 'r').read())['web']['client_id']`

Ensure to look through `__ini__.py` for every instance of this change and replace as stated.

Also replace

`if __name__ == '__main__':

app.secret_key = 'super_secret_key'

app.debug = True

app.run(0.0.0.0, port=5000, port=8000)`

  

with

  

`if __name__ == '__main__':

app.secret_key = 'super_secret_key'

app.debug = True

app.run()`

  

11. Configure and enable the virtual host

note: sites-available/: This is an apache2 directory that contains all of the virtual host files that define different web sites. These will establish which content gets served for which requests.

  

-  `$ sudo nano /etc/apache2/sites-available/catalog.conf`

- Paste the following code and save

```

<VirtualHost *:80>

ServerName 3.126.249.3

ServerAlias http://ec2-3-126-249-3.eu-central-1.compute.amazonaws.com/

ServerAdmin admin@3.126.249.3

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

You can find your ServerAlias (host name) in this link: http://www.hcidata.info/host2ip.cgi

  

12. Now we need to set up the database

-  `$ sudo apt-get install libpq-dev python-dev`

-  `$ sudo apt-get install postgresql postgresql-contrib`

-  `$ sudo -u postgres -i`

  

You should see the username changed again in command line to `postgers=#`, and type `$ psql` to get into postgres command line

  

13. Create a user to create and set up the database. My database for example is named `catalog` and the user I am creating is also called `catalog`

-  `$ CREATE USER catalog WITH PASSWORD 'put your password here';`

-  `$ ALTER USER catalog CREATEDB;`

-  `$ CREATE DATABASE catalog WITH OWNER catalog;`

- Connect to database `$ \c catalog`

-  `$ REVOKE ALL ON SCHEMA public FROM public;`

-  `$ GRANT ALL ON SCHEMA public TO catalog;`

- Quit postgres as follows: `$ \c` and then `$ exit`

  

List the existing roles: `\du`. The output should be like this:

```

List of roles

Role name | Attributes | Member of

-----------+------------------------------------------------------------+-----------

catalog | Create DB | {}

postgres | Superuser, Create role, Create DB, Replication, Bypass RLS | {}

```

  

14. use `sudo nano` command to change all engine to `engine = create_engine('postgresql://catalog:your password@localhost/catalog)` e.g engine = create_engine('postgresql://catalog:137137@localhost/catalog')

Base.metadata.bind = engine.

Ensure to do this in the `database_setup.py` and the `__init__.py` and the `inital_data_population.py`.

  

15. Initiate the database: `python database_setup.py `

  

16. To get the Google+ authorization working:

- Go to the project on the Developer Console: https://console.developers.google.com/project

- Navigate to APIs & auth > Credentials

- add your host name and public IP-address to your 'Authorized JavaScript origins' and your host name + oauth2callback to 'Authorized redirect URIs', e.g. http://ec2-52-25-0-41.us-west-2.compute.amazonaws.com/oauth2callback then click `save`

- Download the new 'json' file then copy it's content

- now we want to update `client_secrets.json`:

	`$ cd /var/www/catalog/catalog/` then `sudo nano client_secrets.json`

- delete every thing in the file and paste the content of the new json file.

  

17. Restart Apache server `$ sudo service apache2 restart` and enter your host name or your public IP address into the browser.

  

18. If everything works, the application should come up, if getting an internal server error, check the Apache error files:

	`$ sudo tail -20 /var/log/apache2/error.log`

  

19. If default apache2 page persists, it means you have not enabled your site then execute `sudo a2ensite [name of app]`...'Name of app' here would be the folder the app is saved in inside /var/www/catalog/catalog for example mine is `sudo a2ensite catalog`

  
  

## References:

It's important to mention that this sheet based mainly on the follwing profound projects, so special thanks for :

- https://github.com/juvers/Linux-Configuration

- https://github.com/stueken/FSND-P5_Linux-Server-Configuration

- https://github.com/AmroYasser/Linux-Server-Configuration

  

Other Resources:

- https://www.digitalocean.com/community/tutorials/how-to-configure-the-apache-web-server-on-an-ubuntu-or-debian-vps

- https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-16-04

- https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-ubuntu-16-04

- https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps

- https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
