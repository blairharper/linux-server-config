# Linux Server

Basic configuration instructions for serving flask apps using Apache2 with postgresql.

### Server details

IP: 18.130.160.66
SSH Port: 2200
URL: http://18.130.160.66.xip.io/

### Prerequisites for catalog application

*git
*pip
*Apache2
*mod_wsgi
*libpq-dev
*PostgreSQL
*virtualenv
*Python 2.x
*httplib2
*Flask
*SQLAlchemy
*oauth2client
*Psycopg2

### Server configurations

1. Install updates by running:

```
sudo apt-get update
sudo apt-get upgrade
```

2. Configure firewall by running each of the commands below. Blocks all incoming except www(80) and SSH(2200).

```
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow ssh
sudo ufw allow 2200/tcp
sudo ufw allow www
sudo ufw deny 22
sudo ufw enable
```

If you are using a cloud service (like Amazon lightsail) you may also have to configure firewall settings in your control panel.

3. Change SSH port in `/etc/ssh/sshd_config`

4. Restart the SSH daemon: `sudo service ssh restart`

5. Install required dependencies as listed above down to and including Python.

6. Set up a virtual environment with by running `virtualenv ENVNAME`. Replace ENVNAME with a suitable name.

7. Activate the virtual environment by running `. ENVNAME/bin/activate` Use pip to install remaining dependencies. Replacing ENVNAME with the name set at step 6.

8. Create a virtual host file in `/etc/apache2/sites-available/` and insert:
```
<VirtualHost *:80>
                ServerName XX.XX.XX.XX
                ServerAdmin blair@blairly
                WSGIScriptAlias / /var/www/<YOUR APP>/<YOUR APP>.wsgi
                <Directory /var/www/<YOUR APP>/<YOUR DIR>/>
                        Order allow,deny
                        Allow from all
                        Options -Indexes
                </Directory>
                Alias /static /var/www/<YOUR APP>/<YOUR DIR>/static
                <Directory /var/www/<YOUR APP>/<YOUR DIR>/static/>
                        Order allow,deny
                        Allow from all
                        Options -Indexes
                </Directory>
                ErrorLog ${APACHE_LOG_DIR}/error.log
                LogLevel warn
                CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
Replacing <YOUR APP> and <YOUR DIR> as appropriate.

9. Create the directories and clone your application as above into <YOUR DIR>

10. Create a WSGI file in the <YOUR APP> directory containing:

```
activate_this = '/var/www/<YOUR APP>/<YOUR DIR>/<YOUR VIRTUAL ENV>/bin/activate_this.py'
execfile(activate_this, dict(__file__=activate_this))

#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/<YOUR APP>/")

from <YOUR DIR> import app as application
application.secret_key = 'YOUR APP KEY'

```

11. Create a user to manage your postgresql database and set up any required tables. Refer to postgresql documentation.

```
https://www.postgresql.org/docs/manuals/
```

12. Disable the default apache site by running the below and follow any prompts:

```
sudo a2dissite 000-default.conf
```

13. Rename your main application.py file to __innit__.py and run it. Resolve any errors and then change www-data to the owner of your application folder in /var/www using `sudo chown -R www-data:www-data YOURAPP/` from /var/www/

14. Restart apache by running: `sudo service apache2 restart`

### Helpful links:
https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
https://www.postgresql.org/docs/manuals/
https://www.digitalocean.com/community/tutorials/how-to-set-up-apache-virtual-hosts-on-ubuntu-16-04