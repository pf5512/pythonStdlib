在网络通信中，数据基于二进制流传输，发送端对int,char等基本数据需要某种规则转换成二进制流的字符串，接收端也需要某种机制逆转二进制数据流还原真正的数据。这就需要`struct`模块。

其主要方法和属性是：

- pack(), 打包
- unpack(), 解包
- calcsize(fmt), 计算给定的格式(fmt)占用多少字节的内存
- Struct类 format
- size，字节大小
- format, 格式
- pack()和unpack()都是在内存中分配资源，对于大文件则耗内存，可以通过利用buffer(通过`ctypes.create_string_buffer(size)`)，使用`pack_into`和`unpack_from`方法对一个已经提前分配好的buffer进行字节的填充，而不会每次都产生一个新对象对字节进行存储。

format格式很多，还需要考虑**字节顺序**，如果在format没用指出，则字节序按系统设定。

参考:[**浅析Python中的struct模块**](http://www.cnblogs.com/coser/archive/2011/12/17/2291160.html)

## 1.进制转换
    
    print bin(10)       # decimal to binary
    print hex(10)       # decimal to hex
    print oct(10)       # decimal to octal
    
    print int('0b1010', 2)      # binary to decimal
    print int('0xa', 16)        # hex to decimal
    print int('012', 8)         # octal to decimal
    
    hex_num = 0xa
    binary_num = 0b1010
    print bin(hex_num)              # hex to binary
    print hex(binary_num)           # bin to hex
    

## 2.进制处理

同正则表达式，**格式指示符由字符串转换为一种编译表示，这种转换是消耗资源的**，所以`Struct`类实例调用相应的方法完成一次转换会更加高效。

所以有两种形式

	# 模块级函数
	import struct
	values = (1, 'ab', 2.4)
	s = struct.pack('I2sf', *values)

	# Struct类实例
	ss = struct.Struct('I2sf')
	bs = ss.pack(*values)

**格式指示符包含数据类型，可选数量以及字节序** 如下所示：


![](http://7fvf56.com1.z0.glb.clouddn.com/struct_docs.png)

注意：

- q和Q只在机器支持64位操作时有意思;
- 每个格式前可以有一个数字，表示个数
- s格式表示一定长度的字符串，4s表示长度为4的字符串，但是p表示的是pascal字符串
- P用来转换一个指针，其长度和机器字长相关; 注5.最后一个可以用来表示指针类型的，占4个字节

字节序

![](http://7fvf56.com1.z0.glb.clouddn.com/struct_docs2.png)

在[Python使用struct处理二进制](http://www.cnblogs.com/gala/archive/2011/09/22/2184801.html)中，关于二进制和普通文件读写的区别：


我们使用处理二进制文件时，需要用如下方法

	binfile=open(filepath,'rb')    读二进制文件

	binfile=open(filepath,'wb')    写二进制文件

那么和`binfile=open(filepath, 'r')`的结果到底有何不同呢？

不同之处有两个地方：

- 第一，使用'r'的时候如果碰到'0x1A'，就会视为文件结束，这就是EOF。使用'rb'则不存在这个问题。即，如果你用二进制写入再用文本读出的话，如果其中存在'0X1A'，就只会读出文件的一部分。使用'rb'的时候会一直读到文件末尾。

- 第二，对于字符串`x='abc\ndef'`，我们可用`len(x)`得到它的长度为7，`\n`我们称之为换行符，实际上是`'0X0A'`。当我们用`'w'`即文本方式写的时候，在windows平台上会自动将`'0X0A'`变成两个字符`'0X0D'`，`'0X0A'`，即文件长度实际上变成8。当用`'r'`文本方式读取时，又自动的转换成原来的换行符。如果换成`'wb'`二进制方式来写的话，则会保持一个字符不变，读取时也是原样读取。所以如果用文本方式写入，用二进制方式读取的话，就要考虑这多出的一个字节了。`'0X0D'`又称回车符。linux下不会变。因为linux只使用`'0X0A'`来表示换行。
