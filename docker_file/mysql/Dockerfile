ARG MYSQL_VERSION=${MYSQL_VERSION}
FROM mysql:${MYSQL_VERSION}
LABEL maintainer="pota <wrpota@gmail.com>"

ADD startup /etc/mysql/startup

RUN chown -R mysql:root /var/lib/mysql/
   
CMD ["mysqld"]

EXPOSE 3306