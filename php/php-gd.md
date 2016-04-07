# 安装gd插件
## 问题：
sudo apt-get install libpng-dev libjpeg-dev
安装GD插件后， 仍然出现
`
Call to undefined function imagecreatefromjpeg()
`
这个是由于

如果只有PNG/GIF Support，而没有JPEG Support那一项，那意味着libjpeg没有被编译进GD2.1里面去，

首先安装jpeg：

  `
  tar zxvf jpegsrc.v8d.tar.gz   
  cd jpeg-8d/  
  ./configure --prefix=/usr/local/jpeg --enable-shared  
  make && make install  
  `

然后重新编译GD：

`
cd ../gd  
phpize  
./configure --with-jpeg-dir=/usr/local/jpeg   
make clean  
make && make install  
`
