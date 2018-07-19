---
title: Linux命令_awk
tags: linux命令
grammar_cjkRuby: true
---

### 语法
	awk [选项参数] ‘script’ var=value file(s)
	或
	awk [选项参数]  -f scriptfile var=value file(s)
	
选项参数说明：
* -F fs， 指定输入文件分隔符，fs是一个字符串或者一个正则表达式，比如 -F:
* -v var=value，赋值一个用户定义变量
* -f scripfile, 从脚本文件中读取awk命令


### 用法
	用法一：
	awk '{ [parttern] action }' { filenames }  # 行匹配语句 awk ' ' 只能用单引号
	比如： awk '{ print $1, $4}' log.txt
	
	用法二：
	awk -F  # -F相当于内置变量FS,指定分割字符
	比如：awk -F, '{ print $1, $2}' log.txt
	 	awk -F '[ ,]' '{ print $1, $2}' log.txt #使用多个分隔符，先用空格分割，然后对分割结果再使用 ， 分割
		
	用法三：
	awk -v  # 设置变量
	比如：awk -v a=1 '{ print $1, $1+a }' log.txt
			awk -v a=1 -v b = 2'{ print $1, $1+a+b }' log.txt
			
	用法四：
	awk -f { awk脚本} {文件名}
	比如：awk -f cal.awk log.txt
		
		
### awk脚本
关于awk脚本，我们需要注意两个关键词BEGIN和END.
* BEGIN { 这里面放的是执行前的语句 }
* END { 这里面放的是处理完所有行后要执行的语句 }
* { 这里面放的是处理每一行是要执行的语句 }

	

	