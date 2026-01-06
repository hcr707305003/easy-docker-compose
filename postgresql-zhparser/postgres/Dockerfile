# postgres-zhparser-example/docker/postgres/Dockerfile
# Stage 1: Builder
FROM postgres:16-alpine AS builder

# Use Aliyun mirror for faster downloads in China
RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g' /etc/apk/repositories \
    && echo "Using Aliyun mirror for Alpine Linux packages"

# Update package index first
RUN apk update --no-cache

# Install build dependencies for SCWS and zhparser
# Split into multiple steps for better error handling and progress visibility
RUN apk --no-cache add \
    git \
    build-base \
    linux-headers \
    make \
    bash

RUN apk --no-cache add \
    automake \
    libtool \
    autoconf \
    m4

RUN apk --no-cache add \
    postgresql-dev

RUN apk --no-cache add \
    libxml2-dev

# Install clang and llvm separately as they are large packages
RUN apk --no-cache add clang

RUN apk --no-cache add llvm

# Compile SCWS from source
WORKDIR /opt/build
COPY scws-1.2.2.tar.gz /opt/build/
RUN set -ex \
    && tar -xzf scws-1.2.2.tar.gz \
    && cd scws-1.2.2 \
    && touch README \
    && aclocal \
    && autoconf \
    && autoheader \
    && libtoolize --force \
    && automake --add-missing \
    && ./configure --prefix=/usr/local \
    && if ! grep -q '#include <unistd.h>' libscws/xdict.c; then \
        echo '#include <unistd.h>' | cat - libscws/xdict.c > libscws/xdict.c.tmp && \
        mv libscws/xdict.c.tmp libscws/xdict.c; \
    fi \
    && make \
    && make -C libscws install \
    && (make -C cli install 2>/dev/null || true) \
    && (make -C etc install 2>/dev/null || \
        (mkdir -p /usr/local/etc && \
         cp etc/rules*.ini /usr/local/etc/ 2>/dev/null || true))

# Compile zhparser
WORKDIR /opt/build
COPY zhparser-2.3.tar.gz /opt/build/
RUN set -ex \
    && tar -xzf zhparser-2.3.tar.gz \
    && cd zhparser-2.3 \
    && (which clang-19 >/dev/null 2>&1 || (CLANG_PATH=$(which clang) && mkdir -p /usr/local/bin && ln -sf $CLANG_PATH /usr/local/bin/clang-19)) \
    && make PG_CONFIG=/usr/local/bin/pg_config SCWS_HOME=/usr/local \
    && (make PG_CONFIG=/usr/local/bin/pg_config SCWS_HOME=/usr/local install || true) \
    && mkdir -p /usr/local/lib/postgresql /usr/local/share/postgresql/extension /usr/local/share/postgresql/tsearch_data \
    && install -m 755 zhparser.so /usr/local/lib/postgresql/ \
    && install -m 644 zhparser.control /usr/local/share/postgresql/extension/ \
    && install -m 644 zhparser--*.sql /usr/local/share/postgresql/extension/ \
    && install -m 644 dict.utf8.xdb rules.utf8.ini /usr/local/share/postgresql/tsearch_data/

# Stage 2: Final image
FROM postgres:16-alpine

# Use Aliyun mirror for faster downloads in China
RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g' /etc/apk/repositories \
    && echo "Using Aliyun mirror for Alpine Linux packages"

# Update package index first
RUN apk update --no-cache

# Set language environment variable for UTF-8 support
ENV LANG zh_CN.UTF-8

# Install runtime dependencies for SCWS (if it was linked against libxml2)
RUN apk --no-cache add libxml2

# Copy compiled artifacts from builder stage
COPY --from=builder /usr/local/lib/postgresql/zhparser.so /usr/local/lib/postgresql/
COPY --from=builder /usr/local/lib/libscws.so* /usr/local/lib/
COPY --from=builder /usr/local/share/postgresql/extension/zhparser.control /usr/local/share/postgresql/extension/
COPY --from=builder /usr/local/share/postgresql/extension/zhparser--*.sql /usr/local/share/postgresql/extension/
COPY --from=builder /usr/local/share/postgresql/tsearch_data/dict.utf8.xdb /usr/local/share/postgresql/tsearch_data/
COPY --from=builder /usr/local/share/postgresql/tsearch_data/rules.utf8.ini /usr/local/share/postgresql/tsearch_data/
# Copy bitcode if it exists (might not if not compiled with Clang/LLVM for PG JIT)
COPY --from=builder /usr/local/lib/postgresql/bitcode/zhparser* /usr/local/lib/postgresql/bitcode/

# Set LD_LIBRARY_PATH so PostgreSQL can find libscws.so
ENV LD_LIBRARY_PATH=/usr/local/lib${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}

# The init-db.sql script (mounted via docker-compose) will handle
# CREATE EXTENSION zhparser; and other TEXT SEARCH CONFIGURATION.

WORKDIR /