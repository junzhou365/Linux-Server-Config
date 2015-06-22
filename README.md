# Linux-Server-Config
IP:52.25.51.208 

Catalog App:https://github.com/junzhou365/WebCatalog

## Steps

### Add a new user and grant admin privileges to it

`$ adduser grader` and `passwd grader`

`$ visudo`, then append `grader	ALL=(ALL:ALL) ALL` to `root	ALL=(ALL:ALL) ALL`

### Change the SSH port from 22 to 2200

`$ vim /etc/ssh/sshd_config`

Change the `PermitRootLogin without-password` to `PermitRootLogin no`

Add `AllowUsers grader` to the file

Change the port to 2200

Save the file.

`$ cat ~/.ssh/authorized_keys` and copy the output string

Then switch to grader by 'su - grader'

`$ cd ~/.ssh`, then `$ echo $PASTE_HERE >> authorized_keys`

After this, we can restart the ssh service `sudo service ssh restart`

Then exit the current session and reconnect by

`$ ssh -i ~/.ssh/udacity_key.rsa -p 2200 -v grader@52.25.51.208`

### Configure the UFW

Default: `$ sudo ufw default deny incoming` and `sudo ufw default allow outgoing`

Allow ssh port 2200: `$ sudo ufw allow 2222/tcp`

Allow http: `$ sudo ufw allow http`

Allow http: `$ sudo ufw allow ntp`

Enable the firewall: `$ sudo ufw enable`


### Install and configure Apache to serve a Python mod_wsgi app

First install apache: `$ sudo apt-get install apache2`

Then install wsgi:

`$ sudo apt-get install libapache2-mod-wsgi python-dev`

and

`sudo a2enmod wsgi`

Now we need to create a configuration file for out website:

`$ cd /etc/apache2/sites-available`, `$ cp 000-default.conf WebCatalog.conf`

Then `$ sudo vim WebCatalog.conf` and replace corresponding content with the following lines:

```
ServerName final.com
ServerAdmin admin@final.com
WSGIscriptAlias / /var/www/WebCatalog/WebCatalog.wsgi
<Directory /var/www/WebCatalog/>
        Order allow,deny
        Allow from all
</Directory>

ErrorLog /var/www/WebCatalog/error.log
CustomLog /var/www/WebCatalog/access.log combined
```

Save the file.

### Install and configure PostgreSQL


Install PostgreSQL`$ sudo apt-get install postgresql postgresql-contrib`

Create a superuser named 'grader':

`$ sudo -u postgres createuser --superuser grader`

Connect to PostgreSQL using default username 'postgres':

`$ sudo -u postgres psql`

According to [doc][1], "Client programs, by default, connect to the local host using your Ubuntu login name and expect to find a database with that name too. So to make things REALLY easy, use your new superuser privileges granted above to create a database with the same name as your login name".

We don't have to do this since our user would be `catalog`. But it would be easier for us to access to the PostgreSQL.

`$ sudo -u postgres createdb grader`

Create a user named `catalog` without privileges to create database:

`$ createuser catalog`

Connect to postgresql

`$ psql`

Grant login permission to `catalog`

`grader=# ALTER ROLE catalog LOGIN;`

Give `catalog` a password:

`grader=#\password catalog`

Then exit

`grader=#\q`

### Install Git, SQLAlchemy, and Flask

Use apt-get and pip to install them

### Setup Catalog App

Clone the project to `var/www/`

In the `var/www/WebCatalog/`, the `WebCatalog.wsgi` file has the following content:

```
#!/usr/bin/python

import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, '/var/www/WebCatalog')

from runserver import app as application
```

The method to access database is defined:
`engine = create_engine('postgresql+psycopg2://catalog:julian@localhost/catalog')`

In `/var/www/WebCatalog/Catalog`, create the `catalog` database:

`$ psql -f database_setup.sql`

Add some test items and categories

`$ python database_init.py`

### Run the server

First disable default html site:

`$ sudo a2dissite html`

Then enable our catalog site:

`$ sudo a2ensite WebCatalog`

Restart the apache2:

`$ sudo service apache2 restart`

Visit the website at the address, [52.25.51.208][2] or my personal website [http://www.junzhou365.com/catalog][3] on which I have deployed the project running on my own AWS. 

[1]: https://help.ubuntu.com/community/PostgreSQL
[2]: http://52.25.51.208
[3]: http://www.junzhou365.com/catalog





















