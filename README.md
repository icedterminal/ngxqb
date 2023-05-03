# NGXQB
<p align="center">
:warning: <em>Absolutely no support is provided. This is for my own personal use. You are welcome to use it though.</em> :warning:
</p>

A custom build of NGINX server for the modern web with OpenSSL 3+ (HTTP/3 + QUIC), Brotli and additional components compiled into one.

The target OS is Ubuntu 20.04 and later. 18.04 and earlier is untested. No builds or instructions for containers or other distributions will be provided. I have no interest. [NGINX Proxy Manager](https://nginxproxymanager.com/guide/) is available for those seeking containers or have little know-how. I do not know if this version of NGINX supports HTTP/3 + QUIC. This build is intended for my personal use. I prefer to write manual vhost/conf files or need the software to run on bare metal. If you fit this, feel free to use it.

---

# Components

| Name | Purpose |
| --- | --- |
| [NGINX QUIC](https://hg.nginx.org/nginx-quic) | This is the core of it all. Contains support for HTTP/3 and QUIC connections. |
| [OpenSSL](https://github.com/quictls/openssl) | Alternatively called "quictls", this fork is based on the latest OpenSSL with QUIC implimentations. Requried for this build of NGINX. |
| [PCRE2](https://github.com/PCRE2Project/pcre2/releases/) | Enables regular expression support for NGINX. |
| [Zlib](https://github.com/madler/zlib) | Standard compression method. |
| [Brotli](https://github.com/google/ngx_brotli) | An improved compression method over GZip (zlib). [All major browsers](https://caniuse.com/?search=Brotli) currently support Brotli. There is little need for GZip, however, the latest version is included for priority over system installed zlib package. |
| [Dev kit](https://github.com/vision5/ngx_devel_kit) | Extends NGINX functionality. Required by Set Misc. |
| [Set Misc](https://github.com/openresty/set-misc-nginx-module) | Enables the use of `set_xxx` directives. These are required for services like [Authelia](https://www.authelia.com/integration/proxies/nginx/). |

# Use
You can either use the prebuilt binary, or build yourself. Installer packages are currently not provided.

## Prebuilt
1. Download the zip from releases.
2. Place the zip at the root of your system.
3. `unzip -o nginx.zip; systemctl daemon-reload; systemctl enable nginx; systemctl start nginx`

Visit `http://localhost:80` or `http://127.0.0.1:80` to verify. Because there are no files present, you will be greeted by a `404` error page.

## Build yourself
### Prep
```bash
apt install git gcc cmake mercurial libpcre3 libpcre3-dev zlib1g zlib1g-dev libperl-dev libxslt1-dev libgd-ocaml-dev libgeoip-dev -y;
```
```bash
git clone https://github.com/icedterminal/ngxqb.git; cd ngxqb; git submodule update --init --recursive; cd pcre2-10.42; chmod +x configure; ./configure; cd ../nginx*;
```

### Configure
You may need to edit the configuration parameters to suit your needs. [A complete list is here](https://nginx.org/en/docs/configure.html).

```bash
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
--with-http_v2_module \
--with-http_v3_module \
--with-http_image_filter_module \
--with-http_xslt_module \
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
--add-module=../ngx_devel_kit \
--add-module=../set-misc-nginx-module \
--with-cc-opt='-g -O2 -fstack-protector-strong -Wformat -Werror=format-security -Wp,-D_FORTIFY_SOURCE=2 -fPIC' \
--with-ld-opt='-Wl,-Bsymbolic-functions -Wl,-z,relro -Wl,-z,now -Wl,--as-needed -pie'
```
### Build and install
```bash
make
```
Once building completes, you won't have the required structure in place to start NGINX. You'll need to do this:
```bash
cp objs/nginx /usr/sbin/nginx; chmod 755 /usr/sbin/nginx
```
Create a startup service:
```bash
nano /lib/systemd/system/nginx.service
```
Paste the following contents in:
```
[Unit]
Description=NGINX-QUIC web server
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
```bash
mkdir -p /etc/nginx/{dh,modules,sites-available,sites-disabled,conf.d} /var/cache/nginx/{client_temp,proxy_temp,fastcgi_temp,uwsgi_temp,scgi_temp} /var/log/nginx /var/www/html 
```
Set the permissions:
```bash
chown www-data:adm /var/log/nginx; chmod 755 /var/log/nginx; find /var/cache/nginx -type d | xargs chown www-data:root; find /var/cache/nginx -type d | xargs chmod 755
```
Copy the default files to the proper location:
```bash
cp -r conf/. /etc/nginx/; cp -r docs/html/. /var/www/html/;
```
Enable and start:
```bash
systemctl daemon-reload; systemctl enable nginx; systemctl start nginx
```

You can check your NGINX build information with `nginx -V`.

## Verify
- https://http3check.net/
- https://geekflare.com/tools/http3-test
