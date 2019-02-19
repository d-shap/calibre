# Book management web server
Docker image for book management web server.

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

# Donation
If you find my code useful, you can [bye me a coffee](https://www.paypal.me/dshapovalov)
