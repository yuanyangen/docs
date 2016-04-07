# Windows 下进行训练

* 安装jre8
* 下载最新的jTessBoxEditor
* 创建文件夹， 并png文件放入， 命名规则为：prefix.font.exp0.png，
* 设置Language为prefix，设置bootstrap Language为prefix， 点击make box file only
* 在修复正生成的box file
* 设置bootstrap Language为prefix， 点击train with existing box， 最终得到prefix.traineddata
* 讲traineddata放入对应tesseract-tor的tessdata文件夹中, 在下次生成boxfile的时候 就可以根据这个使用了







# linux下进行训练

## 安装依赖
sudo apt-get install tesseract-ocr libicu-dev libpango1.0-dev libcairo2-dev leptonica libtool automake


Ubuntu 14.04 使用 tesseract-ocr 训练样本
简介

使用tesseract的时候, 发现一些肉眼很容易识别的字符, tesseract的识别效果不是很好, 就希望自己去培训样本, 提高识别率. 通过Google找到了一篇文章, 介绍如何训练样本. 但是发现在Ubuntu 14.04下apt-get安装的tesseract-ocr不包含训练样本所需命令: tesseract, unicharset_extractor, mftraining, cntraining, combine-tessdata.

后来Google, 定位到tesseract-orc在其Google Code的训练教程, 中说明需要自己在源码上编译安装training tools. 然后照着说明, 发现了不少问题.现记录下来.


下载tesseract-ocr 源码
进入 tesseract-ocr 解压目录, 安装里面 INSTALL 内说是只需:` ./autogen.sh & ./configure`, 就能生成make文件了, 但是autogen.sh中需要 libtool 和 automake, 通过命令sudo apt-get install libtool automake 安装以上两个依赖. 执行./configure 的时候由发现一个问题, 报出 leptonica not found 的错误, 导致生命 make 文件进行不下去. 需要安装 leptonica.
安装 leptonica. 这是一个什么包? 在其官网上, 可以看到说明: 开源的图片处理以及分析工具集. 通过下载源码进行编译安装.
编译安装训练工具. 进入 tesseract-ocr 源码文件夹, 输入如下命令: > ./configure & make cd training sudo make install


## 安装leptonica-1.7.3



        export LIBLEPT_HEADERSDIR=/home/alien/leptonica-1.73/src/
        ./autogen.sh
        ./configure --prefix= --with-extra-libraries=/home/alien/leptonica-1.73/src/
        make  
        sudo make install





## 安装jTessBoxEditor
这个工具可以用来将图片文件, 转化为tif格式. 用于生成训练样本box. 需要java运行环境.  这个工具建议在windows下面运行，下载完成包后， 安装jre8的环境， 再直接双击jar文件即可

    下载地址：
    http://tenet.dl.sourceforge.net/project/vietocr/jTessBoxEditor/jTessBoxEditor-1.5.zip


##训练过程
先使用原始的图片你生成tif图片， 在linux下可以使用一下命令实现：

    convert src.png -flatten -monochrome target.tif

 如果使用的是其他格式的图片， 例如jpg， 需要先转为 png 格式， 才能使用这个命令

### 生成box文件并对其进行修正：
* 在jTessBoxEditor中设置文件夹信息， 语言信息， 并设置菜单为只生成box文件
* 在box editor中对文字进行修正，保存。

这里也可以使用命令行进行创建：

    tesseract code.font.exp0.tif code.font.exp0 batch.nochop makebox

### 创建字体特这文件
 文件的名字为： 其中fontname为字体名称，必须与[lang].[fontname].exp[num].box中的名称保持一致。
<italic> 、<bold> 、<fixed> 、<serif>、 <fraktur>的取值为1或0，表示字体是否具有这些属性。
这里在样本图片所在目录下创建一个名称为prefix_font_properties的文件，用记事本打开，输入以下下内容：

    font 0 0 0 0 0  

### 创建常用的词与词库：
    创建文件名为： prefix.frequent_words_list 和prefix_words_list的文件， 文件内容设置为空



这里用到了 -psm 8 , Treat the image as a single text line. 因为我用到的图片样本都是单行, 如:

调整识别结果, 通过jTessBoxEditor打开box文件, 对默认识别结果调整.

### 从box文件训练样本

    tesseract code.font.exp0.tif code.font.exp0 box.train

    > 注意这里可以选择模式， 例如加上  -psm 7 等， 这里训练的时候， 要和最初创建的时候保持一致才行

    得到 code.font.exp0.tr 文件. (注: 这里 tesseract 用到的参数需要和 4 中的一样)

### 计算字集

    unicharset_extractor code.font.exp0.box

### 聚类

    shapeclustering -F font_properties -U unicharset code.font.exp0.tr
    mftraining -F font_properties -U unicharset -O code.unicharset code.font.exp0.tr
    cntraining code.font.exp0.tr

重命名 经过上面的步骤, 我们可以得到 code.unicharest normproto inttemp pffmtable shapetable 等文件, 将他们重名为下面的格式 [fontname].[name]. 如: code.normproto.

### 生成tessdata文件

    combine_tessdata code. 最后的这个.一定不能忘.

## 使用
将生成的tessdata文件移至tesseract-ocr的tessdata位置, 如果是ubuntu使用apt-get安装的， 则对应在/usr/share/tesseract-orc/中，

tesseract src.png output -l code+eng
