FROM php:7-fpm
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
ARG INSTALL_OCI8=false
ENV INSTALL_OCI8 ${INSTALL_OCI8}

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
    echo 'instantclient,/usr/local/instantclient' | pecl install oci8 && \
    cp /tmp/oci8.ini /etc/php/7.2/cli/conf.d/oci8.ini \
;fi

RUN rm -rf /tmp/*.zip && \
    rm -rf /tmp/oci8.ini

# set up ssh
RUN apt-get update -yqq && apt-get install -yqq openssh-server \
 && mkdir /var/run/sshd \
;

# add default public key to authorized_keys
USER ${APP_USER}
COPY ${SERVICE_DIR}/.ssh/insecure_id_rsa.pub /tmp/insecure_id_rsa.pub
RUN mkdir -p ~/.ssh \
 && cat /tmp/insecure_id_rsa.pub >> ~/.ssh/authorized_keys \
 && chown -R ${APP_USER}: ~/.ssh \
 && chmod 700 ~/.ssh \
 && chmod 600 ~/.ssh/authorized_keys \
;
USER root

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

# @see https://docs.docker.com/engine/examples/running_ssh_service/
CMD ["/usr/sbin/sshd", "-D"]
ENTRYPOINT ["/bin/docker-entrypoint/resolve-docker-host-ip.sh"]

