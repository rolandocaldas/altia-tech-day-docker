# Taller Docker: Desarrollando entre conteneores

## Instalación docker CE

- Ubuntu: https://docs.docker.com/install/linux/docker-ce/ubuntu/
- Windows: https://store.docker.com/editions/community/docker-ce-desktop-windows
- Mac: https://store.docker.com/editions/community/docker-ce-desktop-mac
- Legacy:
  - Windows: https://docs.docker.com/toolbox/toolbox_install_windows/
  - Mac: https://docs.docker.com/toolbox/toolbox_install_mac/

# Timeline

> \> tty 1
```bash
$ sudo groupadd docker  	
$ sudo usermod -aG docker $USER
```

> \> tty 1
```bash
$ cd $HOME
$ mkdir dockerWorkshop && cd dockerWorkshop
$ docker run hello-world
$ docker ps
$ docker ps -a
$ docker run ubuntu
$ docker ps
$ docker ps -a
$ docker images
$ docker run -it ubuntu
root@f709b1535b47:/#
```

> \> tty 2
```bash
$ cd $HOME/dockerWorkshop
$ docker run -it node
```

> \> tty 3
```bash
$ cd $HOME/dockerWorkshop
$ docker ps
```

> \> tty 1
```bash
root@f709b1535b47:/# top
```

> \> tty 3
```bash
$ ps afux
```

> \> tty 1
```bash
root@f709b1535b47:/# Ctrl+C
root@f709b1535b47:/# exit
$ docker run --name=php-fpm php:7.2-fpm
```
 
> \> tty 2
```bash
$ Ctrl+C
$ docker exec php-fpm php -v
$ docker run -p 8080:80 -it --name=php-apache php:7.2-apache
```
 
> \> tty 3
```bash
$ docker exec -it php-apache bash
root@be8415f43808:/var/www/html# apt update && apt install vim
root@be8415f43808:/var/www/html# vim index.php
```

```php
<?php 
phpinfo();
```

```bash
:wq
$ exit
```

> \> tty 1
```bash
$ Ctrl+C
$ docker run -it --name=php-fpm php:7.2-fpm
$ docker start php-fpm
```

> \> tty 1
```bash
$ docker ps
```

> \> tty 1
```bash
$ mkdir docker && cd docker
$ mkdir php-base && cd php-base
$ touch Dockerfile
$ vim Dockerfile
```

> dockerWorkshop/docker/php-base/Dockerfile

```dockerfile
FROM php:7.2-fpm

WORKDIR "/application"
RUN apt-get update \
    && apt-get install -y libicu-dev libpng-dev libjpeg-dev libpq-dev  \
    && apt-get clean; rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* /usr/share/doc/* \
    && docker-php-ext-configure gd --with-png-dir=/usr --with-jpeg-dir=/usr \
    && docker-php-ext-install intl gd mbstring
RUN apt-get update && apt-get install -y mysql-client \
    && apt-get clean; rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* /usr/share/doc/* \
    && docker-php-ext-install mysqli pdo pdo_mysql
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer
```

> \> tty 1
```bash
$ docker build -t "php-fpm-base" .
$ docker run --name="php-fpm-base" php-fpm-base
```

> \> tty 3
```bash
$ docker exec php-fpm-base php -i
$ docker exec php-fpm-base composer -V
```

```bash
$ cd docker && mkdir php-fpm-dev && cd php-fpm-dev
$ touch Dockerfile
$ vim Dockerfile
```

> dockerWorkshop/docker/php-fpm-dev/Dockerfile
```dockerfile
FROM php-fpm-base

RUN pecl install xdebug && docker-php-ext-enable xdebug

RUN apt-get update \
    && apt-get -y install unzip zlib1g-dev \
    && apt-get clean; rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* /usr/share/doc/* \
    && docker-php-ext-install zip
```

> \> tty 3
```bash
$ docker build -t "php-fpm-dev" .
$ docker run --name="php-fpm-dev" php-fpm-dev
```

> \> tty 1
```bash
$ Ctrl+C
$ docker exec php-fpm-dev php -v
```

> \> tty 3
```bash
$ Ctrl+C
```

> \> tty 2
```bash
$ Ctrl+C
```

> \> tty 1
```bash
$ Ctrl+C
$ cd ..
$ git init
$ git config --global user.email "rolando.caldas@gmail.com"
$ git config --global user.name "Rolando Caldas"
$ git add .
$ git commit -m "My php docker containers configuration for development"
$ git remote add origin https://github.com/rolando-caldas/cebem-docker-php.git
$ git push -u origin master
```

> Google Chrome

En https://hub.docker.com nos creamos una cuenta y la enlazamos con Github. Para enlazarla con Github,
una vez creada ni ideay estando conectados accedemos a https://hub.docker.com/account/authorized-services/
y desde ahí vamos a "Link Github"

Con la cuenta enlazada creamos un automated build:

![docker hub](https://raw.githubusercontent.com/phpvigo/docker-workshop/master/images/01-create-automated-build.png)

Seleccionamos "Create Auto-build Github"

![Create Auto-build Github](https://raw.githubusercontent.com/phpvigo/docker-workshop/master/images/02-create-automated-build.png)

Buscamos el repositorio de nuestro github docker-workshop 

![Github](https://raw.githubusercontent.com/phpvigo/docker-workshop/master/images/03-create-automated-build.png)

Podemos seleccionar visibilidad privada (de forma gratuita tenemos uno disponible) o público, en nuestro caso este último

![Select public](https://raw.githubusercontent.com/phpvigo/docker-workshop/master/images/04-create-automated-build.png)

Una vez creado, vamos a Build Settings y lo dejamos como en a siguiente imagen:

![Config builds](https://raw.githubusercontent.com/phpvigo/docker-workshop/master/images/05-create-automated-build.png)

Clicamos en "Trigger" por orden, primero php-base, luego php-xdebug, php-dev y php-dev-mysql como último.

Los build entrarán en cola en el hub y una vez finalicen en tags tendremos las imágenes disponibles.

![Tags available](https://raw.githubusercontent.com/phpvigo/docker-workshop/master/images/06-create-automated-build.png)

> \> tty 1
```bash
$ vim php-fpm-dev/Dockerfile
```

> workshopDocker-phpVigo/docker/php-xdebug/Dockerfile
```dockerfile
FROM rolandocaldas/cebem-php:php-base
[...]
```

> \> tty 1
```bash
$ git add .
$ git commit -m "Changed Dockerfiles to set the FROM new value"
$ git push origin master
```

> \> tty 1
```bash
$ docker run rolandocaldas/cebem-php:php-fpm-dev
```

> \> tty 1
```bash
$ Ctrl+C
$ cd ..
$ git clone https://github.com/rolando-caldas/workshop-gitlab-backend.git
$ git clone https://github.com/rolando-caldas/workshop-gitlab-frontendgit
```
