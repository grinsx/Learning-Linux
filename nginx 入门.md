### nginx 入门
nginx 有一个主进程和其他子进程。主进程的主要工作是加载和执行配置文件，并且驻留子进程。子进程用来作为实际的请求处理。

nginx采取基于事件的模型和OS依赖的机制，在多个子进程之间的高效的分配请求，子进程的个数会直接写在配置文件中，并且对于给定的配置可以是固定的，或者根据可用的CPU核数自动的进行调整。

nginx和它模块的工作方式在配置文件中写好的。默认情况下，这个配置文件通常命名为nginx.conf，并且会放置在/usr/local/nginx/conf， /etc/nginx，或者 /usr/local/etc/nginx

#### 启用、停止和重载配置
运行可执行文件就可以开启nginx，比如：
```cpp
// -c 为 nginx的配置文件
nginx -c /usr/local/nginx/conf/nginx.conf
```
如果nginx已经开启，那么它可以通过使用 -s 参数的可执行命令控制，使用以下格式：
```cpp
nginx -s signal
```
signal可以为下列命令之一：
* stop ：直接关闭nginx
* quit ：会在处理完当前请求后退出，也叫优雅关闭
* reload：重新加载配置文件，相当于重启
* reopen：重新打开日志文件

比如，等待当前子进程处理完正在执行的请求后，结束nginx进程，可以使用下列命令：
```cpp
nginx -s quit
```
执行该命令的用户需要和启动的nginx的用户一致。如果重载配置文件的命令没有传递给nginx或者nginx没有重启，那么配置文件的改动是不会被使用的。重载配置文件的命令可以使用：
```cpp
nginx -s reload
```
一旦主进程接收到重载配置文件的命令后，它会先检查配置文件语法的合法性，如果没有错误，则会重新加载配置文件。如果成功，则主进程会重新创建一个子进程并且发送关闭请求给以前的子进程。如果没有成功，会停止接收新的请求并且继续处理当前请求，直到处理完毕。之后，该子进程就直接退出了。

在Unix工具的帮助下，比如使用kill工具，该信号会被发送给nginx进程。在这种情况下，信号会被直接发送给带有进程ID的进程。nginx的主进程的进程ID是写死在nginx.pid文件中的。该文件通常放在/usr/local/nginx/logs或者/var/run目录下。比如，如果主进程的ID是1628，为了发送QUIT信号来使nginx优雅退出，可以执行：kill -s QUIT 1628

#### 配置文件结构
nginx是由一些模块组成，我们一般在配置文件中使用一些具体的指令来控制它们。指令被分为简单指令和块级指令。

一个简单指令是由名字和参数组成，中间用空格分开，并且以分号结尾。例如：
```cpp
//简单指令
root /data/www;
```
块级指令和简单指令一样有着类似的结构，但是末尾不是分号，而是{}大括号包裹的额外指令集。如果一个块级指令的大括号有其他指令，则它被叫做一个上下文。在配置文件中，没有放在任何上下文中的指令则是处在主上下文中，events和http的指令是放在主上下文中，server放在http中，location放在server中。
以#开头的行，会被当做注释。
```cpp
# this is a comment
events {
	worker_connections 4096;  ## default 1024
}

http {
	server {
		listen 80;
		server_name domain1.com www.domain1.com;
		access_log logs/domain1.access.log main;
		root html;
	
		location ~ \.php$ {
			fastcgi_pass 127.0.0.1:1025;
		}
	}
}
```

#### 静态服务器
一个重要的网络服务器的任务是处理文件(比如图片或者静态html文件)。这里，你会实践一个例子，文件会从不同的目录中映射(取决于请求)：/data/www (放置html文件) 和 /data/images(放置图片)。这里需要配置一个文件，将带有两个location的指令的server的块级命令放置在server指令中。

首先，创建一个/data/www的目录，然后放置一个事先写好内容的index.html文件，接着，创建一个/data/images目录，然后放置一些图片。下一个，打开配置文件，默认的配置文件已经包含了一些关于server指令的样式，大多数情况下直接把他们给注释掉，现在注释掉其他的区块，然后写一个写的server区块：
```cpp
http {
	server {

	}
}
```
通常，该配置文件可能会包含多个server指令，这些server指令监听不同的端口和服务器名。一旦nginx决定哪个服务进程处理请求，它会根据在server块级指令中定义好的location指令的参数，来匹配请求头中的指定的URI，将下列的location指令添加到server指令中：
```cpp
location / {
	root /data/www;
}
```
该location指令相对于请求中的URI执行了"/"的前缀，为了匹配请求，URI会被添加到root命令指令的路径后，即/data/www，得到贝蒂文件系统中请求文件的路径。如果，有几个location匹配到，那么nginx会选择最长的前缀。上面location提供了长度为1的前缀，所以，仅当其他的location匹配失败后，该指令才会使用。接着，添加第二个location区块：
```cpp
location /images/ {
	root /data;
}
```
它会匹配到以/images/开头的请求 (location / 也会匹配到该请求，只是前缀更短) 。 server块级命令的配置结果如下：
```cpp
server {
	location / {
		root /data/www;
	}

	location /images/ {
		root /data;
	}
}
```
这已经是一个可用的服务器配置，它监听标准的80端口并且可以在本地上通过http://localhost/访问。对于URI以/images/开头的请求，服务器会从/data/images目录中，返回对应的文件。例如，nginx会返回/data/images/example.png文件，当接收到http://localhost/images/example.png的请求响应时。如果文件不存在，nginx会返回一个404错误的响应，没有以/images/开头的URI请求，将会直接映射到/data/www目录中。

为了使用新的配置文件，如果还没有开启nginx需要先开启，然后将重装信号发送给nginx的主进程，通过执行：
```cpp
nginx -s reload
```
如果你发现有些地方出了问题，你可以在/usr/local/nginx/logs或者/var/log/nginx目录下的access.log和error.log文件中，找到原因。

#### 搭建一个简易的代理服务
nginx常常用来作为代理服务器，这代表着服务器接收请求，然后将它们传递给被代理的服务器，得到请求的响应，再将它们发送给客户端。我们将配置一个基本的代理服务器，它会处理本地图片文件的请求并返回其他的请求给被代理的服务器。在这个例子中，两个服务器都会定义在一个nginx实例中。首先，通过在nginx配置文件中添加另一个server的实例，来定义一个被代理的服务器，像以下的配置：
```cpp
server {
	listen 8080;
	root /data/up1;

	location / {
	}
}
```
上面就是一个简单的服务器，它监听8080端口(之前，listen并没有被定义，是因为默认监听的80端口)，并且会映射所有的请求给本地文件目录/data/up1。创建该目录，然后添加index.html文件。注意，root指令是放在server上下文中。当响应请求的location区块中，没有自己的root指令，上述root指令才会被使用。

接着，使用前面章节的server配置，然后将它改为一个代理服务器配置。在第一个location区块中，放置已经添加被代理服务器的协议，名字和端口等参数的proxy_pass指令：
```cpp
server {
	location / {
		proxy_pass http://localhost:8080;
	}

	location /images/ {
		root /data;
	}
}
```
我们将修改第二个区块，使它返回一些典型后缀的图片文件请求，现在它只会映射带有/images/前缀请求到/data/images目录下，修改后的location指令如下：
```cpp
location ~ \.(gif|jpg|png)$ {
	root /data/images;
}
```
 该参数是一个正则表达式，它会匹配所有以.gif，.jpg或者.png结尾的URI。一个正则表达式需要以 ~ 开头。匹配到的请求会被映射到/data/images目录下。当nginx在选择location去响应一个请求时，它会先检测带有前缀的location指令，记住先检测带有最长前缀的location，然后检测正则表达式。如果有一个正则的匹配的规则，nginx会选择该location，否则，会选择之前缓存的规则。