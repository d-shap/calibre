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
1. Pull docker image.

2. Create user and group to own calibre files and to run docker container:
    ```
    sudo groupadd -g 965 calibre
    ```
    ```
    useradd -u 965 -g 965 -M calibre
    ```

3. Proceed to configuration.

### Installation from source
1. Pull project sources from version control system.

2. Create user and group to own calibre files and to run docker container:
    ```
    sudo useradd -r calibre
    ```

3. Make **build** executable:
    ```
    sudo chmod u+x ./build
    ```

4. Execute **build**:
    ```
    sudo ./build calibre
    ```

5. Proceed to configuration.

### Configuration
1. Create folders for calibre files:
    ```
    sudo mkdir /calibre
    ```
    ```
    sudo mkdir /calibre/data
    ```

2. Create folder for logs:
    ```
    sudo mkdir /var/log/calibre
    ```

3. Create folder for backups:
    ```
    sudo mkdir /var/backups/calibre
    ```

4. Grant permit to all folders:
    ```
    sudo chown -R calibre:calibre /calibre
    ```
    ```
    sudo chown calibre:calibre /var/log/calibre
    ```
    ```
    sudo chown calibre:calibre /var/backups/calibre
    ```

5. Copy **etc/init.d/calibre** to **/etc/init.d** folder:
    ```
    sudo cp ./etc/init.d/calibre /etc/init.d
    ```

6. Copy **usr/sbin/calibre** to **/usr/sbin** folder:
    ```
    sudo cp ./usr/sbin/calibre /usr/sbin
    ```

7. Copy **usr/bin/clutil** to **/usr/bin** folder:
    ```
    sudo cp ./usr/bin/clutil /usr/bin
    ```

8. Make all files executable:
    ```
    sudo chmod a+x /etc/init.d/calibre
    ```
    ```
    sudo chmod a+x /usr/sbin/calibre
    ```
    ```
    sudo chmod a+x /usr/bin/clutil
    ```

9. Register service:
    ```
    sudo update-rc.d calibre defaults
    ```

10. Specify library name in **/usr/sbin/calibre** file:
    ```
    docker run ... -e LIBRARIES="<library_name>" ...
    ```

11. Start calibre service:
    ```
    sudo service calibre start
    ```

12. Create user account:
    ```
    sudo clutil manageusers
    ```

13. Restart calibre service:
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

1. Enable apache mod_proxy:
    ```
    sudo a2enmod proxy proxy_ajp proxy_html proxy_http rewrite deflate headers proxy_balancer proxy_connect
    ```

2. Configure proxy:
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

3. Copy **./etc/apache2/sites-available/calibre.conf** to **/etc/apache2/sites-available** folder:
    ```
    sudo cp ./etc/apache2/sites-available/calibre.conf /etc/apache2/sites-available
    ```

4. Enable apache calibre site:
    ```
    sudo a2ensite calibre
    ```

5. Restart apache service:
    ```
    sudo service apache2 restart
    ```

## HOW TO
### How to use multiple libraries
1. Stop calibre service:
    ```
    sudo service calibre stop
    ```

2. Specify libraries names (with colon as a delimiter) in **/usr/sbin/calibre** file:
    ```
    docker run ... -e LIBRARIES="library-1:library-2:library-3" ...
    ```

3. Start calibre service:
    ```
    sudo service calibre start
    ```

### How to provide anonymous access to the libraries
1. Stop calibre service:
    ```
    sudo service calibre stop
    ```

2. Specify container environment variable **WITHAUTH** in **/usr/sbin/calibre** file:
    ```
    docker run ... -e WITHAUTH="false" ...
    ```

3. Start calibre service:
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

### How to remove books
To remove the book with id **&lt;id&gt;** from the library **&lt;library_name&gt;**, the next command can be used:
```
sudo clutil calibredb --library-path <library_name> remove <id>
```

Multiple book ids can be specified (with comma separator, or with range):
```
sudo clutil calibredb --library-path <library_name> remove <id1>,<id2>,<id3>
```
```
sudo clutil calibredb --library-path <library_name> remove <id1>-<id3>
```

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
