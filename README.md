# Django production server with Nginx and Gunicorn

### Introduction
Django is an open-source Python framework that can be used for deploying Python applications. It comes with a development server to test your Python code in the local system. If you want to deploy a Python application on the production environment then you will need a powerful and more secure web server. In this case, you can use Gunicorn as a WSGI HTTP server and Nginx as a proxy server to serve your application securely with robust performance.

## Install Required Packages

First, you will need to install Nginx and other Python dependencies on your server. You can install all the packages with the following command:
```
sudo apt-get install vim python3-pip python3-dev libpq-dev curl nginx -y
```
Once all the packages are installed, start the Nginx service and enable it to start at system reboot:

```
sudo systemctl start nginx
sudo systemctl enable nginx
```

## Create new user and clone your repo

Change user to root with following command
```
sudo su
```
Create new user 
```
sudo adduser djangouser
```
Use the usermod command to add the user to the sudo group
```
sudo usermod -aG sudo djangouser
```
Change user and clone github repo
```
sudo su - djangouser
sudo git clone https://github.com/github-username/repo-name.git
```

## Install and Configure PostgreSQL

Next, you will need to install the PostgreSQL server on your server. You can install it with the following command:
```
sudo apt-get install postgresql postgresql-contrib -y
```
After the installation, log in to PostgreSQL shell with the following command:
```
sudo su - postgres
psql
```
Next, create a database and user for Django with the following command:

```
CREATE DATABASE djangodb;
CREATE USER djangouser WITH PASSWORD 'password';
```
Next, grant some required roles with the following command:
```
ALTER ROLE djangouser SET client_encoding TO 'utf8';
ALTER ROLE djangouser SET default_transaction_isolation TO 'read committed';
ALTER ROLE djangouser SET timezone TO 'UTC';
GRANT ALL PRIVILEGES ON DATABASE djangodb TO djangouser;
```
Next, exit from the PostgreSQL shell using the following command:
```
\q
```

## Create a Python Virtual Environment
Next, you will need to create a Python virtual environment for the Django project.

First, upgrade the PIP package to the latest version:
```
pip install --upgrade pip
```
Next, install the virtualenv package using the following command:
```
pip install python3.9-dev python3.9-venv
```
Next, create a directory for the Django project using the command below:
```
mkdir ~/django_project
```
Next, change the directory to django_project and create a Django virtual environment:
```
cd ~/django_project
python3.9 -m venv venv
```
Next, activate the Django virtual environment:
```
source venv/bin/activate
```
Next, install the Gunicorn and other packages with the following commands:
```
pip install gunicorn psycopg2-binary
pip install -r requirements.txt
```
## Configure Django

Edit the settings.py and define your database settings:
```
sudo vim ~/django_project/django_project/settings.py
```

Find and change the following lines:
```
ALLOWED_HOSTS = ['django.example.com', 'localhost']
DATABASES = {  
	'default': {     
		'ENGINE': 'django.db.backends.postgresql_psycopg2',       
		'NAME': 'djangodb',       
		'USER': 'djangouser',        
		'PASSWORD': 'password',        
		'HOST': 'localhost',       
		'PORT': '',    
	}
}
STATIC_URL = '/static/'
import os
STATIC_ROOT = os.path.join(BASE_DIR, 'static/')
```

Save and close the file then migrate the initial database schema to the PostgreSQL database:
```
python manage.py makemigrations
python manage.py migrate
```

Next, create an admin user with the following command:
```
python manage.py createsuperuser
```

Next, gather all the static content into the directory
```
python manage.py collectstatic
```

## Test the Django Development Server
Now, start the Django development server using the following command:
```
python manage.py runserver 0.0.0.0:8000
```
You should see the following output:
```
Watching for file changes with StatReloader
Performing system checks...
System check identified no issues (0 silenced).
June 22, 2021 - 11:15:57
Django version 3.2.4, using settings 'django_project.settings'
Starting development server at http://0.0.0.0:8000/
Quit the server with CONTROL-C.
```

Now, open your web browser and access your Django app using the URL http://django.example.com:8000/admin/. You will be redirected to the Django login page.
Provide your admin username, password and click on the Login. You should see the Django dashboard on the following page:

Now, go back to your terminal and press CTRL + C to stop the Django development server.

## Test Gunicorn
Next, you will need to test whether the Gunicorn can serve the Django or not. You can start the Gunicorn server with the following command:
```
gunicorn --bind 0.0.0.0:8000 django_project.wsgi
```

If everything is fine, you should get the following output:
```
[2021-06-22 11:20:02 +0000] [11820] [INFO] Starting gunicorn 20.1.0
[2021-06-22 11:20:02 +0000] [11820] [INFO] Listening at: http://0.0.0.0:8000 (11820)
[2021-06-22 11:20:02 +0000] [11820] [INFO] Using worker: sync
[2021-06-22 11:20:02 +0000] [11822] [INFO] Booting worker with pid: 11822
```
Press CTRL + C to stop the Gunicorn server.

Next, deactivate the Python virtual environment with the following command:
```
deactivate
```

## Create a Systemd Service File for Gunicorn
It is a good idea to create a systemd service file for the Gunicorn to start and stop the Django application server.

To do so, create a socket file with the following command:
```
sudo vim /etc/systemd/system/gunicorn.socket
```
Add the following lines:
```
[Unit]
Description=gunicorn socket
[Socket]
ListenStream=/run/gunicorn.sock
[Install]
WantedBy=sockets.target
```

Save and close the file then create a service file for Gunicorn:
```
sudo vim /etc/systemd/system/gunicorn.service
```
Add the following lines that match your Django project path:
```
[Unit]
Description=gunicorn daemon
Requires=gunicorn.socket
After=network.target

[Service]
User=root
Group=www-data
WorkingDirectory=/root/django_project
ExecStart=/root/django_project/venv/bin/gunicorn --access-logfile - --workers 5 --bind 
unix:/run/gunicorn.sock          django_project.wsgi:application

[Install]
WantedBy=multi-user.target
```

Save and close the file then set proper permission to the Django project directory:
```
sudo chown -R www-data:root ~/django_project
```
Next, reload the systemd daemon with the following command:
```
sudo systemctl daemon-reload
```
Next, start the Gunicorn service and enable it to start at system reboot:
```
sudo systemctl start gunicorn.socket
sudo systemctl enable gunicorn.socket
```
To check the status of the Gunicorn, run the command below:
```
sudo systemctl status gunicorn.socket
```
You should get the following output:
```
● gunicorn.socket - gunicorn socket     
Loaded: loaded (/etc/systemd/system/gunicorn.socket; enabled; vendor preset: enabled)     
Active: active (running) since Tue 2021-06-22 12:05:05 UTC; 3min 7s ago   Triggers: ● gunicorn.service     
Listen: /run/gunicorn.sock (Stream)     
CGroup: /system.slice/gunicorn.socket
Jun 22 12:05:05 django systemd[1]: Listening on gunicorn socket.
```

## Configure Nginx as a Reverse Proxy to Gunicorn Application
Next, you will need to configure Nginx as a reverse proxy to serve the Gunicorn application server.

To do so, create an Nginx configuration file:
```
sudo vim /etc/nginx/conf.d/django.conf
```

Add the following lines:
```
server {
        server_name django.example.com;

        client_body_buffer_size 200K;
        client_header_buffer_size 2k;
        client_max_body_size 100M;
        large_client_header_buffers 3 1k;

        client_body_timeout 5s;
        client_header_timeout 5s;

        location = /favicon.ico { access_log off; log_not_found off; }

        location /static {
                alias ~/django_project;
        }

        location /media {
                alias ~/django_project;
        }

        location / {
                include proxy_params;
                proxy_pass http://unix:/run/gunicorn.sock;
                proxy_set_header CLIENT-IP $remote_addr;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                limit_conn two 10;
        }

}

```
Save and close the file then verify the Nginx for any configuration error:
```
nginx -t
```
Output:
```
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```
Finally, restart the Nginx service to apply the changes:
```
systemctl restart nginx
```
Now, you can access the Django application using the URL http://django.example.com.


## SSL certificate with certbot
Visit https://certbot.eff.org/, choose your machine configuration and comlete written commands.


Ubuntu 20.04 + Nginx https://certbot.eff.org/lets-encrypt/ubuntufocal-nginx

To confirm that your site is set up properly, visit https://django.example.com/ in your browser and look for the lock icon in the URL bar.

## Automatic SSL certificate renewal crontab job

Open crontab with following command:
```
sudo crontab -e
```
Add the following line to the end of the file:
```
30 4 1 * * sudo cerbot renew --quiet
```

