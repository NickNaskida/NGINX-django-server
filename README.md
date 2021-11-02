# Django production server with NGINX and Gunicorn

### Introduction
Django is an open-source Python framework that can be used for deploying Python applications. It comes with a development server to test your Python code in the local system. If you want to deploy a Python application on the production environment then you will need a powerful and more secure web server. In this case, you can use Gunicorn as a WSGI HTTP server and Nginx as a proxy server to serve your application securely with robust performance.

## Install Required Packages

First, you will need to install Nginx and other Python dependencies on your server. You can install all the packages with the following command:
```
sudo apt-get install python3-pip python3-dev libpq-dev curl nginx -y
```
Once all the packages are installed, start the Nginx service and enable it to start at system reboot:

```
sudo systemctl start nginx
sudo systemctl enable nginx
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
Next, install the Django, Gunicorn, and other packages with the following command:
```
pip install django gunicorn psycopg2-binary
```
