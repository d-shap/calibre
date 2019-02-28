# Book management web server
Docker image for book management web server.

Container runs as non-root user.
This user owns calibre process and owns calibre libraries.

To run container next volumes should be mapped:
* folder for calibre libraries
* log folder
* backup folder

## Installation
### Installation from docker image
Pull docker image.

Create user and group to own calibre files and to run docker container:
```
sudo groupadd -g 965 calibre
```
```
useradd -u 965 -g 965 -M calibre
```

Proceed to configuration.

### Installation from source
Pull project sources from version control system.

Create user and group to own calibre files and to run docker container:
```
sudo useradd -r calibre
```

Make **build** executable:
```
sudo chmod u+x ./build
```

Execute **build**:
```
sudo ./build calibre
```

Proceed to configuration.

### Configuration
Create folders for calibre files:
```
sudo mkdir /calibre
```
```
sudo mkdir /calibre/data
```

Create folder for logs:
```
sudo mkdir /var/log/calibre
```

Create folder for backups:
```
sudo mkdir /var/backups/calibre
```

Grant permit to all folders:
```
sudo chown -R calibre:calibre /calibre
```
```
sudo chown calibre:calibre /var/log/calibre
```
```
sudo chown calibre:calibre /var/backups/calibre
```

Copy **etc/init.d/calibre** to **/etc/init.d** folder:
```
sudo cp ./etc/init.d/calibre /etc/init.d
```

Copy **usr/sbin/calibre** to **/usr/sbin** folder:
```
sudo cp ./usr/sbin/calibre /usr/sbin
```

Copy **usr/bin/clutil** to **/usr/bin** folder:
```
sudo cp ./usr/bin/clutil /usr/bin
```

Make all files executable:
```
sudo chmod a+x /etc/init.d/calibre
```
```
sudo chmod a+x /usr/sbin/calibre
```
```
sudo chmod a+x /usr/bin/clutil
```

Register service:
```
sudo update-rc.d calibre defaults
```

Specify library name in **/usr/sbin/calibre** file:
```
docker run ... -e LIBRARIES="<library_name>" ...
```

Start calibre service:
```
sudo service calibre start
```

Create user acount:
```
sudo clutil manageusers
```

Restart calibre service:
```
sudo service calibre restart
```

## Management
### Service management
```
sudo service calibre (start|stop|status|restart)
```

### User management
```
sudo clutil manageusers
```

### Create backup
```
sudo clutil backup <filename>
```

Backup file **/var/backups/calibre/&lt;filename&gt;.tar.gz** will be created.

### Restore backup
```
sudo clutil restore <filename>
```

### Command line (bash)
```
sudo clutil bash
```

## Apache mod_proxy configuration
Book management web server can be located with another web applications.
For example, mercurial, bugzilla, wiki etc can be run as docker containers on the same host.
In this case apache server can be used to redirect requests to different docker containers.

Enable apache mod_proxy:
```
sudo a2enmod proxy proxy_ajp proxy_html proxy_http rewrite deflate headers proxy_balancer proxy_connect
```

Configure proxy:
```
<VirtualHost *:80>

...

ProxyPreserveHost On
<Proxy *>
    Order allow,deny
    Allow from all
</Proxy>

...

</VirtualHost>
```

Copy **./etc/apache2/sites-available/calibre.conf** to **/etc/apache2/sites-available** folder:
```
sudo cp ./etc/apache2/sites-available/calibre.conf /etc/apache2/sites-available
```

Enable apache calibre site:
```
sudo a2ensite calibre
```

Restart apache service:
```
sudo service apache2 restart
```

# Donation
If you find my code useful, you can [bye me a coffee](https://www.paypal.me/dshapovalov)
