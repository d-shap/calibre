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

Create user account:
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

### Database management
```
sudo clutil calibredb command [options] [arguments]
```

**calibredb** command is passed to the running docker container with **clutil** utility.

If **calibredb** command requires library path to be specified, then library name (as specified in the **LIBRARIES** environment variable) should be used.
The **clutil** utility translates library name to the library path.
For example, to list the specified book fields in the library **&lt;library_name&gt;**, next command can be used:
```
sudo clutil calibredb list --library-path <library_name> -f id,authors,author_sort,title,series,series_index,publisher,tags
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

## HOW TO
### How to use multiple libraries
Stop calibre service:
```
sudo service calibre stop
```

Specify libraries names (with colon as a delimiter) in **/usr/sbin/calibre** file:
```
docker run ... -e LIBRARIES="library-1:library-2:library-3" ...
```

Start calibre service:
```
sudo service calibre start
```

### How to provide anonymous access to the libraries
Stop calibre service:
```
sudo service calibre stop
```

Specify container environment variable **WITHAUTH** in **/usr/sbin/calibre** file:
```
docker run ... -e WITHAUTH="false" ...
```

Start calibre service:
```
sudo service calibre start
```

Everyone can access the libraries, but no one can make changes.
To upload new books, or to change the book metadata, the container environment variable **WITHAUTH** should be **true**.
After the changes are made, the container environment variable **WITHAUTH** can be changed to **false** again.

### How to clear book metadata
```
sudo clutil clearmetadata <library> <column_name>
```

This command will clean the specified **&lt;column_name&gt;** for all books in the specified **&lt;library&gt;**.

The valid column names are:
* identifiers
* languages
* pubdate
* publisher
* timestamp
* rating

### How to remove excessive permissions
Calibre gives 777 permit to all folders in the libraries.
Calibre also gives execute permission to the log files.
To remove excessive permissions, just restart calibre service:
```
sudo service calibre restart
```

### How to create cron job for backups
```
sudo crontab -l | { cat; echo "minute hour * * * /usr/bin/clutil backup <filename>"; echo ""; } | sudo crontab -
```

# Donation
If you find my code useful, you can [bye me a coffee](https://www.paypal.me/dshapovalov)
