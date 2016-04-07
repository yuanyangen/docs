
# ubuntu 源码编译php

    sudo apt-get install -f libxml2-dev libssl-dev libpng-dev libjpeg-dev

    wget http://www.ijg.org/files/jpegsrc.v8d.tar.gz
    ./configure --prefix=/usr/local/jpeg --enable-shared  
    make
    make install

    wget http://jaist.dl.sourceforge.net/project/mcrypt/Libmcrypt/2.5.8/libmcrypt-2.5.8.tar.gz
    wget http://jaist.dl.sourceforge.net/project/mcrypt/MCrypt/2.6.8/mcrypt-2.6.8.tar.gz
    wget http://jaist.dl.sourceforge.net/project/mhash/mhash/0.9.9.9/mhash-0.9.9.9.tar.gz

    tar -zxvf libmcrypt-2.5.8.tar.gz
    cd libmcrypt-2.5.8
    ./configure
    make
    sudo make install

    3.安装mhash
    tar -zxvf mhash-0.9.9.9.tar.gz
    cd mhash-0.9.9.9
    ./configure
    make
    sudo make install

    4.安装mcrypt
    tar -zxvf mcrypt-2.6.8.tar.gz
    cd mcrypt-2.6.8
    LD_LIBRARY_PATH=/usr/local/lib ./configure
    make
    sudo make install

    wget http://cn2.php.net/distributions/php-7.0.5.tar.bz2
    tar -xf php-7.0.5.tar.bz2
    ./configure --enable-fpm  --enable-zip --with-openssl  --with-mcrypt  --enable-mbstring=all --enable-tokenizer --with-pdo-mysql --with-gd
    make
    sudo make install
