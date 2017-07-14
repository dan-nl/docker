# httpd-php
___note:___ this repo is in an alpha state and may be removed

Dockerfile was developed based on these Dockerfiles:
* [httpd:2.4-alpine][dockerfile-httpd-url]
* [php:7.1-alpine][dockerfile-php-url]

```sh
docker container run \
-itd \
-p 80:80 \
-v $PWD/public:/usr/local/apache2/htdocs/ \
-v $PWD/conf:/usr/local/apache2/conf/ \
--rm \
--name httpd-php \
httpd-php
```

```sh
docker container exec -it httpd-php sh
```

## apache directory structure
* [2.4.x config layout][config-layout-url]
* [filesystem hierarchy standard][filesystem-standard-url]


[dockerfile-httpd-url]: https://github.com/docker-library/httpd/blob/109e6332af1ac4176aefd7aad1c28e42f9e10644/2.4/alpine/Dockerfile
[dockerfile-php-url]: https://github.com/docker-library/php/blob/ddc7084c8a78ea12f0cfdceff7d03c5a530b787e/7.1/alpine/Dockerfile
[config-layout-url]: http://svn.apache.org/viewvc/httpd/httpd/branches/2.4.x/config.layout?view=markup
[filesystem-standard-url]: https://en.wikipedia.org/wiki/Filesystem_Hierarchy_Standard
