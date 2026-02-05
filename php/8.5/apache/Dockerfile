# Set the PHP version as a build argument, defaulting to 8.3
ARG PHP_VERSION=8.5

# Start from the official PHP 8.3 Apache image
FROM php:${PHP_VERSION}-apache

# Set user and group IDs as build arguments with defaults
ARG UID=1000
ARG GID=1000

# --- 1. System & PHP Configuration ---
# Combine installations into a single RUN command to reduce image layers

RUN set -eux; \
	\
	apt-get update; \
	apt-get install -y --no-install-recommends \
		# PHP extension build dependencies
		libfreetype6-dev \
		libjpeg-dev \
		libpng-dev \
		libpq-dev \
		libwebp-dev \
		libzip-dev \
		# Other tools
		git \
		unzip \
		patch \
	; \
	\
	# Configure and install PHP extensions
	docker-php-ext-configure gd --with-freetype --with-jpeg --with-webp; \
	docker-php-ext-install -j "$(nproc)" gd zip; \
	\
	# --- Cleanup ---
	# This removes build dependencies and cleans apt cache
	apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false; \
	rm -rf /var/lib/apt/lists/*

# --- 2. Apache Configuration ---
# Enable desired Apache modules
RUN a2enmod rewrite headers expires

# --- 3. User and Permissions Setup ---
# This is the key step for changing the run user's ID
RUN usermod -u ${UID} www-data && groupmod -g ${GID} www-data

# --- 4. Application Setup ---
# Install Composer globally
COPY --from=composer:2 /usr/bin/composer /usr/local/bin/composer

# Switch to the non-root user for all subsequent commands and for runtime
# This is a critical security best practice.
USER www-data

# Example: Now you can run Composer commands as the correct user
# RUN composer install --no-dev --optimize-autoloader
# COPY . .
