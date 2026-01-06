FROM alpine:3.19

RUN apk add --no-cache pgbouncer

RUN adduser -D -H -s /sbin/nologin pgbouncer \
    && mkdir -p /etc/pgbouncer /var/run/pgbouncer \
    && chown -R pgbouncer:pgbouncer /etc/pgbouncer /var/run/pgbouncer

USER pgbouncer

EXPOSE 6432

CMD ["pgbouncer", "/etc/pgbouncer/pgbouncer.ini"]
