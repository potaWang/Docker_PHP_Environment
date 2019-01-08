FROM nginx:1.15.6-alpine
LABEL maintainer="pota <wrpota@gmail.com>"
ENV TZ=Asia/Shanghai

RUN mkdir -p /run/nginx \
    && sed -i 's/dl-cdn.alpinelinux.org/mirrors.ustc.edu.cn/g' /etc/apk/repositories \
    && sed -i 's/http/https/g' /etc/apk/repositories \
    && apk upgrade \
    && apk add --no-cache bash \
    && adduser -D -H -u 1000 -s /bin/bash www-data \
    && mv /etc/nginx/conf.d/default.conf /etc/nginx/conf.d/default.conf1

COPY nginx.conf /etc/nginx/nginx.conf

CMD ["nginx", "-g", "daemon off;"]

EXPOSE 80 443