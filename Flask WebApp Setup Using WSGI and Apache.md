
![[assets/Pasted image 20230221143717.png]]

## Installing Dependence

### Debian

```sh
sudo apt-get install libapache2-mod-wsgi python-dev
```

```sh
pip3 install virtualenv
```

### Fedora

```sh
sudo dnf install python-dev python3-pip mod_wsgi
```

```sh
pip3 install virtualenv
```

## Setting up the File System

```sh
mkdir /var/www/webapp
mkdir /var/www/webapp/logs
mkdir /var/www/webapp/static
touch app.py && touch webapp.wsgi
cd /var/www/webapp
```

```sh
virtualenv venv
source venv/bin/activate
```

```sh
pip install flask
```

`app.py`  content

```python
from flask import Flask

app = Flask(__name__)

@app.route('/')
def hello():
    return 'Hello, World!'

if __name__ == "__main__":
	app.run()
```

`webapp.wsgi`  content

```python
#!/usr/bin/python
import sys
import logging

activate_this = "/var/www/webapp/venv/bin/activate_this.py"
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, '/var/www/web-app/webapp')

with open(activate_this) as file_:
    exec(file_.read(), dict(__file__=activate_this))

from app import app as application
```

## Setting up httpd

### Fedora

In  `/etc/httpd/conf.d/` create a new config file of any name `flaskwebapp.conf`

```xml
<VirtualHost *:80>

    ServerName <you_server_ip>

    ServerAdmin <email>

    WSGIDaemonProcess <webapp> user=apache group=apache threads=5
    WSGIScriptAlias / /var/www/webapp/webapp.wsgi

    <Directory /var/www/webapp/>
        Order allow,deny
        Allow from all
    </Directory>

	#FOR FLASK STATIC FOLDER

    Alias /static /var/www/web-app/webapp/static

    <Directory /var/www/webapp/static/>
        Order allow,deny
        Allow from all
    </Directory>

</VirtualHost>
```

### Debian

for Debian base system running apache2, create `.conf` in `/etc/apache2/sites-available/FlaskApp`

`.conf` content

```xml
<VirtualHost *:80>

    ServerName <you_server_ip>

    ServerAdmin <email>

    WSGIDaemonProcess <webapp> user=apache group=apache threads=5
    WSGIScriptAlias / /var/www/webapp/webapp.wsgi

    <Directory /var/www/webapp/>
        Order allow,deny
        Allow from all
    </Directory>

	#FOR FLASK STATIC FOLDER

    Alias /static /var/www/web-app/webapp/static

    <Directory /var/www/webapp/static/>
        Order allow,deny
        Allow from all
    </Directory>

</VirtualHost>
```

after creating the `.conf` run this `sudo a2ensite <naem_of_the_conf_without_.conf>`

## changing context's for Fedora

For project folder

```sh
cd /var/www
semanage fcontext -a -t httpd_sys_content_t './webapp(/.*)?'
restorecon -vvRF ./webapp
```

For the WSGI script

```sh
cd /var/www
semanage fcontext -a -t httpd_sys_script_exec_t './webapp/webapp.wsgi'
restorecon -vvRF ./webapp/webapp.wsgi
```

To check context

```sh
ls -ldZ <file_or_dir>
```

## Refresh the server

```sh
sudo systemctl stop httpd
sudo systemctl start httpd
sudo systemctl enable httpd
```

To check the Status

```sh
sudo systemctl status httpd
```