### php

```shell
docker buildx build \
  --platform linux/amd64,linux/arm64 \
  --builder cloud-diolektor-default-builder \
  --tag diolektor/php-cli-embed:php8.3.10-alpine3.20 \
  --push \
  .
```

### nginx-unit
```shell
docker buildx build \
  --platform linux/amd64,linux/arm64 \
  --builder cloud-diolektor-default-builder \
  --tag diolektor/nginx-unit-php:unit1.32.1-php8.3.10-alpine3.20 \
  --target base \
  --push \
  .
```

### app-server

- without xdebug
```shell
docker buildx build \
  --platform linux/amd64,linux/arm64 \
  --builder cloud-diolektor-default-builder \
  --tag diolektor/app-php-server:unit1.32.1-php8.3.10-alpine3.20 \
  --target base \
  --push \
  .
```


- with xdebug
```shell
docker buildx build \
  --platform linux/amd64,linux/arm64 \
  --builder cloud-diolektor-default-builder \
  --tag diolektor/app-php-server:unit1.32.1-php8.3.10-alpine3.20-dev \
  --target dev \
  --push \
  .
```
