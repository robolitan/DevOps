#Dockerfile

ARG NGINX_VERSION=1.13.7

FROM debian:9 as builder
ARG NGINX_VERSION
ENV LUA_VERSION 5.1
ENV LUAJIT2_VERSION 2.1-20171103
ENV NGINX_VERSION ${NGINX_VERSION}
ENV NGINX_NDK_VERSION 0.3.0
ENV NGINX_LUA_HTTP_MODULE_VERSION 0.10.11
ENV NGINX_LUA_RESTY_CORE_VERSION 0.1.13

WORKDIR /opt
RUN set -x \
	&& apt update \
	&& apt install -y wget build-essential libpcre3 libpcre3-dev libghc-zlib-dev libssl1.0-dev
RUN set -x \
  && wget -O luajit2-${LUAJIT2_VERSION}.tar.gz https://github.com/openresty/luajit2/archive/v${LUAJIT2_VERSION}.tar.gz \
  && tar -xzvf luajit2-${LUAJIT2_VERSION}.tar.gz \
  && cd luajit2-${LUAJIT2_VERSION} \
  && make -j2 \
  && make install
RUN set -x \
  && wget http://nginx.org/download/nginx-${NGINX_VERSION}.tar.gz \
  && tar -xzvf nginx-${NGINX_VERSION}.tar.gz
RUN set -x \
  && wget -O nginx-ndk-${NGINX_NDK_VERSION}.tar.gz https://github.com/simpl/ngx_devel_kit/archive/v${NGINX_NDK_VERSION}.tar.gz \
  && tar -xzvf nginx-ndk-${NGINX_NDK_VERSION}.tar.gz
RUN set -x \
  && wget -O lua-nginx-module-${NGINX_LUA_HTTP_MODULE_VERSION}.tar.gz https://github.com/openresty/lua-nginx-module/archive/v${NGINX_LUA_HTTP_MODULE_VERSION}.tar.gz \
  && tar -xzvf lua-nginx-module-${NGINX_LUA_HTTP_MODULE_VERSION}.tar.gz
RUN set -x \
  && wget -O lua-resty-core-${NGINX_LUA_RESTY_CORE_VERSION}.tar.gz https://github.com/openresty/lua-resty-core/archive/v${NGINX_LUA_RESTY_CORE_VERSION}.tar.gz \
  && tar -xzvf lua-resty-core-${NGINX_LUA_RESTY_CORE_VERSION}.tar.gz
RUN set -x \
  && cd nginx-${NGINX_VERSION}/ \
  && export LUAJIT_LIB=/usr/local/lib \
  && export LUAJIT_INC=/usr/local/include/luajit-2.1 \
  && ./configure --prefix=/opt \
      --prefix=/etc/nginx \
      --sbin-path=/usr/sbin/nginx \
      --modules-path=/usr/lib/nginx/modules \
      --conf-path=/etc/nginx/nginx.conf \
      --error-log-path=/var/log/nginx/error.log \
      --http-log-path=/var/log/nginx/access.log \
      --pid-path=/var/run/nginx.pid \
      --lock-path=/var/run/nginx.lock \
      --with-cc-opt='-g -O2 -fdebug-prefix-map=/data/builder/debuild/nginx-${NGINX_VERSION}/debian/debuild-base/nginx-${NGINX_VERSION}=. -specs=/usr/share/dpkg/no-pie-compile.specs -fstack-protector-strong -Wformat -Werror=format-security -Wp,-D_FORTIFY_SOURCE=2 -fPIC' \
      --with-ld-opt='-specs=/usr/share/dpkg/no-pie-link.specs -Wl,-z,relro -Wl,-rpath,/usr/lib/x86_64-linux-gnu,-z,now -Wl,--as-needed -pie' \
      --add-module=/opt/ngx_devel_kit-${NGINX_NDK_VERSION} \
      --add-module=/opt/lua-nginx-module-${NGINX_LUA_HTTP_MODULE_VERSION} \
  && make -j2 \
  && make install
RUN set -x \
  && cd lua-resty-core-${NGINX_LUA_RESTY_CORE_VERSION} \
  && make install

#FROM nginx:${NGINX_VERSION}
FROM debian:9
LABEL maintainer="Lambert Artem <27.11.2020>"
RUN set -x \
	&& apt update \
	&& apt install -y libssl1.0.2 \
	&& apt purge -y --auto-remove \
	&& rm -rf /var/lib/apt/lists/*
RUN set -x \
	&& mkdir /var/log/nginx /var/cache/nginx \
	&& touch /var/log/nginx/error.log \
	&& useradd --no-create-home nginx 
COPY --from=builder /etc/nginx /etc/nginx
COPY --from=builder /usr/sbin/nginx /usr/sbin/nginx
COPY --from=builder /usr/local/lib/libluajit-5.1.so.2.1.0 /usr/lib/x86_64-linux-gnu/libluajit-5.1.so.2
COPY --from=builder /usr/local/lib/lua /usr/local/share/lua
CMD ["/usr/sbin/nginx","-g","daemon off;"]
