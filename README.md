# NGXQB
[![Automated Builder](https://github.com/icedterminal/ngxqb/actions/workflows/c-cpp.yml/badge.svg)](https://github.com/icedterminal/ngxqb/actions/workflows/c-cpp.yml)
<p align="center">
:warning: <em>Absolutely no support is provided. This is for my own personal use. You are welcome to use it though.</em> :warning:
</p>

A custom build of NGINX server for the modern web with OpenSSL 3+ (HTTP/3 + QUIC), Brotli and additional components compiled into one.

---

# Components

| Name | Purpose |
| --- | --- |
| [NGINX](https://hg.nginx.org/nginx) | This is the core of it all. Contains support for HTTP/3 and QUIC connections. |
| [OpenSSL](https://github.com/quictls/openssl) | Alternatively called "quictls", this fork is based on the latest OpenSSL with QUIC implimentations. Requried for this build of NGINX. |
| [PCRE2](https://github.com/PCRE2Project/pcre2/releases/) | Enables regular expression support for NGINX. |
| [Zlib](https://github.com/madler/zlib) | Standard compression method. |
| [Brotli](https://github.com/google/ngx_brotli) | An improved compression method over GZip (zlib). [All major browsers](https://caniuse.com/?search=Brotli) currently support Brotli. There is little need for GZip, however, the latest version is included for priority over system installed zlib package. |
| [Dev kit](https://github.com/vision5/ngx_devel_kit) | Extends NGINX functionality. Required by Set Misc. |
| [Set Misc](https://github.com/openresty/set-misc-nginx-module) | Enables the use of `set_xxx` directives. These are required for services like [Authelia](https://www.authelia.com/integration/proxies/nginx/). |
| [NGINX JS](https://hg.nginx.org/njs/) | Enables the use of [server-side JavaScript](https://www.nginx.com/blog/harnessing-power-convenience-of-javascript-for-each-request-with-nginx-javascript-module/). |

# Use
You can either use the prebuilt binary, or build yourself. Installer packages are currently not provided.

https://www.icedterminal.me/ngxqb

<!--
## Prebuilt

### Initial
1. Download the latest zip from releases to the root (`/`) of your system while elevated as the `root` user.
    ```
    wget https://github.com/icedterminal/ngxqb/releases/latest/download/nginx-arch.zip
    wget https://github.com/icedterminal/ngxqb/releases/latest/download/nginx-debian.zip
    ```
3. Extract the contents
    ```bash
    unzip -o nginx-*.zip
    ```
4. Set the permissions
    ```bash
    chown [www-data|http]:adm /var/log/nginx; chmod 755 /var/log/nginx; find /var/cache/nginx -type d | xargs chown [www-data|http]:root; find /var/cache/nginx -type d | xargs chmod 755
    ```
5. Load the service (If you do not specify a user in `nginx.conf`, the default user will be used).
    ```bash
    systemctl daemon-reload; systemctl enable nginx
    ```
6. Start the service
    ```
    systemctl start nginx
    ```

Visit [`http://localhost:80`](http://localhost:80) or [`http://127.0.0.1:80`](http://127.0.0.1:80). The default `nginx.conf` file uses the build prefix path of `/etc/nginx`. You should store configuration files here. Not web files. The default html files are placed here so you can verify you have a working service. You are encouraged to use `/var/www/html` as your root for web files. You can delete the zip file if you like as there is no need to keep it.

You can check your NGINX build information with `nginx -V`.

### Updates
1. Download the latest zip from releases to the root (`/`) of your system while elevated as the `root` user.
    ```bash
    wget https://github.com/icedterminal/ngxqb/releases/latest/download/nginx-arch.zip
    wget https://github.com/icedterminal/ngxqb/releases/latest/download/nginx-debian.zip
    ```
3. Stop the service, Extract updated binary, start the service
    ```bash
    systemctl stop nginx; unzip -oj nginx-*.zip "sbin/nginx" -d /sbin/; systemctl start nginx
    ```

You can check your NGINX build information with `nginx -V`.

## Build yourself
### Prep
**Debian based**
```bash
apt install git gcc cmake mercurial libpcre3 libpcre3-dev zlib1g zlib1g-dev libperl-dev libxslt1-dev libgd-ocaml-dev libgeoip-dev -y;
```
**Arch based**
```bash
pacman -Sy git gcc cmake mercurial base-devel
```
Now clone and init
```bash
git clone https://github.com/icedterminal/ngxqb.git; cd ngxqb; git submodule update --init --recursive; cd nginx;
```

### Configure
You may need to edit the configuration parameters to suit your needs. [A complete list is here](https://nginx.org/en/docs/configure.html).

```bash
./auto/configure \
--prefix=/etc/nginx \
--sbin-path=/usr/sbin/nginx \
--conf-path=/etc/nginx/nginx.conf \
--modules-path=/etc/nginx/modules \
--error-log-path=/var/log/nginx/error.log \
--http-log-path=/var/log/nginx/access.log \
--pid-path=/run/nginx.pid \
--lock-path=/run/nginx.lock \
--http-client-body-temp-path=/var/cache/nginx/client_temp \
--http-proxy-temp-path=/var/cache/nginx/proxy_temp \
--http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp \
--http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp \
--http-scgi-temp-path=/var/cache/nginx/scgi_temp \
--user=[www-data|http] \
--group=[www-data|http] \
--with-debug \
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
--with-http_dav_module \
--with-http_stub_status_module \
--with-http_slice_module \
--with-mail \
--with-mail_ssl_module \
--with-stream \
--with-stream_realip_module \
--with-stream_ssl_module \
--with-stream_ssl_preread_module \
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
Once building completes, you won't have the required structure in place to start NGINX. You'll need to do this as root:
```bash
sudo su
```
Create the initial directories:
```bash
mkdir -p /etc/nginx/{dh,modules,sites-available,sites-disabled,conf.d,html} /var/cache/nginx/{client_temp,proxy_temp,fastcgi_temp,uwsgi_temp,scgi_temp} /var/log/nginx /var/www/html 
```
Copy the default files to the proper location:
```bash
cp -r conf/. /etc/nginx/; cp -r docs/html/. /var/www/html/; cp -r docs/html/. /etc/nginx/html/; cp objs/nginx /usr/sbin/nginx; 
```
Set the permissions:
```bash
chmod 755 /usr/sbin/nginx; chown [www-data|http]:adm /var/log/nginx; chmod 755 /var/log/nginx; find /var/cache/nginx -type d | xargs chown [www-data|http]:root; find /var/cache/nginx -type d | xargs chmod 755
```
Create a startup service:
```bash
nano /etc/systemd/system/nginx.service
```
Paste the following contents in:
```
[Unit]
Description=NGINX web server
Documentation=https://nginx.org/en/docs/
After=network-online.target remote-fs.target nss-lookup.target
Wants=network-online.target

[Service]
Type=forking
PIDFile=/run/nginx.pid
ExecStart=/usr/sbin/nginx -c /etc/nginx/nginx.conf
ExecReload=/bin/sh -c "/bin/kill -s HUP $(/bin/cat /run/nginx.pid)"
ExecStop=/bin/sh -c "/bin/kill -s TERM $(/bin/cat /run/nginx.pid)"

[Install]
WantedBy=multi-user.target
```
Load the service.
```bash
systemctl daemon-reload; systemctl enable nginx
```
Verify the process user before you start the service!
```
systemctl start nginx
```

You can check your NGINX build information with `nginx -V`.

## Verify
- https://http3check.net/
- https://geekflare.com/tools/http3-test
-->
