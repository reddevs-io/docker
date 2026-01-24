# Use Alpine Linux as the base image
FROM alpine:3.23

# Install mysql-client package which includes both mysql and mysqldump
RUN apk add --no-cache mysql-client

# Set the working directory
WORKDIR /app

# Command to keep container running (can be overridden)
CMD ["tail", "-f", "/dev/null"]
