# Notebook
Understanding the world with Jupyter/IPython notebooks

## Setup

- Create DNS mapping from notebook.volgy.com/nb.volgy.com to the server

- Install anaconda3 under /opt/anaconda3

- Checkout the repo

```
$ git clone git@github.com:volgy/notebook.git /www/notebook
```

- Setup reverse proxy

```
sudo apt-get install apache2-utils nginx
htpasswd -c /www/notebook/.htpasswd <http-username>
sudo vi /etc/nginx/sites-available/notebook
```

Contents of `/etc/nginx/sites-available/notebook`:

```
upstream notebook {
        server localhost:8888;
}

server{
        server_name notebook.volgy.com nb.volgy.com;
        listen 80;

        auth_basic "Restricted";
        auth_basic_user_file /www/notebook/.htpasswd;

        location / {
                proxy_pass            http://notebook;
                proxy_set_header      Host $host;
        }

        location ~ /api/kernels/ {
                proxy_pass            http://notebook;
                proxy_set_header      Host $host;
                # websocket support
                proxy_http_version    1.1;
                proxy_set_header      Upgrade "websocket";
                proxy_set_header      Connection "Upgrade";
                proxy_read_timeout    86400;
        }

        location ~ /terminals/ {
                proxy_pass            http://notebook;
                proxy_set_header      Host $host;
                # websocket support
                proxy_http_version    1.1;
                proxy_set_header      Upgrade "websocket";
                proxy_set_header      Connection "Upgrade";
                proxy_read_timeout    86400;
        }
}
```

```
$ sudo ln -s /etc/nginx/sites-available/notebook /etc/nginx/sites-enabled/
$ sudo /etc/init.d/nginx reload
```
### Ubutnu 14.04

- Create a new upstart config file for Jupyter

```
$ sudo vi /etc/init/notebook.conf
```

Contents of `/etc/init/notebook.conf`:

```
description "Jupyter notebook service"
author      "Peter Volgyesi"

start on filesystem or runlevel [2345]
stop on shutdown
respawn

script
    echo $$ > /var/run/jupyter.pid
    exec su -s /bin/sh -c 'exec "$0" "$@"' volgy -- /opt/anaconda3/bin/jupyter  notebook --no-browser --notebook-dir='/www/notebook'
end script

pre-stop script
    rm /var/run/jupyter.pid
end script
```

- Start the Jupyter service

```
$ sudo start notebook
```


### Ubutnu 16.04

- Create a new systemd service unit for Jupyter

```
$ sudo vi /usr/lib/systemd/system/notebook.service
```

Contents of `/lib/systemd/system/notebook.service`:

```
[Unit]
Description=Jupyter notebook service

[Service]
Type=simple
PIDFile=/var/run/notebook.pid
ExecStart=/opt/anaconda3/bin/jupyter notebook --no-browser
User=volgy
Group=volgy
WorkingDirectory=/www/notebook

[Install]
WantedBy=multi-user.target
```

- Start the Jupyter service

```
$ sudo systemctl enable notebook
$ sudo systemctl start notebook
```
