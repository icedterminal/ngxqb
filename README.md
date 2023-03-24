# NGINX + QUIC + Brotli

This is a custom build of NGINX server with OpenSSL 3+ (QUIC) and Brotli compiled.

Absolutely no support is provided. This is for my own personal use. You are welcome to use it if you find it useful.

Target OS: Ubuntu 20.04 and later.

```
git clone https://github.com/icedterminal/ngxqb.git; cd ngxqb/nginx*; git submodule update --init; cd ../ngx_brotli; git submodule update --init; cd ..; wget https://github.com/PCRE2Project/pcre2/releases/download/pcre2-10.42/pcre2-10.42.zip; unzip pcre2*.zip; rm pcre2*.zip; cd pcre*; chmod +x configure; ./configure; cd ../nginx*;
```
```
./auto/configure \
`nginx -V 2>&1 | sed "s/ \-\-/ \\\ \n\t--/g" | grep "\-\-" | grep -ve opt= -e param= -e build=` \
--build=nginx-quic \
--prefix=/etc/nginx \
--sbin-path=/usr/sbin/nginx \
--conf-path=/etc/nginx/nginx.conf \
--modules-path=/etc/nginx/modules \
--error-log-path=/var/log/nginx/error.log \
--http-log-path=/var/log/nginx/access.log \
--pid-path=/var/run/nginx.pid \
--lock-path=/var/run/nginx.lock \
--http-client-body-temp-path=/var/cache/nginx/client_temp \
--http-proxy-temp-path=/var/cache/nginx/proxy_temp \
--http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp \
--http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp \
--http-scgi-temp-path=/var/cache/nginx/scgi_temp \
--user=www-data \
--group=www-data \
--with-compat \
--with-file-aio \
--with-threads \
--with-http_auth_request_module \
--with-http_dav_module \
--with-http_gunzip_module \
--with-http_gzip_static_module \
--with-http_mp4_module \
--with-http_realip_module \
--with-http_ssl_module \
--with-http_stub_status_module \
--with-http_v2_module \
--with-http_v3_module \
--with-http_image_filter_module \
--with-http_xslt_module \
--with-http_dav_module \
--with-http_stub_status_module \
--with-stream \
--with-stream_realip_module \
--with-stream_ssl_module \
--with-stream_ssl_preread_module \
--with-stream_quic_module \
--with-zlib=../zlib \
--with-pcre=../pcre2 \
--with-openssl=../openssl \
--with-openssl-opt=enable-ktls \
--with-openssl-opt=enable-fips \
--add-module=../ngx_brotli \
--with-cc-opt='-I/src/libressl/build/include -g -O2 -fstack-protector-strong -Wformat -Werror=format-security -Wp,-D_FORTIFY_SOURCE=2 -fPIC' \
--with-ld-opt='-L/src/libressl/build/lib -Wl,-Bsymbolic-functions -Wl,-z,relro -Wl,-z,now -Wl,--as-needed -pie'
```
```
make
```
