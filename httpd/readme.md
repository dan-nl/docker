# httpd
___note:___ this repo is in an alpha state and may be removed

Dockerfile was developed based on this Dockerfile:
* [httpd:2.4-alpine][dockerfile-url]

[dockerfile-url]: https://github.com/docker-library/httpd/blob/109e6332af1ac4176aefd7aad1c28e42f9e10644/2.4/alpine/Dockerfile


```sh
docker container run \
-itd \
-p 80:80 \
-v $PWD/public:/usr/local/apache2/htdocs/ \
-v $PWD/conf:/usr/local/apache2/conf/ \
--rm \
--name httpd \
httpd
```

```sh
docker container exec -it httpd sh
```

## apache directory structure
* [2.4.x config layout][config-layout-url]
* [filesystem hierarchy standard][filesystem-standard-url]


[config-layout-url]: http://svn.apache.org/viewvc/httpd/httpd/branches/2.4.x/config.layout?view=markup
[filesystem-standard-url]: https://en.wikipedia.org/wiki/Filesystem_Hierarchy_Standard
