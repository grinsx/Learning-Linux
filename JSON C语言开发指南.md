### JSON C语言开发指南

原贴见：[json-c开发指南](https://www.cnblogs.com/qingergege/p/5997762.html)

#### 1. Json简介
JSON(JavaScript Object Notation) 是一种轻量级的数据交换格式。易于人阅读和编写。同时也易于机器解析和生成。它基于JavaScript Programming Language, Standard ECMA-262 3rd Edition - December 1999的一个子集。 JSON采用完全独立于语言的文本格式，但是也使用了类似于C语言家族的习惯（包括C, C++, C#, Java, JavaScript, Perl, Python等）。这些特性使JSON成为理想的数据交换语言。

跟XML相比，JSON的优势在于格式简洁短小，特别是在处理大量复杂数据的时候，这个优势便显得非常突出。从各浏览器的支持来看，JSON解决了因不同浏览器对XML DOM解析方式不同而引起的问题。

##### Json建构于两种结构
* “名称/值”对的集合（A collection of name/value pairs）。不同的语言中，它被理解为对象（object），纪录（record），结构（struct），字典（dictionary），哈希表（hash table），有键列表（keyed list），或者关联数组 （associative array）。
* 值的有序列表（An ordered list of values）。在大部分语言中，它被理解为数组（array）。

这些都是常见的数据结构。事实上大部分现代计算机语言都以某种形式支持它们。这使得一种数据格式在同样基于这些结构的编程语言之间交换成为可能。

##### Json具有以下这些形式：
* 对象是一个无序的“‘名称/值’对”集合。一个对象以“{”（左括号）开始，“}”（右括号）结束。每个“名称”后跟一个“:”（冒号）；“‘名称/值’ 对”之间使用“,”（逗号）分隔。
* 数组是值（value）的有序集合。一个数组以“[”（左中括号）开始，“]”（右中括号）结束。值之间使用“,”（逗号）分隔。
* 值（value）可以是双引号括起来的字符串（string）、数值(number)、true、false、 null、对象（object）或者数组（array）。这些结构可以嵌套。
* 字符串（string）是由双引号包围的任意数量Unicode字符的集合，使用反斜线转义。一个字符（character）即一个单独的字符串（character string）。字符串（string）与C或者Java的字符串非常相似。
* 数值（number）也与C或者Java的数值非常相似。除去未曾使用的八进制与十六进制格式。除去一些编码细节。

#### 2. Json库函数的使用
##### JSON对象的生成
* Json对象的类型：

```cpp
json_type_object, "名称/值"对的集合
```
* Json对象的值类型

```cpp
json_type_boolean,
json_type_double,
json_type_int,
json_type_array, "值的集合"
json_type_string
```
*  常用库函数

```cpp
1. 创建一个空的json_type_object类型的JSON对象
	struct json_object* json_object_new_object();

2. 创建一个json_type_boolean值类型的json对象
	struct json_object* json_object_new_boolean(Boolean b);

3. 从json对象中boolean值类型得到boolean值
	Boolean json_obejct_get_boolean(struct json_object* obj);

	以此类推：
	struct json_object* json_object_new_int(int i);
	int json_object_get_int(struct json_object *this);
	struct json_object* json_object_new_double(double d);
	double json_object_get_double(struct json_object *this);
	struct json_object* json_object_new_string(char *s);
	char* json_object_get_string(struct json_object *this);

4. 创建一个空的json_type_array类型json数组值对象
	struct json_object* json_object_new_array();

5. 由str里的json字符串生成JSON对象，str是json_object_to_json_string()生成的
	参数： str - json字符串
	struct json_object *json_tokener_parse(char *str);

6. 从json中按名字取出一个对象，
	参数： json - json对象，  name - jason域名字
	struct json_object* json_object_object_get(struct json_object *json, char *name);
```

##### JSON对象的释放
* 增加引用计数
```cpp
struct json_object *json_object_get(struct json_object *obj);
```
说明：增加对象引用计数。使用c库最关心的是内存谁来分配, 谁来释放。 jsonc的内存管理方式, 是基于引用计数的内存树(链)，如果把一个struct json_object 对象a，add到另一个对象b上，就不用显式的释放(json_object_put) a了，相当于把a挂到了b的对象树上，释放b的时候，就会释放a。
当a既add到b上，又add到对象c上时会导致a被释放两次(double free)，这时可以增加a的引用计数(调用函数json_object_get(a))，这时如果先释放b，后释放c，当释放b时， 并不会真正的释放a，而是减少a的引用计数为1，然后释放c时，才真正释放a。

* 减少引用计数
```cpp
void json_object_put(struct json_object *obj);
```
说明：减少对象引用计数一次，当减少到0就释放(free)资源。

* 样例片段
```cpp
my_string = json_object_new_string("\t");
	
/*输出 my_string=   */ \t(table键)
printf("my_string=%s\n", json_object_get_string(my_string));
	
/*转换json格式字符串 输出my_string.to_string()="\t"*/
printf("my_string.to_string()=%s\n", json_object_to_json_string(my_string));

/*释放资源*/
json_object_put(my_string);
```

##### JSON对象的操作
```cpp
1. 检查json_object是json的某个类型
	参数：obj - json_object实例， 
		type -  json_type_boolean,json_type_double, json_type_int, json_type_object, json_type_array, json_type_string
	int json_object_is_type(struct json_object *obj, enum json_type type);

2. 得到json_object的类型
	enum json_type json_object_get_type(struct json_object *obj);

3. 将json_object内容转换成json格式字符串，其中可能含有转义符
	char *json_object_to_json_string(struct json_object *obj);

4. 添加一个对象域到json对象中
	参数：obj - json对象， key - 域名字， val - json值对象
	void json_object_object_add(struct json_object *obj, char *key, struct json_object *val);

5. 删除key值json对象
	void json_object_object_del(struct json_object *obj, char *key);

6. 得到json对象数组的长度
	int json_object_array_length(struct json_object *obj);

7. 在json数组末端添加一个元素
	int json_object_array_add(struct json_object *obj, struct json_object *val);

8. 在指定的json对象数组下标插入或替换一个json对象元素
	int json_obejct_array_put_idx(struct json_object *obj, int idx, struct json_object *val);

9. 从数组中，按下标取json值对象
	struct json_object * json_object_array_get_idx(struct json_object* json_array, int idx);

10. 遍历json对象的key和值(key, val默认参数不变)
	定义宏 json_object_object_foreach(obj, key, val)
```

