# Set the PHP version as a build argument, defaulting to 8.3
ARG PHP_VERSION=8.4
# Use NGINX Unit with the specified PHP version as the base image
FROM unit:php${PHP_VERSION}

# Set user and group IDs as build arguments
ARG UID=1000
ARG GID=1000

# Update base image packages
RUN apt-get update -y \
    && apt-get upgrade -y

# Install PHP extensions and dependencies
RUN set -eux; \
	\
	# Save list of currently installed packages
	savedAptMark="$(apt-mark showmanual)"; \
	\
	# Update package lists and install build dependencies
	apt-get update; \
	apt-get install -y --no-install-recommends \
		libfreetype6-dev \
		libjpeg-dev \
		libpng-dev \
		libpq-dev \
		libwebp-dev \
		libzip-dev \
	; \
	\
	# Configure GD extension
	docker-php-ext-configure gd \
		--with-freetype \
		--with-jpeg=/usr \
		--with-webp \
	; \
	\
	# Install PHP extensions
	docker-php-ext-install -j "$(nproc)" \
		gd \
		opcache \
		pdo_mysql \
		pdo_pgsql \
		zip \
	; \
  # Install PECL extensions
  pecl install \
    apcu \
    uploadprogress \
    opcache \
    redis \
  ; \
  pecl clear-cache; \
  # Enable installed extensions
  docker-php-ext-enable \
    apcu \
    uploadprogress \
    opcache \
    redis \
  ; \
	# Clean up
	apt-mark auto '.*' > /dev/null; \
	apt-mark manual $savedAptMark; \
	ldd "$(php -r 'echo ini_get("extension_dir");')"/*.so \
		| awk '/=>/ { so = $(NF-1); if (index(so, "/usr/local/") == 1) { next }; gsub("^/(usr/)?", "", so); print so }' \
		| sort -u \
		| xargs -r dpkg-query -S \
		| cut -d: -f1 \
		| sort -u \
		| xargs -rt apt-mark manual; \
	\
	apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false; \
	rm -rf /var/lib/apt/lists/*

# Install additional tools
RUN apt-get update; \
    apt-get install -y --no-install-recommends \
    git \
    unzip \
    patch;

# Change the UID and GID of the unit user
RUN usermod -u ${UID} unit && groupmod -g ${GID} unit

# Install Composer
COPY --from=composer:2 --chown=unit:unit /usr/bin/composer /usr/local/bin/composer
# Allow Composer to run as superuser
ENV COMPOSER_ALLOW_SUPERUSER=1
