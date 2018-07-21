## WebVirtCloud - Bandic007 (B7) updated and fixed edition - Version 1.1.1

## Credits
This version of WebVirtCloud was forked from [@catborise](https://github.com/catborise/webvirtcloud) repo, which is a forked and modified (with upgrades of UI and functionality) versio of [@retspen](https://github.com/retspen/webvirtcloud) repo.
Thanks to both of them for the amazing job they did!

## Changelog by version
The B7 version in this repo has some fixes applied on problems from the upgraded [@catborise](https://github.com/catborise/webvirtcloud) repo. They were:

Version 1.0:
* fixed not working login page
* fixed not working (loading) "Instances" page - missconfigured variable in the code
* fixed python required packages version
* added missed python required packages
* fixed problem with deployment, because of not fully configured settings.py.template file - missed opperators
* updated manual and update procedures
* all fixes included in this release for instant working deployment
* added B7 mod version, displayed on login, logout pages and top bar.
* added included custom python scrypt for easier Django "SECRET_KEY" creation

Version 1.1:
* included all previous fixes of course
* added visual changes in instances web page
* fixed some requirements versions
* removed old outdated requirements
* fixed VM conversion into template problem, caused by html code problem
* fixed network details not opening in compute section because of changes needed in python code
* changed the order of "used size / total size" display in single storage html page
  * Version 1.1 - Patch 1 (1.1.1)
    * fixed variable typo in python code
    * fixed issues if there is an inactive storage, which was causing a problem
    * volume file extension converted to choosabe via settings
    * volume file extensions reorganized and added "qcow2" image format as default one
    * changed hosts behavior when getting list of active/inactive sotrages to avoid break whena storage is inactive
    * removed data sort possibility

This version will always be updated with the latest changes from both repos and I will try to apply fixes to all bugs I find, in order to make it always running from the first time you deploy it!
Also, I will escalate "Pull requests" to both those guys in order to offer them my fixes in their repos!

## Features

* User can add SSH public key to root in Instance (Tested only Ubuntu)
* User can change root password in Instance (Tested only Ubuntu)
* Supports cloud-init datasource interface

### Warning!!!

How to update <code>gstfsd</code> daemon on hypervisor:

```bash
wget -O - https://raw.githubusercontent.com/Bandic007/WebVirtCloud-B7/master/modules/gstfsd | sudo tee -a /usr/local/bin/gstfsd
sudo service supervisor restart
```

### Description

WebVirtCloud is a virtualization web interface for admins and users. It can delegate Virtual Machine's to users. A noVNC viewer presents a full graphical console to the guest domain.  KVM is currently the only hypervisor supported.

### Install WebVirtCloud panel (Ubuntu)

```bash
sudo apt-get -y install git python-virtualenv python-dev python-lxml libvirt-dev zlib1g-dev nginx supervisor libsasl2-modules gcc pkg-config python-libguestfs
git clone https://github.com/Bandic007/WebVirtCloud-B7.git
cd WebVirtCloud-B7
cp webvirtcloud/settings.py.template webvirtcloud/settings.py
# now put secret key to webvirtcloud/settings.py
sudo cp conf/supervisor/webvirtcloud.conf /etc/supervisor/conf.d
sudo cp conf/nginx/webvirtcloud.conf /etc/nginx/conf.d
cd ..
sudo mv WebVirtCloud-B7/ /srv/webvirtcloud
sudo chown -R www-data:www-data /srv/webvirtcloud
cd /srv/webvirtcloud
virtualenv venv
source venv/bin/activate
pip install -r conf/requirements.txt
python manage.py migrate
sudo chown -R www-data:www-data /srv/webvirtcloud
sudo rm /etc/nginx/sites-enabled/default
```
### Generate secret key
You should generate SECRET_KEY after cloning repo and BEFORE starting the application for the first time.
Execute the python script, that is located in the main directory of the cloned repo. Copy the output that it prints.

```bash
cd /srv/webvirtcloud
python generate_secret_key.py
```

### Restart services for running WebVirtCloud:

```bash
sudo service nginx restart
sudo service supervisor restart
```

### Setup libvirt and KVM on server

```bash
wget -O - https://raw.githubusercontent.com/retspen/webvirtcloud/master/dev/libvirt-bootstrap.sh | sudo sh
```

### Install WebVirtCloud panel (CentOS)

```bash
sudo yum -y install python-virtualenv python-devel libvirt-devel glibc gcc nginx supervisor libxml2 libxml2-devel git python-libguestfs
```

#### Creating directories and cloning repo

```bash
cd /tmp
sudo git clone https://github.com/Bandic007/WebVirtCloud-B7 && cd WebVirtCloud-B7
cd WebVirtCloud-B7
cp webvirtcloud/settings.py.template webvirtcloud/settings.py
# now put secret key to webvirtcloud/settings.py
cd ..
sudo mkdir /srv
sudo mv WebVirtCloud-B7/ /srv/webvirtcloud
cd /srv/webvirtcloud
```

#### Start installation webvirtcloud
```
sudo virtualenv venv
sudo source venv/bin/activate
sudo venv/bin/pip install -r conf/requirements.txt
sudo cp conf/nginx/webvirtcloud.conf /etc/nginx/conf.d/
sudo venv/bin/python manage.py migrate
```

#### Configure the supervisor for CentOS
Add the following after the [include] line (after **files = ... ** actually):
```bash
sudo vim /etc/supervisord.conf

[program:webvirtcloud]
command=/srv/webvirtcloud/venv/bin/gunicorn webvirtcloud.wsgi:application -c /srv/webvirtcloud/gunicorn.conf.py
directory=/srv/webvirtcloud
user=nginx
autostart=true
autorestart=true
redirect_stderr=true

[program:novncd]
command=/srv/webvirtcloud/venv/bin/python /srv/webvirtcloud/console/novncd
directory=/srv/webvirtcloud
user=nginx
autostart=true
autorestart=true
redirect_stderr=true
```
### Generate secret key
You should generate SECRET_KEY after cloning repo and BEFORE starting the application for the first time.
Execute the python script, that is located in the main directory of the cloned repo. Copy the output that it$

```bash
cd /srv/webvirtcloud
python generate_secret_key.py
```

#### Edit the nginx.conf file
You will need to edit the main nginx.conf file as the one that comes from the rpm's will not work. Comment the following lines:

```
#    server {
#        listen       80 default_server;
#        listen       [::]:80 default_server;
#        server_name  _;
#        root         /usr/share/nginx/html;
#
#        # Load configuration files for the default server block.
#        include /etc/nginx/default.d/*.conf;
#
#        location / {
#        }
#
#        error_page 404 /404.html;
#            location = /40x.html {
#        }
#
#        error_page 500 502 503 504 /50x.html;
#            location = /50x.html {
#        }
#    }
}
```

Also make sure file in **/etc/nginx/conf.d/webvirtcloud.conf** has the proper paths:
```
upstream gunicorn_server {
    #server unix:/srv/webvirtcloud/venv/wvcloud.socket fail_timeout=0;
    server 127.0.0.1:8000 fail_timeout=0;
}
server {
    listen 80;

    server_name servername.domain.com;
    access_log /var/log/nginx/webvirtcloud-access_log; 

    location /static/ {
        root /srv/webvirtcloud;
        expires max;
    }

    location / {
        proxy_pass http://gunicorn_server;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-for $proxy_add_x_forwarded_for;
        proxy_set_header Host $host:$server_port;
        proxy_set_header X-Forwarded-Proto $remote_addr;
        proxy_connect_timeout 600;
        proxy_read_timeout 600;
        proxy_send_timeout 600;
        client_max_body_size 1024M;
    }
}
```

Change permissions so nginx can read the webvirtcloud folder:

```bash
sudo chown -R nginx:nginx /srv/webvirtcloud
```

Change permission for selinux:

```bash
sudo semanage fcontext -a -t httpd_sys_content_t "/srv/webvirtcloud(/.*)"
```

Add required user to the kvm group:
```bash
sudo usermod -G kvm -a webvirtmgr
```

Let's restart nginx and the supervisord services:
```bash
sudo systemctl restart nginx && systemctl restart supervisord
```

And finally, check everything is running:
```bash
sudo supervisorctl status

novncd                           RUNNING    pid 24186, uptime 2:59:14
webvirtcloud                     RUNNING    pid 24185, uptime 2:59:14

```

#### Apache mod_wsgi configuration
```
WSGIDaemonProcess webvirtcloud threads=2 maximum-requests=1000 display-name=webvirtcloud
WSGIScriptAlias / /srv/webvirtcloud/webvirtcloud/wsgi_custom.py
```

#### Install final required packages for libvirtd and others on Host Server
```bash
wget -O - https://raw.githubusercontent.com/Bandic007/WebVirtCloud-B7/master/modules/libvirt-bootstrap | sudo sh
```

Done!!

Go to http://serverip and you should see the login screen.

### Default credentials
<pre>
login: admin
password: admin
</pre>

### Cloud-init
Currently supports only root ssh authorized keys and hostname. Example configuration of the cloud-init client follows.
```
datasource:
  OpenStack:
      metadata_urls: [ "http://webvirtcloud.domain.com/datasource" ]
```

### How To Update
```bash
cd /srv/webvirtcloud
git pull
virtualenv venv
source venv/bin/activate
pip install -U -r conf/requirements.txt
sudo venv/bin/pip install -r conf/requirements.txt #this command is for CentOS only!!!
python manage.py migrate
sudo venv/bin/python manage.py migrate #this command is for CentOS only!!!
sudo service supervisor restart
systemctl restart supervisord #this command is for CentOS only!!!
deactivate
```

### License

WebVirtCloud is licensed under the [Apache Licence, Version 2.0](http://www.apache.org/licenses/LICENSE-2.0.html).

