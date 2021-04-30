ARG NGINX_VERSION=1.20.0

FROM nginx:${NGINX_VERSION}-alpine AS builder

ARG NGINX_DAV_EXT_MODULE=3.0.0

# Download sources
RUN wget "https://nginx.org/download/nginx-${NGINX_VERSION}.tar.gz" -O nginx.tar.gz
RUN wget "https://github.com/arut/nginx-dav-ext-module/archive/refs/tags/v${NGINX_DAV_EXT_MODULE}.tar.gz" -O nginx-dav-ext-module.tar.gz

# For latest build deps, see https://github.com/nginxinc/docker-nginx/blob/master/mainline/alpine/Dockerfile
RUN apk add --no-cache --virtual .build-deps \
                gcc \
                libc-dev \
                make \
                openssl-dev \
                pcre-dev \
                zlib-dev \
                linux-headers \
                libxslt-dev \
                gd-dev \
                geoip-dev \
                perl-dev \
                libedit-dev \
                mercurial \
                bash \
                alpine-sdk \
                findutils

# Reuse same cli arguments as the nginx:alpine image used to build
RUN CONFARGS=$(nginx -V 2>&1 | sed -n -e 's/^.*arguments: //p') \
  mkdir -p /usr/src && \
  tar -zxC /usr/src -f nginx.tar.gz && \
  tar -zxC /usr/src -f nginx-dav-ext-module.tar.gz && \
  cd /usr/src/nginx-$NGINX_VERSION && \
  ./configure --with-compat $CONFARGS --add-dynamic-module=/usr/src/nginx-dav-ext-module-${NGINX_DAV_EXT_MODULE} && \
  make modules

FROM nginx:${NGINX_VERSION}-alpine

COPY --from=builder /usr/src/nginx-${NGINX_VERSION}/objs/ngx_http_dav_ext_module.so /usr/lib/nginx/modules/

CMD ["nginx", "-g", "daemon off; load_module /usr/lib/nginx/modules/ngx_http_dav_ext_module.so;"]
