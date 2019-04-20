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
Save and close file.


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
Save and close file.


### 3. Enabling and Starting Socket
This will create the socket file at {/home/user/project1/} now and at boot. When a connection is made to that socket, systemd will automatically start the gunicorn.service to handle it:
```
sudo systemctl start project1.gunicorn.socket
sudo systemctl enable project1.gunicorn.socket
```

To check if everything is ok use the following commands:
```
sudo systemctl status project1.gunicorn.socket
sudo systemctl status gunicorn
```

If you got any problem, use the following commands check logs
```
sudo journalctl -u gunicorn
```

After change socket or service file is necessary reload daemon and gunicorn typing:
```
sudo systemctl daemon-reload
sudo systemctl restart gunicorn
```


### 4. Nginx Settings

Create a server block for your website. I will use only http to this tutorial, and if you want to set up https see this other link: https://www.digitalocean.com/community/tutorials/how-to-secure-nginx-with-let-s-encrypt-on-ubuntu-16-04

```
sudo vim /etc/nginx/sites-available/project1.com
```

Paste the following code changing everything inside {} for your own values:
```
server {
    listen 80;
    server_name server_domain_or_IP;

    location = /favicon.ico { access_log off; log_not_found off; }
    location /static/ {
        root {/home/user/project1};
    }

    location / {
        include proxy_params;
        proxy_pass http://unix:{/home/user/project1/project1}.sock;
    }
}
```

Now, you need to enable server block and restart Nginx:
```
sudo ln -s /etc/nginx/sites-available/project1.com /etc/nginx/sites-enabled
sudo systemctl restart nginx
```
