```shell
# 安装必要依赖
yum install -y gcc zlib zlib-devel openssl openssl-devel

# 解压
tar zxvf Python-2.7.14.tgz
```

```shell
./configure --enable-optimizations --enable-shared --prefix=/usr/local/Python2.7
vim Modules/Setup

# 编辑后，将这三行的注释去除，以开启SSL模块
# ssl _ssl.c \
#       -DUSE_SSL -I$(SSL)/include -I$(SSL)/include/openssl \
#       -L$(SSL)/lib -lssl -lcrypto

make && make install
```

```shell
# python2.7可以软连接
# 先安装setuptools在安装pip
unzip setuptools-44.1.1.zip
cd setuptools-44.1.1
python2.7 setup.py install
tar zxvf pip-20.2.4.tar.gz
cd pip-20.2.4
python2.7 setup.py install
```

