FROM alpine:3.9

# Default vars
ENV PARENT_PROXY_HOST=proxy.example.com
ENV PARENT_PROXY_PORT=8080
ENV PARENT_PROXY_ID=hoge@example.com
ENV PARENT_PROXY_PW=hogepasswd

ENV CLIENT_SIDE_PORT=3128

# Install package
RUN apk --no-cache add squid gettext

EXPOSE ${CLIENT_SIDE_PORT}

# Create template conf
RUN { \
      echo 'http_port ${CLIENT_SIDE_PORT}'; \
      echo 'acl all_ports port 0-65535'; \
      echo 'acl CONNECT method CONNECT'; \
      echo 'http_access allow all_ports'; \
      echo 'http_access allow CONNECT all_ports'; \
      echo 'http_access allow all'; \
      echo 'never_direct allow CONNECT'; \
      echo 'cache_peer ${PARENT_PROXY_HOST} parent ${PARENT_PROXY_PORT} 0 no-query no-netdb-exchange login=${PARENT_PROXY_ID}:${PARENT_PROXY_PW}'; \
      echo 'shutdown_lifetime 5 seconds'; \
      echo 'logfile_rotate 0'; \
      echo 'access_log stdio:/dev/stdout squid'; \
      echo 'pid_filename /var/run/squid/squid.pid'; \
    } > /etc/squid/squid.conf.template \
    && chown -R squid:squid /etc/squid

# Create endpoint script
RUN { \
      echo '#!/bin/sh -e'; \
      echo 'if [ $1 = "squid" ]; then'; \
      echo '  envsubst < /etc/squid/squid.conf.template > /etc/squid/squid.conf'; \
      echo '  set -- /usr/sbin/squid -f /etc/squid/squid.conf -N -d 1'; \
      echo 'fi'; \
      echo 'exec "$@"'; \
    } > /usr/local/bin/docker-entrypoint.sh \
    && chmod +x /usr/local/bin/docker-entrypoint.sh

ENTRYPOINT ["docker-entrypoint.sh"]

USER squid

CMD ["squid"]