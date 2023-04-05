# NGXQB
<p align="center">
:warning: <em>Absolutely no support is provided. This is for my own personal use. You are welcome to use it though.</em> :warning:
</p>
A custom build of NGINX server for the modern web with OpenSSL 3+ (HTTP/3 + QUIC), Brotli, Zlib, and PCRE2 compiled into one.

---

| Component | Source |
| --- | --- |
| NGINX QUIC | Mirrored via `hg clone https://hg.nginx.org/nginx-quic; hg update quic;` |
| PCRE2 | Download via https://github.com/PCRE2Project/pcre2/releases/ |
| Zlib | Submodule via https://github.com/madler/zlib |
| OpenSSL | Submodule via https://github.com/quictls/openssl |
| Brotli | Submodule via https://github.com/google/ngx_brotli |

**Target OS:** Ubuntu 20.04 and later. 18.04 and earlier is untested. No builds or instructions for containers, or other distirubtions will be provided. I have no interest.

## Prebuilt
1. Download the zip from releases.
2. Place it at the root of your system.
3. `unzip -o nginx.zip`
4. `systemctl daemon-reload; systemctl enable nginx; systemctl start nginx`

Visit `http://localhost:80` or `http://127.0.0.1:80` to verify. You should see welcome / success page.

## Build yourself
### Prep
```
apt install git gcc cmake mercurial libpcre3 libpcre3-dev zlib1g zlib1g-dev libperl-dev libxslt1-dev libgd-ocaml-dev libgeoip-dev -y;
```
```
git clone https://github.com/icedterminal/ngxqb.git; cd ngxqb/nginx*; git submodule update --init; cd ../ngx_brotli; git submodule update --init; cd ..; wget https://github.com/PCRE2Project/pcre2/releases/download/pcre2-10.42/pcre2-10.42.zip; unzip pcre2-10.42.zip; rm pcre2-10.42.zip; cd pcre2-10.42; chmod +x configure; ./configure; cd ../nginx*;
```

### Configure
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
--with-pcre=../pcre2-10.42 \
--with-openssl=../openssl \
--with-openssl-opt=enable-ktls \
--with-openssl-opt=enable-fips \
--add-module=../ngx_brotli \
--with-cc-opt='-I/src/libressl/build/include -g -O2 -fstack-protector-strong -Wformat -Werror=format-security -Wp,-D_FORTIFY_SOURCE=2 -fPIC' \
--with-ld-opt='-L/src/libressl/build/lib -Wl,-Bsymbolic-functions -Wl,-z,relro -Wl,-z,now -Wl,--as-needed -pie'
```
### Build and install
```
make
```
Once building completes, you won't have the required structure in place to start NGINX. You'll need to do this.
```
cp objs/nginx /usr/sbin/nginx; chmod 755 /usr/sbin/nginx
```
Create a startup service:
```
[nano|gedit] /lib/systemd/system/nginx.service
```
Paste the following contents in:
```
[Unit]
Description=nginx - high performance web server
Documentation=https://nginx.org/en/docs/
After=network-online.target remote-fs.target nss-lookup.target
Wants=network-online.target

[Service]
Type=forking
PIDFile=/var/run/nginx.pid
ExecStart=/usr/sbin/nginx -c /etc/nginx/nginx.conf
ExecReload=/bin/sh -c "/bin/kill -s HUP $(/bin/cat /var/run/nginx.pid)"
ExecStop=/bin/sh -c "/bin/kill -s TERM $(/bin/cat /var/run/nginx.pid)"

[Install]
WantedBy=multi-user.target
```
Create the initial directories:
```
mkdir -p /etc/nginx/{dh,modules,sites-available,sites-disabled,conf.d} /var/cache/nginx/{client_temp,proxy_temp,fastcgi_temp,uwsgi_temp,scgi_temp} /var/log/nginx /var/www/html 
```
Set the permissions:
```bash
chown www-data:adm /var/log/nginx; chmod 755 /var/log/nginx; find /var/cache/nginx -type d | xargs chown www-data:root; find /var/cache/nginx -type d | xargs chmod 755
```
Copy the default files to the proper location:
```
cp -r conf/. /etc/nginx/; cp -r docs/html/. /var/www/html/;
```
Enable and start
```
systemctl enable nginx; systemctl start nginx
```

You can check your NGINX build information with `nginx -V`.

## Verify
- https://http3check.net/
- https://geekflare.com/tools/http3-test
