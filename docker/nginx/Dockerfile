FROM nginx:alpine

COPY nginx.conf /etc/nginx/

RUN apk update \
    && apk upgrade \
    && apk add --no-cache bash \
    && adduser -D -H -u 1000 -s /bin/bash www-data

# Set upstream conf and remove the default conf
COPY upstream.conf /etc/nginx/conf.d/
RUN rm /etc/nginx/conf.d/default.conf

CMD ["nginx"]

EXPOSE 80 443
