ARG TARGET_PHP_VERSION=7.3
FROM php:${TARGET_PHP_VERSION}-cli

COPY ./code/ /code/

# since we're using context from the docker-compose.yml, we need to specify the directory from the context specified.
ARG SERVICE_DIR="./.docker/workspace"
COPY ./.docker/.shared/scripts/ /tmp/scripts/
RUN chmod +x -R /tmp/scripts/

# set timezone
ARG TZ=UTC
RUN /tmp/scripts/set_timezone.sh ${TZ}

# add users
ARG APP_USER=www-data
ARG APP_GROUP=www-data
ARG APP_USER_ID=1000
ARG APP_GROUP_ID=1000

RUN /tmp/scripts/create_user.sh ${APP_USER} ${APP_GROUP} ${APP_USER_ID} ${APP_GROUP_ID}

RUN /tmp/scripts/install_php_extensions.sh

RUN /tmp/scripts/install_software.sh

#####################################
# OCI8:
#####################################
ENV INSTALL_OCI8 true 

COPY ${SERVICE_DIR}/oci8.ini /tmp/

ENV LD_LIBRARY_PATH /usr/local/instantclient/

RUN if [ ${INSTALL_OCI8} = true ]; then \
    apt-get update && \
    apt-get install -y zip unzip libaio1 wget && \
    wget "https://s3-sa-east-1.amazonaws.com/4success-assets/instantclient-basic-linux.x64-12.2.0.1.0.zip" -P "/tmp" && \
        wget "https://s3-sa-east-1.amazonaws.com/4success-assets/instantclient-sdk-linux.x64-12.2.0.1.0.zip" -P "/tmp" && \
    unzip /tmp/instantclient-basic-linux.x64-12.2.0.1.0.zip -d /usr/local/ && \
    unzip /tmp/instantclient-sdk-linux.x64-12.2.0.1.0.zip -d /usr/local/ && \
    ln -s /usr/local/instantclient_12_2 /usr/local/instantclient && \
    ln -s /usr/local/instantclient/libclntsh.so.12.1 /usr/local/instantclient/libclntsh.so && \
    export LD_LIBRARY_PATH=/usr/local/instantclient && \
    echo 'instantclient,/usr/local/instantclient' | pecl install oci8 \
    # cp /tmp/oci8.ini /etc/php/7.2/cli/conf.d/oci8.ini \
;fi

RUN rm -rf /tmp/*.zip && \
    rm -rf /tmp/oci8.ini
# php config
COPY ./.docker/.shared/config/php/conf.d/*  /usr/local/etc/php/conf.d/

# workdir
ARG APP_CODE_PATH="/var/www/current"
WORKDIR "$APP_CODE_PATH"

# entrypoint
RUN mkdir -p /bin/docker-entrypoint/ \
 && cp /tmp/scripts/docker-entrypoint/* /bin/docker-entrypoint/ \
 && chmod +x -R /bin/docker-entrypoint/ \
;

RUN /tmp/scripts/cleanup.sh
ENTRYPOINT ["/bin/docker-entrypoint/resolve-docker-host-ip.sh"]
