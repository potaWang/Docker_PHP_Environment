FROM redis:4-alpine
LABEL maintainer="pota <wrpota@gmail.com>"
ENV TZ=${WORKSPACE_TIMEZONE}
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

VOLUME /data

EXPOSE 6379

CMD ["redis-server"]