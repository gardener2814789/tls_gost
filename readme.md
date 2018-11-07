# Build Nginx 1.14 + OpenSSL 1.1.1 + gost-engine


### Prepare

```sh
apt-get install -y libpcre3-dev cmake zlib1g-dev unzip rsync
### Create folders
mkdir -p /home/user/proj/src
mkdir -p /home/user/proj/bin/nginx2
mkdir -p /www/
mkdir -p /var/log/nginx
```

### Build Nginx + OpenSSL
```sh
cd /home/user/proj/src
wget https://nginx.org/download/nginx-1.14.0.tar.gz
wget https://www.openssl.org/source/openssl-1.1.1.tar.gz
tar -zxvf nginx-1.14.0.tar.gz
tar -zxvf openssl-1.1.1.tar.gz
cd nginx-1.14.0
sed -i 's|--prefix=$ngx_prefix no-shared|--prefix=$ngx_prefix|' auto/lib/openssl/make
./configure --prefix=/home/user/proj/bin/nginx2 \
            --user=nginx --group=nginx --with-http_ssl_module \
            --with-openssl=/home/user/proj/src/openssl-1.1.1
make
make install
useradd --no-create-home nginx
cd ..
```

### Build gost-engine
```sh
wget https://github.com/gost-engine/engine/archive/master.zip
unzip master.zip
cd engine-master; mkdir build
cd build
cmake -DOPENSSL_ROOT_DIR=/home/user/proj/src/openssl-1.1.1 -DCMAKE_BUILD_TYPE=Release ..
cmake --build . --config Release
make install
```

### Configure OpenSSL + gost-engine
```sh
mkdir -p /home/user/proj/src/openssl-1.1.1/.openssl/ssl
# PUT openssl.cnf HERE
echo /home/user/proj/src/openssl-1.1.1 > /etc/ld.so.conf.d/openssl-1.1.1.conf
ldconfig -v
```


### Create GOST-keys
```sh
cd /home/user/proj/bin/nginx2/conf; mkdir keys; cd keys;
openssl genpkey -algorithm gost2001 -pkeyopt paramset:A -out gost.key
openssl req -engine gost -new -key gost.key -out gost.csr
```


