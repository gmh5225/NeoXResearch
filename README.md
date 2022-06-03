#### 确认是否是neox引擎
解包看是否有大量NeoX相关

#### 确认CPython版本
NeoX魔改了CPython，确认NeoX的CPython版本，以便参考源码。  

CPython有很多调试字符串，里面记录了源文件路径，我们可以借此得到python主次版本号。  
IDA Pro打开核心so库，开Strings窗口，搜python    
![image](https://user-images.githubusercontent.com/21135715/171880588-df7fbaec-d307-443d-925f-c8458eac20d5.png)  

CPython定义了PY_VERSION字符串，格式是MAJOR.MINOR.PATCH，我们再次搜索2.7.  
![image](https://user-images.githubusercontent.com/21135715/171880752-0c2db9c7-5b09-439a-b60b-d40bc072a415.png)  
可得到完整版本号为2.7.3

#### 获得opcode映射表
由前人工作，可知NeoX修改了CPython的opcode定义。可以看反汇编代码，也可以对比魔改的Python产生的pyc与原版，得到映射关系。  
这里原本试图找到其他更简洁的方法，于是翻CPython源码，发现有个opcode模块，可以直接打印opcode表。但是NeoX是Embed Python，opcode是py module，没有内置，无法访问。  
也没有想到其他的好办法，最后还是对比pyc，拿到了opcode映射表。  

#### 修复无法识别的opcode
按前人记录，拿到opcode映射表以后就可以解码pyc了。实测后发现，NeoX又额外引入了一些opcode，这些opcode的作用与已有的不同，opcode映射表无法解决这些新opcde。  
由CPython源码，找到CPython解释执行opcode的实际函数PyEval_EvalFrameEx，对比源码与反汇编代码，得到这些新opcode的作用，基本就是旧opcode的组合。
修复opcode时按规则将新opcode换成旧opcode的组合即可。  

参考  
[neox-tools](https://github.com/xforce/neox-tools)  
[NeteaseUnpackTools](https://github.com/yuanbi/NeteaseUnpackTools)  
[unnpk](https://github.com/YJBeetle/unnpk)

基础流程没有变  
不过pyc混淆不止重映射opcode了，还加入了新的opcode  
需要根据含义将新引入的opcode改成常规opcode  
