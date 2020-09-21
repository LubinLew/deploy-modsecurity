# Deploy Modsecurity v3 on CentOS 7

```bash
# modesecurity version
MSVER=v3.0.4
# lua version
LUAVER=5.3.5
# GeoIP/libmaxmind version
GEOVER=1.4.3

## install tools
yum install -y wget git automake autoconf libtool gcc-c++ 

## install compile dependencies
yum install -y libxml2-devel pcre-devel libcurl-devel libevent-devel ncurses-devel readline-devel \
               yajl-devel lmdb-devel ssdeep-devel libtermcap-devel 

## install Lua
curl -R -O http://www.lua.org/ftp/lua-${LUAVER}.tar.gz
tar zxf lua-${LUAVER}.tar.gz
cd lua-${LUAVER}
make linux
make install
cd ..

## install GeoIP/libmaxmind
wget https://github.com/maxmind/libmaxminddb/releases/download/${GEOVER}/libmaxminddb-${GEOVER}.tar.gz
tar xf libmaxminddb-${GEOVER}.tar.gz
cd libmaxminddb-${GEOVER}
./configure --prefix=/usr/local --libdir=/usr/local/lib64
make install
cd ..

wget https://github.com/SpiderLabs/ModSecurity/releases/download/${MSVER}/modsecurity-${MSVER}.tar.gz
tar xf modsecurity-${MSVER}.tar.gz
cd modsecurity-${MSVER}
```
