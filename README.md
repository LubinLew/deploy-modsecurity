```bash
MSVER=v3.0.4
LUAVER=5.4.0
GEOVER=1.4.3

## install tools
yum install -y wget git automake autoconf libtool gcc-c++ 

## install compile dependencies
yum install -y libxml2-devel pcre-devel libcurl-devel yajl-devel lmdb-devel ssdeep-devel libtermcap-devel ncurses-devel libevent-devel readline-devel

## install Lua
curl -R -O http://www.lua.org/ftp/lua-${LUAVER}.tar.gz
tar zxf lua-${LUAVER}.tar.gz
cd lua-${LUAVER}
make all install

## GeoIP/maxmind
wget https://github.com/maxmind/libmaxminddb/releases/download/${GEOVER}/libmaxminddb-${GEOVER}.tar.gz
tar xf libmaxminddb-${GEOVER}.tar.gz
cd libmaxminddb-${GEOVER}
make install

wget https://github.com/SpiderLabs/ModSecurity/releases/download/${MSVER}/modsecurity-${MSVER}.tar.gz
tar xf modsecurity-${MSVER}.tar.gz
cd modsecurity-${MSVER}
```
