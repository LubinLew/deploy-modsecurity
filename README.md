# Deploy Modsecurity v3 on CentOS 7

## Compile
```bash
# modesecurity version
MSVER=v3.0.4

# libmaxmind version
GEOVER=1.4.3

# lua version,Modsecurity v3 only support Lua5.3(not support Lua5.4)
LUAVER=5.3.5

##################################################################################################

## install tools
yum install -y wget git automake autoconf libtool gcc-c++ 

## install compile dependencies
yum install -y libxml2-devel pcre-devel libcurl-devel libevent-devel ncurses-devel readline-devel \
               yajl-devel lmdb-devel ssdeep-devel libtermcap-devel 

PWD=`pwd`
mkdir build
ROOT=${PWD}/build
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
./configure --prefix=${MAXMINDDIR} --enable-shared=no
make install
cd ..

# build modsecurity v3
wget https://github.com/SpiderLabs/ModSecurity/releases/download/${MSVER}/modsecurity-${MSVER}.tar.gz
tar xf modsecurity-${MSVER}.tar.gz
cd modsecurity-${MSVER}
./configure --with-lua=${LUADIR} --with-maxmind=${MAXMINDDIR}
```

## Deploy



## See Also
- [lua](https://github.com/lua/lua)
- [libmaxminddb](https://github.com/maxmind/libmaxminddb)
