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

```shell
docker buildx build \
  --platform linux/amd64,linux/arm64 \
  --builder cloud-diolektor-default-builder \
  --tag diolektor/nginx-unit-php:unit1.33.0-php8.3.14-alpine3.20 \
  --target base \
  --push \
  .
```

```shell
docker buildx build \
  --platform linux/amd64,linux/arm64 \
  --builder cloud-diolektor-default-builder \
  --tag diolektor/nginx-unit-php:unit1.33.0-php8.4.1-alpine3.20 \
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

```shell
docker buildx build \
  --platform linux/amd64,linux/arm64 \
  --builder cloud-diolektor-default-builder \
  --tag diolektor/app-php-server:unit1.33.0-php8.3.14-alpine3.20 \
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

```shell
docker buildx build \
  --platform linux/amd64,linux/arm64 \
  --builder cloud-diolektor-default-builder \
  --tag diolektor/app-php-server:unit1.33.0-php8.3.14-alpine3.20-dev \
  --target dev \
  --push \
  .
```

```shell
docker buildx build \
  --platform linux/amd64,linux/arm64 \
  --builder cloud-diolektor-default-builder \
  --tag diolektor/app-php-server:unit1.33.0-php8.4.1-alpine3.20 \
  --target base \
  --push \
  .
```

```shell
docker buildx build \
  --platform linux/amd64,linux/arm64 \
  --builder cloud-diolektor-default-builder \
  --tag diolektor/app-php-server:unit1.33.0-php8.4.1-alpine3.20-dev \
  --target dev \
  --push \
  .
```