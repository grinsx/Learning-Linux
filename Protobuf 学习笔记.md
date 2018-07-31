### Protobuf 学习笔记
Google Protocol Buffer(Protobuf)是一种轻便高效的结构化数据存储格式，平台无关、语言无关、可扩展，可用于**通信协议**和**数据存储**等领域。

在protobuf中，协议是由一系列的消息组成的。因此最重要的是如何定义通信时使用到的消息格式。

#### Protobuf消息定义
消息由至少一个字段组合而成，类似于C语言中的结构体。每个字段都有一定的格式。

字段格式：限定修饰符 1 | 数据类型 2 | 字段名称 3 | =  | 字段编码值 4 | [ 字段默认值 5]

1. 限定修饰符包括：required、optional、repeated
	* required：表示一个必须字段。相对于发送方，在发送消息之前必须设置该字段的值，对于接收方，必须能够识别该字段的意思。发送之前没有设置required字段或者无法识别required字段都会引发编解码异常，导致消息被丢弃。
	* optional：表示一个可选字段，可选对于发送方，在发送消息时，可以有选择性的设置或者不设置该字段的值。对于接收方，如果能够识别可选字段就进行相应的处理，如果无法识别，则忽略该字段，消息中的其它字段正常处理。**注意：**因为optional字段的特性，很多接口在升级版本中都把后来添加的字段都统一的设置为optional字段，这样老的版本无需升级程序也可以正常的与新的软件进行通信，只不过新的字段无法识别而已，因为并不是每个节点都需要新的功能，因此可以做到按需升级和平滑过渡。
	* repeated：表示该字段可以包含0 ~ N个元素。其特性和optional一样，但是每一次可以包含多个值。可以看做在传递一个数组。

2. 数据类型：Protobuf定义了一套基本数据类型，几乎都可以映射到C++、Java等语言的基础数据类型。

3. 字段名称：字段名称的命名与C、C++等语言的变量命名方式几乎相同。

4. 字段编码值：有了该值，同学双方才能互相识别对方的字段。当然，同样的编码值，其限定修饰符和数据类型必须相同。编码值的范围 1 ~ 2^32。
	* 1 ~ 15的编码时间和空间效率是最高的。编码值越大，其编码的时间和空间效率就越低(相对于1 ~ 15)。
	* 1900 ~ 2000编码为Google protobuf系统内部保留值，建议不要使用。

5. 默认值：当在传递数据时，对于required数据类型，如果用户没有设置值，则使用默认值传递到对端。

#### Protobuf示例
1. 首先定义一个test.proto文件，如下：
```cpp
message Info{
	optional string name = 1;
	optional int32 age = 2;
}
```

2. 定义proto文件之后需要用到protobuf提供的编译工具将proto文件编译成不同语言的源码
```cpp
protoc -I=./ --cpp_out=./ test.proto
```
将会生成两个文件，`test.pb.h`和`test.pb.cpp`

3. 使用方式如下：
```cpp
#include <stdio.h>
#include <iostream>
#include <string>
#include "test.pb.h"
using namespace std;
int main(int argc, char *argv[])
{
    Info *pinfo = new Info();
    pinfo->set_name("testname");
    pinfo->set_age(120);
    cout<<"info.name="<<pinfo->name()<<", age="<<pinfo->age()<<endl;

    delete pinfo;
    return 0;
}
```

#### Protobuf常用API
1. protoc为message的每个required字段和optional字段都定义了以下几个(不限于这几个):

```cpp
TypeName xxx() const;     //获取字段的值
bool has_xxx();	           //判断是否设值
void set_xxx(const TypeName&);     //设值
void clear_xxx();              //使其变为默认值
```	

2. 为每个repeted字段定义了以下几个：
```cpp
TypdeName* add_xxx();   //增加节点
TypeName xxx(int) const;  //获取指定序号的节点，类似于C++的[]运算符
TypeName *mutable_xxx(int);   //类似于上一个，但是获取的指针
int xxx_size();         //获取节点的数量
```

3. 下面是几个常用的序列化和反序列化函数：
```cpp
bool SerializeToOstream(std::ostream *output) const;  //输出到输出流中
bool SerializeToString(string *output) const;  //输出到string中
bool SerializeToArray(void *data, int size) const;  //输出到字节流

//对应的反序列化函数
bool ParseFromIstream(std::istream *input);    //从输入流解析
bool ParseFromString(const string &data);     //从string解析
bool ParseFromArray(const void *data, int size);   //从字节流解析
```

4. 其他常用函数：
```cpp
bool IsInitialized();   //检查是否所有required字段都被设值
size_t ByteSize() const;  //获取二进制字节序的大小
```

5. [官方API文档](https://developers.google.com/protocol-buffers/docs/reference/overview)

6. 编译生成可执行代码：编译格式和普通的C++代码一样，但是要加上 `-lprotobuf`和`-pthread`

#### Protobuf 数据类型
一个标量消息字段可以含有一个如下的类型——该表格展示了定义于.proto文件中的类型，以及与之对应的、在自动生成的访问类中定义的类型：

![protobuf数据类型](https://raw.githubusercontent.com/lengender/MarkdownPhotos/master/protobuf%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B.png)