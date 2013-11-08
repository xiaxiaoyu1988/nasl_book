基础：NASL语法
===============================================

NASL的语法同C非常类似，除了移除掉一些C中令人困扰的东西。你不用关心你的变量的类型，也不用关心内存的申请和释放。在使用变量之前也不需要声明。你只需要专注于你的安全测试。

如果你不知道C，那么此篇为C程序员写的教程阅读起来可能会比较困难。不要抱怨，这本手册将来更具可读性。

.. toctree::
   :maxdepth: 2

注释
-----------------------------------------------

注释字符是“#”。它仅能注释当前行。
例子：

合法的注释：

.. code-block:: python
    
	a = 1; #let a = 1
	#Set b to 2:
	b = 2;
	
非法的注释：

.. code-block:: python

    #
       set a to 1:
                    #
    a = 1;
	a = # set a to 1 #1;
	
变量，类型，内存分配，包含(include)
-----------------------------------------------

你不要在使用变量之前声明它， 也不需要关心变量类型。NASL解释器在你尝试写一些不符合语法的代码时提示你， 比如把一个IP报文和一个整数相加。你也不需要关心内存分配和包含。NASL中没有包含(从目前最新的nasl脚本来看，还是有包含的:( )，而且内存在需要时会自动分配。

数字和字符串
-----------------------------------------------

可以使用三种数制类型：十进制，十六进制，以及二进制。

例如：

.. code-block:: python

	a = 1024;
	b = 0x0A;
	c = 0b001010110110;
	d = 123 + 0xFF;

字符串必须用引号括起来。和C不一样。NASL不会去解析特殊字符，除非使用string()函数。

例如：

.. code-block:: python

	a = "Hello\nI'm Renaud";           # a equals to "Hello\nI'm Renaud"
 	b =  string("Hello\nI'm Renaud");  # b equals to "Hello
                                       #              I'm renaud"  	

 	c = string(a);                     # c equals to b

string()函数将在 :ref:`字符串操作 <String>` 中解释。


命名和匿名参数
-----------------------------------------------

命名函数
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

在处理函数参数上，NASL同C有一个不一样的地方。在C中， 你必须知道每一个参数的位置和顺序。当一个函数的参数多达10个以上的时候，你就头疼了。例如，一个组装IP报文的函数，需要茫茫多的参数，如果你要使用它，就需要通过阅读文档来记住参数们的位置和顺序。这太浪费时间了，所以NASL尝试避免这种情况。

所以，当函数参数有不同的类型，且参数顺序很重要的时候， 就要定义一个命名函数了。你必须给每一个参数命名。当你忘记其中一些参数时，在运行时，NASL会给出一些提示。

例如：

forge_ip_packet()函数有很挫的参数。下面这两个调用方式都是对的，且功能是一样的：

.. code-block:: python

	forge_ip_packet(ip_hl : 5, ip_v : 4, 
			ip_p : IPPROTO_TCP);
			
	forge_ip_packet(ip_p : IPPROTO_TCP,
			ip_v : 4, ip_hl : 5);

在运行时，用户会被提示缺少参数(ip_len等等)。当然安全测试脚本不能和用户交互，但是方便快速编码和调试。

匿名函数
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

匿名函数是只有一个参数或者多个相同类型的参数的函数。

例如：

.. code-block:: python

	send_packet(my_packet);
	send_packet(packet1, packet2, packet3);

这些函数可能会有一些可选项。比如，send_packet()函数在等待目标的响应。如果你觉得没必要等待，可以停用等待加快测试速度:

.. code-block:: python

	send_packet(packet, use_pcap:FALSE);


For和While
-----------------------------------------------

for和while跟c中类似。

for:

.. code-block:: python

	for(instruction_start;condition;end_loop_instruction)
	{
		#
		# Some instructions here 
	 	#
	}

或者

.. code-block:: python

	for(instruction_start;condition;end_loop_instruction)function();

while:

.. code-block:: python

	while(condition)
	{
		#
		# Some instructions here
		#	
	}

或者

.. code-block:: python

	while(condition)function();
	
例子:

.. code-block:: python

	# Count from 1 to 10
	for(i=1;i<=10;i=i+1)
		display("i : ", i, "\n");
	
	# Count from 1 to 9, and say the type
	# of each number (even or odd)
	for(j=1;j<10;j=j+1)
	{
		if(j & 1)
			display(j, " is odd\n");
		else 
			display(j, " is even\n");		
	}

	# Do something completely useless :
		
	i = 0;
	while(i < 10)
	{
	 	i = i+1;
	}


自定义函数
-----------------------------------------------

NASL现在已经支持自定义函数。一个自定义函数可以像这样定义：

.. code-block:: python

	function my_function(argument1, argument2, ....)

自定义函数必须使用命名参数。NASL可以处理递归调用。

例如：

.. code-block:: python

	function fact(n)
	{
  		if((n == 0)||(n == 1))
			return(n);
  		else
			return(n*fact(n:n-1));
	}

	display("5! is ", fact(n:5), "\n");

自定义函数中不能包含其他的自定义函数(实际上是可以的，但在这种情况下，NASL解释器会告警)。
注意，如果你想要你的函数返回值(大部分函数的目的), 那么你需要使用return()函数。 因此，return必须使用括号。

下面是一个错误的用法：

.. code-block:: python

	function func()
	{
   		return 1; # parenthesis are missing here !
	}


操作符
-----------------------------------------------

标准C中的一些操作符，同样可以用在NASL中，例如: +, -, \*, / 以及 %。 目前操作符优先级还不支持，但是以后会更新。除此之外，还支持二进制操作符 | 和 &。另外还有两个C中没有的操作符。


'x'操作符
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

for和while都很实用。但在一些情况下会有性能损失，因为每次迭代中，条件都需要计算。比如你想发送一个SYN风暴或者别的，就比较麻烦了。'x'操作符可以重复相同的函数N次，而且非常快(跟C的速度差不多)。

例如：

.. code-block:: python

	send_packet(udp) x 10;

将会发送相同的udp包10次。


'><'操作符
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

'><' 是一个布尔操作符。如果A字符被包含在B字符中，则返回true。

例如：

.. code-block:: python

	a = "Nessus";
	b = "I use Nessus";
	
	if(a >< b)
	{
		# This will be executed since
		# a is in B
		display(a, " is contained in ", b, "\n");
	}
