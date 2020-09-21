# Deploy Modsecurity v3 on CentOS 7

## Compile
```bash
#!/usr/bin/bash
##################################################################################################

# modesecurity version
MSVER=v3.0.4

# libmaxmind version
GEOVER=1.4.3

# lua version,Modsecurity v3 only support Lua5.3(not support Lua5.4)
LUAVER=5.3.5

# nginx version(stable)
NGXVER=1.18.0

# ModSecurity-nginx version
MNVER=v1.0.1
##################################################################################################

## install tools
yum install -y wget git automake autoconf libtool gcc-c++ 

## install compile dependencies
yum install -y libxml2-devel pcre-devel libcurl-devel ncurses-devel readline-devel \
               yajl-devel lmdb-devel ssdeep-devel libtermcap-devel

## install nginx
cat > /etc/yum.repos.d/nginx.repo <<EOF
[nginx-stable]
name=nginx stable repo
baseurl=http://nginx.org/packages/centos/\$releasever/\$basearch/
gpgcheck=1
enabled=1
gpgkey=https://nginx.org/keys/nginx_signing.key
module_hotfixes=true
EOF
yum install -y nginx-${NGXVER}

mkdir build
cd build
ROOT=${PWD}
echo "BUILD PATH ${ROOT}"
##################################################################################################

## build Lua(local static library)
curl -R -O http://www.lua.org/ftp/lua-${LUAVER}.tar.gz
tar zxf lua-${LUAVER}.tar.gz
cd lua-${LUAVER}
make linux
make local
LUADIR=${ROOT}/lua-${LUAVER}/install
cd ..

## build libmaxmind(local static library)
wget https://github.com/maxmind/libmaxminddb/releases/download/${GEOVER}/libmaxminddb-${GEOVER}.tar.gz
tar xf libmaxminddb-${GEOVER}.tar.gz
cd libmaxminddb-${GEOVER}
mkdir install
MAXMINDDIR=${ROOT}/libmaxminddb-${GEOVER}/install
./configure --prefix=${MAXMINDDIR} --enable-shared=no CFLAGS="-fPIC"
make install
cd ..

## build modsecurity v3
wget https://github.com/SpiderLabs/ModSecurity/releases/download/${MSVER}/modsecurity-${MSVER}.tar.gz
tar xf modsecurity-${MSVER}.tar.gz
cd modsecurity-${MSVER}
./configure --with-lua=${LUADIR} --with-maxmind=${MAXMINDDIR}
make
make install
# copy conf
mkdir -p /etc/nginx/modsecurity
cp modsecurity.conf-recommended /etc/nginx/modsecurity/modsecurity.conf
cp unicode.mapping /etc/nginx/modsecurity/
cd ..

## build connector (dynamic library)
wget https://github.com/SpiderLabs/ModSecurity-nginx/releases/download/v1.0.1/modsecurity-nginx-${MNVER}.tar.gz
tar xf modsecurity-nginx-${MNVER}.tar.gz
wget http://nginx.org/download/nginx-${NGXVER}.tar.gz
tar xf nginx-${NGXVER}.tar.gz
cd nginx-${NGXVER}
./configure --add-dynamic-module=${ROOT}/modsecurity-nginx-${MNVER} --with-compat
make
cp objs/ngx_http_modsecurity_module.so /etc/nginx/modules/
cd ..
echo "================= DONE ================="
```

## Deploy
```bash
#!/usr/bin/bash

##################################################################################################

# coreruleset verson
CRSVER=3.3.0

##################################################################################################

## download crs rules
cd /etc/nginx/modsecurity
wget https://github.com/coreruleset/coreruleset/archive/v${CRSVER}.tar.gz
tar xf v${CRSVER}.tar.gz
cd ..

## crs config
cat > /etc/nginx/modsecurity/crs-setup.conf << EOF
  SecDefaultAction "phase:1,log,auditlog,deny,status:403"
  SecDefaultAction "phase:2,log,auditlog,deny,status:403"
EOF

## modsecurity config
cat > /etc/nginx/modsecurity/modsecurity.conf << EOF
  SecRuleEngine On
  SecDebugLog /var/log/modsec_debug.log
  SecDebugLogLevel 9
  SecRuleRemoveById 10
  Include /etc/nginx/modsecurity/crs-setup.conf
  Include /etc/nginx/modsecurity/coreruleset-${CRSVER}/rules/*.conf
EOF

## nginx conf(listen 8080)
# load module
sed -i '8 s/^/load_module modules/ngx_http_modsecurity_module.so;\n/' /etc/nginx/nginx.conf
# add new server
cat > /etc/nginx/conf.d/modsecurity.conf <<EOF
server {
    listen [::]:8080 default_server;
    modsecurity on;
    location / {
        root /usr/share/nginx/html;
        modsecurity_rules_file /etc/nginx/modsecurity/modsecurity.conf;
    }
}
EOF

## restart nginx service
systemctl restart nginx
##################################################################################################

# test
curl -I "http://localhost:8080/?id=<script>alert('xss')</script>"
```



## See Also
- [lua](https://github.com/lua/lua)
- [libmaxminddb](https://github.com/maxmind/libmaxminddb)
- [ModSecurity](https://github.com/SpiderLabs/ModSecurity)
- [ModSecurity-nginx](https://github.com/SpiderLabs/ModSecurity-nginx)
