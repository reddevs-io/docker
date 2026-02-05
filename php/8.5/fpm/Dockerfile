# Set the PHP version as a build argument, defaulting to 8.3
ARG PHP_VERSION=8.5

# Use the official PHP image with FPM and Alpine Linux as the base
FROM php:${PHP_VERSION}-fpm-alpine

# Set user and group IDs as build arguments
ARG UID=1000
ARG GID=1000

# Install dependencies and PHP extensions
RUN set -eux; \
	\
	# Install build dependencies
	apk add --no-cache --virtual .build-deps \
    $PHPIZE_DEPS \
    linux-headers \
    coreutils \
    freetype-dev \
    libjpeg-turbo-dev \
    libpng-dev \
    libzip-dev \
    libwebp-dev \
    libpq-dev \
    jq \
	; \
	\
	# Configure GD extension with support for freetype, jpeg, and webp
	docker-php-ext-configure gd \
    --with-freetype \
    --with-jpeg=/usr/include \
    --with-webp \
  ; \
	\
	# Install PHP extensions
	docker-php-ext-install -j "$(nproc)" \
		gd \
		opcache \
		zip \
	; \
	# Install PECL extensions
	pecl install \
    apcu \
    uploadprogress \
    opcache \
  ; \
  pecl clear-cache; \
  # Enable installed extensions
  docker-php-ext-enable \
    apcu \
    uploadprogress \
    opcache \
  ; \
	\
	# Install runtime dependencies
	runDeps="$( \
		scanelf --needed --nobanner --format '%n#p' --recursive /usr/local \
			| tr ',' '\n' \
			| sort -u \
			| awk 'system("[ -e /usr/local/lib/" $1 " ]") == 0 { next } { print "so:" $1 }' \
	)"; \
	apk add --virtual .drupal-phpexts-rundeps $runDeps; \
	# Remove build dependencies
	apk del .build-deps

# Install additional tools
RUN apk add --no-cache \
    git \
    patch

# Change PHP-FPM user
RUN addgroup -g ${GID} -S web \
    && adduser -u ${UID} -S web -G www-data \
    && sed -i s/'user = www-data'/'user = web'/g /usr/local/etc/php-fpm.d/www.conf \
    && sed -i s/'group = www-data'/'group = web'/g /usr/local/etc/php-fpm.d/www.conf

# Copy Composer from the official Composer image
COPY --from=composer:2 /usr/bin/composer /usr/local/bin/composer

# Switch to the non-root user
USER web
