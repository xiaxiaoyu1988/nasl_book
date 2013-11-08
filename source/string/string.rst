字符串处理函数
===============================================

NASL可以像处理数字一样处理字符串。所以你可以安心的使用==, < , >操作符。

例子:

.. code-block:: python

	a = "version 1.2.3";
	b = "version 1.4.1";
	
	if(a < b)
	{
		#
		# Will be executed, since version 1.2.3 is lower
		# than version 1.4.1
	}
       
 	c = "version 1.2.3";
 
	if(a==c) 
	{
		# Will also be evaluated
	}

也可以获取字符串的某个字符，类似C:

.. code-block:: python

 	a = "test";
 	b = a[1];  # b equals to "e"

也可以加减两个字符串:

.. code-block:: python
	
 	a = "version 1.2.3";
 	b = a - "version ";   # b equals "1.2.3"
 
 	a = "this is a test";
 	b = " is a ";
 	c = a - b;            # c equals to "this test"
 
 	a = "test";
 	a = a+a;              # a equals to "testtest"


除此之外，还有><操作符。NASL定义了很多函数去构造和修改字符串。


.. toctree::
   :maxdepth: 2

.. _String:

处理正则表达式的ereg()函数
-----------------------------------------------

模式匹配通过ereg()函数完成。语法如下:

.. code-block:: python

	result = ereg(pattern:<pattern>, string:<string>)

正则匹配是egrep风格。细节请参考 `egrep <http://unixhelp.ed.ac.uk/CGI/man-cgi?egrep>`_ 手册.。

例子:

.. code-block:: python

	if(ereg(pattern:".*", string:"test"))
	{
	  display("Always executed\n");
	}
	
	mystring = recv(socket:soc, length:1024);
	if(ereg(pattern: "SSH-.*-1\..*", string : mystring))
	{
		display("SSH 1.x is running on this host");
	}
	


egrep()函数
-----------------------------------------------

egrep()函数返回多行文本中匹配的第一行。当用于单行文本时，作用类似ereg()函数。如果没有匹配项，则返回FALSE。语法如下:

.. code-block:: python

	str = egrep(pattern : <pattern>, string: <string>)

例子:

.. code-block:: python

	soc = open_soc_tcp(80);
	str = string("HEAD / HTTP/1.0\r\n\r\n");
	send(socket:soc, data:str);
	
	r = recv(socket:soc, length:1024);
	server = egrep(pattern:"^Server.*", string : r);
	
	if(server)display(server);



crap()函数
-----------------------------------------------

crap()函数在测试缓冲区溢出很方便。它的语法如下：
 - crap(<length>)
 - crap(length:<length>, data:<data>)

例子:

.. code-block:: python

	a = crap(5);				# a = "XXXXX";
	b = crap(4096);				# b = "XXXX...XXXX" (4096 X's)
	c = crap(length:12, data:"hello");	# c = "hellohellohe" (length: 12);


string()函数
-----------------------------------------------

该函数用于从字符或者字符串中产生一个新的字符串。
语法为：

.. code-block:: python

	string(<string1>, [<string2>, ..., <stringN>])

该函数会转义特殊字符，例如\n 或者 \t。

例子：

.. code-block:: python

	name = "Renaud";
    
	a = string("Hello, I am ", name, "\n");		# a equals to "Hello, I am Renaud" 
							# (with a new line at the end)
	b = string(1, " and ", 2, " makes ", 1+2);	# b equals to "1 and 2 makes 3"
	c = string("MKD ", crap(4096), "\r\n");		# c equals to "MKD XXXXX.....XXXX"
							# (4096 X's) followed by a carriage
							# return and a new line


strlen()函数
-----------------------------------------------

返回字符串的长度。

例子:

.. code-block:: python

	a = strlen("abcd"); # a is equal to 4 



raw_string()函数
-----------------------------------------------

整数转换成字符串。

例子：

.. code-block:: python

	a = raw_string(80, 81, 82); # a equals to 'PQR'



strtoint()函数
-----------------------------------------------

该函数用于将一个NASL整数转换成二进制整数。语法为：

.. code-block:: python

	value = strtoint(number:<nasl_integer>, size:<number_of_bytes>);

这个函数适合同raw_string()配合使用。size参数是NASL整形的字节数，必填参数，可以是1,2,4。


tolower()函数
-----------------------------------------------

该函数用于将字符串转换成小写。语法是:

.. code-block:: python

	tolower(<string>)

返回值是参数<string>的小写字符串。

例子:

.. code-block:: python

	a = "Hello";
	b = tolower(a); # b equals to "hello"

