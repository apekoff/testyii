FROM nginx:alpine

ARG VIRTUAL_HOST

ENV VIRTUAL_HOST=${VIRTUAL_HOST}

COPY ./default.conf /etc/nginx/conf.d/default.conf

RUN sed -i s/VIRTUAL_HOST/$VIRTUAL_HOST/g /etc/nginx/conf.d/default.conf

RUN adduser -D -H -u 1000 www-data
