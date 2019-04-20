# Multiple-Django-Single-Server
Learn how to deal with multiple Django services in a single server

### Requirements
- Python
- Django
- Virtualenv
- Gunicorn

### Important
Django and dependencies installation is not covered by this tutorial.
I recommend you create different environments for each project.

If you never set up any Django production environment, I recommend you read this first: https://www.digitalocean.com/community/tutorials/how-to-set-up-django-with-postgres-nginx-and-gunicorn-on-ubuntu-18-04

### 1. Creating Gunicorn Service
```
cd /etc/systemd/system/
sudo vim project1.gunicorn.service
```

Paste the following code changing everything inside {} for your own values:
```
[Unit]
Description=gunicorn daemon
After=network.target

[Service]
User={user} 
Group=www-data
WorkingDirectory={/home/user/project1}
ExecStart={/home/user/project1/your-virtual-environment}/bin/gunicorn --access-logfile - --workers 3 --bind unix:{/home/user/project1/project1}.sock {project1}.wsgi:application

[Install]
WantedBy=multi-user.target
```

### 2. Creating Gunicorn Socket
```
cd /etc/systemd/system/
sudo vim project1.gunicorn.socket
```
Paste the following code changing everything inside {} for your own values:
```
[Unit]
Description=gunicorn socket

[Socket]
ListenStream={/home/user/project1/project1}.sock

[Install]
WantedBy=sockets.target
```
