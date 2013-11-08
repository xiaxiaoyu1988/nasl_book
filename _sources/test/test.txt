编写一个Nessus安全测试脚本
===============================================
.. toctree::
   :maxdepth: 2

如何编写一个高效的Nessus测试脚本
-----------------------------------------------

仅需要很短的时间，nessusd会加载所有的脚本。所以写一个好的脚本一定要用到其他脚本的测试结果。
比如，一个脚本在尝试连接FTP服务器时，应该首先检查远程端口21是否开放。这将节省时间已经带宽，也大大加快了对于目的主机防火墙屏蔽21端口的情况。

探测一个端口是否开放
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

get_port_state(<portnum>)函数在端口开放时返回TRUE，反之返回FALSE。如果该端口未被扫描，同样返回TRUE。这时候端口的状态是未知的。
这个函数仅占用很少的CPU，在需要的情况下，应该尽可能的调用它。


知识库
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

nesssusd为每个主机都维护着一个知识库( :ref:`KB <KB>` ), 它包含了一次测试过程中，所有脚本测试产生的信息。安全测试脚本从中读取内容，也会将获取的信息返回给知识库。端口的状态，实际上被记在知识库的某个地方。

知识库被分为好几类。"Services"类包含了所有已经服务的端口。比如，Services/smtp 的值很可能是25.
然而，如果远程主机有个隐藏的SMTP服务配置在2500端口而不是25端口上，那么该条记录的值会更新为2500。

看 :ref:`附录 <KB>` 可以了解更多知识库的细节。

基本上，有两个函数可以操作知识库。
 - get_kb_item(<name>)用于获取<name>记录的值，这是个匿名参数。
 - set_kb_item(name:<name>, value:<value>)用户设置一个新纪录的值。

注意：你不能马上读取你刚刚设置的值。例如，以下代码可能不会有想象中的结果:

.. code-block:: python

	set_kb_item(name:"attack", value:TRUE);
	if(get_kb_item("attack"))
	{
		# Perform the attack - will not be executed
		# because our local KB has not been updated
	}

原因是因为安全考虑以及代码隔离。实际上，nessusd为每个新起的脚本拷贝一份知识库，而不是原始知识库，set_kb_item()函数会将值设置到原始知识库中，在当前正在跑的脚本中，nessusd是不会去更新它的拷贝知识库的。(*这点很重要*)

NASL脚本结构
-----------------------------------------------

每个NASL脚本都必须向Nessus服务端注册自己。注册包括，脚本名称，描述，作者等等。
这样，NASL脚本可能会是像下面这样的结构。

.. code-block:: python

	#
	# Nasl script to be used with nessusd
	#

	if(description)
	{
		# register information here...
		#
		# I will call this section the 'register' 
		# section
		#
		exit(0);
	}
	#
	# Script code here. I will call this section the
	# 'attack' section.
	#

description是一个全局变量，它会被设置成TRUE或者FALSE，基于该脚本必须还是非必须注册。


注册部分
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

注册部分必须调用以下方法:
 - script_name(language1:<name>, [...]) 设置脚本名称，将会在客户端显示。
 - script_description(language1:<desc>, [...]) 设置描述信息，用户点击脚本名称时显示。
 - script_summary(language1:<summary>, [...]) 设置脚本摘要，将会在提示信息中显示。
 - script_category(<category>) 设置脚本分类。必须是下列的其中之一。
 	- ACT_GATHER_INFO 脚本会在最开始被加载。这对远程主机无害。
 	- ACT_ATTACK	  脚本会尝试去获取远程主机上的某些权限。可能会影响远程主机(比如在测试缓冲区溢出的时候)。
 	- ACT_DENIAL	  尝试让远程主机宕机。
 	- ACT_SCANNER	  端口扫描脚本。
 - script_copyright(language1:<copyright>, [...]) 设置脚本版权信息。可以是你的名字或者其他任何东西。
 - script_family(language1:<family>, [...]) 设置脚本的族。这没有很明显的分类，你可以注册成"Joe's PowerTools" ，但是我不提倡这样。当前已经有个族有：
 	- Backdoors
	- CGI abuses
	- Denial of Service
	- FTP
	- Finger abuses
	- Firewalls
	- Gain a shell remotely
	- Gain root remotely
	- Misc.
	- NIS
	- RPC
	- Remote file access
	- SMTP problems
	- Useless services

你可能已经注意到了，这些函数大部分由language1参数。实际上，并不是这样的。

NASL提供了多语言支持。每个脚本必须至少支持英语。这些函数准确的语法应该是这样：

.. code-block:: python

	script_function(english:english_text, [francais:french_text, 
                                        deutsch:german_text,
					...]);

除此者外，script_dependencies()函数也可能被调用。它的作用是告诉nessusd，当前脚本是在某些其他脚本之后加载。这很有用，尤其你想要使用其他规则的测试结果时。语法是：

.. code-block:: python

	script_dependencies(filename1 [,filename2, ..., filenameN]);

filename参数就是脚本在磁盘上存储的名称。


攻击部分
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

攻击部分可能包含任何你认为对攻击测试有用的代码。
一旦你的攻击完成，你可以通过security_warning()或者security_hole()函数来打印报告。
security_warning()用于攻击成功，但并不是一个很严重的漏洞，意指这个漏洞并不会导致攻击者立马获取访问权限。
这两个函数的语法如下:

.. code-block:: python

	security_warning(<port> [, protocol:<proto>]);
	security_hole(<port> [, protocol:<proto>]);

	security_warning(port:<port>, data:<data> [, protocol:<proto>]);
	security_hole(port:<port>, data:<data> [, protocol:<proto>]);

第一种情况，客户端显示的是script_description()函数定义的脚本描述信息。由于多语言支持的存在，这很方便。
第二种情况，客户端显示的是data参数的内容。这同样很方便，比如必须显示版本信息的时候。


CVE兼容
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

CVE是一个试图解决所有安全漏洞之间关系的产品。访问 `cve.mitre.org <http://cve.mitre.org>`_ 获取更多信息。

Nessus是完全CVE兼容的。如果你正在为一个CVE定义过的漏洞编写脚本，请在描述部分调用script_cve_id()函数，用法如下:

.. code-block:: python

	script_cve_id(string);

例子:

.. code-block:: python

	script_cve_id("CVE-1999-0991");

相对与仅仅在报告中打印出CVE编号，单独调用这个函数非常重要，Nessus客户端会积极的使用这个函数带来的信息。

一个例子
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

除了安全测试， NASL也可以做一些维护工作。比如下面这个例子，该脚本会取人哪些主机运行着SSH，哪些主机没有运行SSH。

.. code-block:: python
	
	#
	# Check for ssh
	#	
	if(description)
	{
		script_name(english:"Ensure the presence of ssh");
		script_description(english:"This script makes sure that ssh is running");	
		script_summary(english:"connects on remote tcp port 22");
		script_category(ACT_GATHER_INFO);
		script_family(english:"Administration toolbox");
		script_copyright(english:"This script was written by Joe U.");
		script_dependencies("find_service.nes");
		exit(0);
	}
	#
	# First, ssh may run on another port. 
	# That's why we rely on the plugin 'find_service'
	#
	port = get_kb_item("Services/ssh");
	if(!port)
		port = 22;
	# declare that ssh is not installed yet
	ok = 0;
	if(get_port_state(port))
	{
		soc = open_sock_tcp(port);
		if(soc)
		{
			# Check that ssh is not tcpwrapped. And that it's really
			# SSH
			data = recv(socket:soc, length:200);
			if("SSH" >< data)
				ok = 1;
		}
		close(soc);
	}
	#
	# Only warn the user that SSH is NOT installed
	#  
	if(!ok)
	{
		report = "SSH is not running on this host !";
		security_warning(port:22, data:report);
	}



调整你的脚本
------------------------------------------------

在一次测试过程中，nessusd可能会加载超过200条脚本。如果脚本都没有优化过，那么将耗费很长的时间去执行他们。这也是为什么需要你确保你的脚本越快越好。

要求nessusd仅在需要的时候才执行脚本
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

优化脚本的最好的方法是告诉nessusd，你的脚本在什么情况下是不需要加载的。比如，你的脚本尝试连接到远程TCP端口123。如果nessusd已知这个端口是关闭的，则不需要执行你的脚本，因为即使执行了也得不到任何结果。script_require_ports(),script_require_keys(),script_exclude_keys()函数就是用于这个目的的。在脚本的描述部分就需要调用。
 - script_require_ports(<port1>, <port2>, ...)
 	仅当参数中列出的端口至少有一个开放时，才会执行你的脚本。<port>参数可以是整数(例如:80)或者一个 :ref:`KB <KB>` 中定义的字符串(例如:"Services/www")。

	.. code-block:: python

		script_require_ports(80, "Services/www")

	注意，如果端口状态是未知(比如还没有扫描任何端口)，脚本还是会被执行。
 	
 - script_require_keys(<key1>, <key2>, ...)
 	仅当 :ref:`KB <KB>` 中包含所有参数值时，nessusd才会执行你的脚本。

 	.. code-block:: python

 		script_require_keys("ftp/anonymous", "ftp/writeable_dir")
 		
 	仅当远程FTP服务器提供匿名访问，且存在可以文件夹时，才会执行脚本。
 	
 - script_exclude_keys(<key1>, <key2>, ...)
 	当 :ref:`KB <KB>` 中没有包含所给参数中的至少一个时，nessusd将不会执行你的脚本

使用其他脚本的结果
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

写脚本之前先看下 :ref:`KB <KB>` ，不要做别的脚本已经做过的事情。例如，在使用open_sock_tcp()打开一个端口时，先通过get_port_state()函数确认这个端口是否已经打开。少即是快。

那么，你想分享你的新脚本咯?
------------------------------------------------

如果你计划要分享你的脚本，那么你应该遵循以下规则:
 - 你的脚本必须不能同用户交互。NASL脚本在服务端执行，所有的输出都不会被用户看到。
 - 你的脚本必须只检测一个漏洞。如果你知道如何检测多个漏洞，请写多条脚本。所以，你可以保持所有脚本的一致性
 - 你的脚本应该属于一个已知的分类。如果你计划分享你的脚本，那么请避免创建例如'Joe's Power Tools' 这样的分类，试着保持一致。
 - 查看CVE是否已经存在一个关于你的脚本的定义。如果你关心CVE兼容性，Nessus维护者就不用自己做这些事情了，这也会节省他的时间。
 - 把它发送给我。是的，就是 :ref:`我 <AuthorSrc>` :)。如果你计划去分享你的脚本，那么，就让任何人都可以使用它吗，而不仅仅是你的朋友或者你所拥有的一个组内。一旦你的脚本被加入到Nessus规则集中，它将会被分配一个唯一的ID号。
