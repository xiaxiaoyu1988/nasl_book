NASL网络相关函数
===============================================

NASL不会允许打开连接到非目的主机的socket。

.. toctree::
   :maxdepth: 2

套接字处理
-----------------------------------------------

套接字(socket)是通过TCP或者UDP同其他主机通信的一种方式。就像一个管道，在给定的协议和端口上发送数据。

如何打开一个套接字
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

open_sock_tcp()和open_sock_udp()会打开一个TCP或者UDP socket。这两个函数使用匿名参数。现在，每次尽可以在一个端口打开socket。将来可能会解决这个问题。

例如：

.. code-block:: python

	# Open a socket on TCP port 80 :
	soc1 = open_sock_tcp(80);
	# Open a socket on UDP port 123 :
	soc2 = open_sock_udp(123);

如果同远程主机无法建立连接，open_sock函数会返回0。通常open_sock_udp()永远不会失败，因为没法确定远程主机的UDP端口是否开放。open_sock_tcp()方法在远程主机端口关闭的情况下会返回0。

一个简单的TCP端口扫描的例子：

.. code-block:: python

	start = prompt("First port to scan ? ");
	end  = prompt("Last port to scan ? ");

	for(i=start;i<end;i=i+1)
	{
		soc = open_sock_tcp(i);
		if(soc) 
		{
			display("Port ", i, " is open\n");
			close(soc);
		}
	}


关闭一个套接字
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

close()函数用于关闭一个socket。在内部会在关闭socket之前调用shutdown()函数。


读写套接字
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

读写socket使用了以下的函数:

 - recv(socket:<socketname>, length:<length> [,timeout : <timeout>])
 	从socketname的socket读取length长度的字节数。可用于TCP和UDP。可选参数timeout的单位是秒。
 - recv_line(socket:<socketname>, length:<length> [, timeout: <timeout>])
 	这个函数同recv()类似，不同点在recv_line在读到\n字符时会停止读取。仅在TCP有效。
 - send(socket:<socket>, data:<data> [, length:<length>])
 	在socket上发送data。可选参数length控制函数发送数据的字节数。如果没有设置length，发送操作会在遇到NULL字符时停止。

这些函数用于从一个socket读取数据，他们有一个内部的超时时长，5秒。如果到达超时市场，将会返回FALSE。

例子:

.. code-block:: python

	# This Example displays the FTP banner of the remote host :

	soc = open_sock_tcp(21);
	if(soc)
	{
		data = recv_line(socket:soc, length:1024);
		if(data)
		{
			display("The remote FTP banner is : \n", data, "\n");
		}
		else
		{
			display("The remote FTP server seems to be tcp-wrapped\n");
		}
		close(soc);
	}



高层函数
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

NASL有一些针对FTP和WWW的高层函数。
 - ftp_log_in(socket:<soc>, user:<login>, pass:<pass>) 
 	将尝试通过socket<soc>登录FTP服务器。如果用户名密码正确返回TRUE，否则返回FALSE。
 - ftp_get_pasv_port(socket:<soc>) 
 	像远程主机发送PASV命令获取连接端口。允许NASL脚本通过FTP下载数据。如果发生错误返回FALSE。
 - is_cgi_installed(<name>) 
 	如果远程主机安装了cgi<name>则返回TRUE。这个函数将发送一个GET请求给远程主机。如果不是以/开头，则认为/cgi-bin/目录被加在前面。这个函数也用于确认某个文件是否存在。

 例子:

.. code-block:: python

	#
	# WWW
	#
	if(is_cgi_installed("/robots.txt"))
	{
		display("The file /robots.txt is present\n");
	}
	if(is_cgi_installed("php.cgi"))
	{
		display("The CGI php.cgi is installed in /cgi-bin\n");
	}
	if(!is_cgi_installed("/php.cgi"))
	{
		display("There is no 'php.cgi' in the remote web root\n");
	}


.. code-block:: python

	#
	# FTP
	#
	# open a connection to the remote host
 	soc = open_sock_tcp(21);
 
	# Log in as the anonymous user
	if(ftp_log_in(socket:soc, user:"ftp", pass:"joe@"))
	{
		# Get a passive port
		port = ftp_get_pasv_port(socket:soc);
		if(port)
		{
			soc2 = open_sock_tcp(port);
			data = string("RETR /etc/passwd\r\n");
			send(socket:soc, data:data);
			password_file = recv(socket:soc2, length:10000);
  			display(password_file);
   			close(soc2);
		}
		close(soc);
	}



原始报文处理
-----------------------------------------------

NASL允许你构造自己的IP报文，而且报文的构造是以一种智能的方式。比如，你改变了TCP报文的参数，TCP报文的校验和会被自动计算。如果你在IP报文上又加了一层，则ip_len会被自动更新，除非你说，不要这样~~~

所有的报文函数都是用命名参数。参数命名都来自BSD包含文件。所有，一个IP报文的长度参数会被命名为ip_len而不是length。


构造IP报文
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

forge_ip_packet()用于构造一个新的IP报文。get_ip_element()函数用于返回一个报文的一个参数，set_ip_elements()用于设置一个已经存在的IP报文的参数。

.. code-block:: python

	<return_value> = forge_ip_packet(
		ip_hl    : <ip_hl>,
		ip_v     : <ip_v>,
		ip_tos   : <ip_tos>,
		ip_len   : <ip_len>,
		ip_id    : <ip_id>,
		ip_off   : <ip_off>,
		ip_ttl   : <ip_ttl>,
		ip_p     : <ip_p>,
		ip_src   : <ip_src>,
		ip_dst   : <ip_dst>,
		[ip_sum  : <ip_sum>] );

ip_sum参数是可选的，如果未被设置，将会自动计算。ip_p参数是一个整型，或者是IPPROTO_TCP, IPPROTO_UDP, IPPROTO_ICMP, IPPROTO_IGMP or IPPROTO_IP其中之一。

.. code-block:: python

	<element> = get_ip_element(
 		ip      : <ip_variable>,
 		element : "ip_hl"|"ip_v"|"ip_tos"|"ip_len"|
 		          "ip_id"|"ip_off"|"ip_ttl"|"ip_p"|
 		          "ip_sum"|"ip_src"|"ip_dst");

get_ip_element()函数用于返回一个报文的一个参数。这个参数必须是"ip_hl", "ip_v", "ip_tos", "ip_len", "ip_id", "ip_off", "ip_ttl", "ip_p", "ip_sum", "ip_src" 或者 "ip_dst"其中之一。注意，引号是必不可少的。

.. code-block:: python

	set_ip_elements( ip	: <ip_variable>,
		  [ip_hl    : <ip_hl>, ]
		  [ip_v     : <ip_v>,  ]
		  [ip_tos   : <ip_tos>,]
		  [ip_len   : <ip_len>,]
		  [ip_id    : <ip_id>, ]
		  [ip_off   : <ip_off>,]
		  [ip_ttl   : <ip_ttl>,]
		  [ip_p     : <ip_p>,  ]
		  [ip_src   : <ip_src>,]
		  [ip_dst   : <ip_dst>,]
		  [ip_sum  : <ip_sum>  ] 
		);

set_ip_elements()用于设置一个已经存在的IP报文的参数，如果没有修改ip_sum的值，它会自动计算。
这个函数没有构造报文的能力，所以还需要在这之前调用forge_ip_packet()。

最后，dump_ip_packet(<packet>)函数用于打印出可读的报文信息。你应该只在调试的时候使用它。


构造TCP报文
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

forge_tcp_packet()用于构造TCP报文。语法如下：

.. code-block:: python

	tcppacket = forge_tcp_packet(ip : <ip_packet>,
                              th_sport : <source_port>,
			      th_dport : <destination_port>,
			      th_flags : <tcp_flags>,
			      th_seq   : <sequence_number>,
			      th_ack   : <acknowledgement_number>,
			      [th_x2   : <unused>],
			      th_off   : <offset>,
			      th_win   : <window>,
			      th_urp   : <urgent_pointer>,
			      [th_sum  : <checkum>],
			      [data    : <data>]);

可选参数th_flags必须为TH_SYN, TH_ACK, TH_FIN, TH_PUSH 或者 TH_RST其中之一，可以使用|符连接。
th_flags也可以是一个整数值。ip_packet必须由forge_ip_packet()函数生成，或者必须是使用send_packet()或pcap_next()函数读取的值。

set_tcp_elements()函数用于修改TCP包的参数值。语法类似forge_tcp_packet():

.. code-block:: python

	set_tcp_elements(tcp : <tcp_packet>,
                              [th_sport : <source_port>,]
			      [th_dport : <destination_port>,]
			      [th_flags : <tcp_flags>,]
			      [th_seq   : <sequence_number>,]
			      [th_ack   : <acknowledgement_number>,]
			      [th_x2    : <unused>,]
			      [th_off   : <offset>,]
			      [th_win   : <window>,]
			      [th_urp   : <urgent_pointer>,]
			      [th_sum   : <checkum>],
			      [data     : <data>] );

这个函数会自动生成TCP校验和，除非你已经指定了th_sum的值。

get_tcp_element()函数用于获取TCP报文中的参数值。语法如下：

.. code-block:: python

	element = get_tcp_elements(tcp: <tcp_packet>,
                  element: <element_name>);

element_name必须是"tcp_sport", ""th_dport", "th_flags", "th_seq", "th_ack", "th_x2", "th_off", "th_win", "th_urp", "th_sum"其中之一，注意引号必须有。


构造UDP报文
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

UDP函数的使用类似TCP函数

.. code-block:: python

	udp = forge_udp_packet(ip:<ip_packet>,
                        uh_sport : <source_port>,
			uh_dport : <destination_port>,
			uh_ulen  : <length>,
			[uh_sum  : <checksum>],
			[data    : <data>]);

set_udp_elements()和get_udp_elements()函数通TCP类似。


构造ICMP报文
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

暂缺(原文未写)


构造IGMP报文
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

暂缺(原文未写)


发送报文
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

当你通过forge_*_packet()函数构造好报文后，就可以通过send_packet()函数来发送它。

语法如下:

.. code-block:: python

	reply = send_packet(packet1, packet2, ...., packetN,
                     pcap_active: <TRUE|FALSE>,
                     pcap_filter: <pcap_filter>);

如果pcap_active被设置为TRUE(默认值)，这个函数会等待远程主机回复。你可以设置pcap_filter参数去选择你需要哪种类型的报文。查找 `pcap <http://www.tcpdump.org/pcap3_man.html>`_ (或者 `tcpdump <http://www.tcpdump.org/tcpdump_man.html>`_ )来学习如何填写pcap_filter参数。

读取报文
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

pcap_next()函数用于读取报文。语法如下:

.. code-block:: python

	reply = pcap_next();

这个函数会从你最近使用的接口中读取报文，报文类型取决于你最后设置的pcap_filter类型。


工具函数
-----------------------------------------------

NASL提供了工具函数来简化编程。
 - this_host() 无需参数，返回运行脚本主机的IP地址。
 - get_host_name() 无需参数，返回目的主机的主机名。
 - get_host_ip() 无需参数，返回目的主机的IP地址。
 - get_host_open_port() 无需参数，返回远程主机打开的第一个端口号。对于某些脚本来说很实用，比如
 	land或者TCP序列分析程序。
 - get_port_state(<portnum>) 如果端口portnum打开或者未知(比如端口未被扫描到，或者不在扫描范围之内), 返回TRUE，
 - telnet_init(<soc>) 在socket<soc>上初始一个telnet会话，并返回数据的第一行。
 	例如:

	.. code-block:: python

		soc = open_sock_tcp(23);
		buffer = telnet_init(soc);
		display("The remote telnet banner is : ", buffer, "\n");
 	
 - tcp_ping() 无需参数，如果远程主机响应了TCP ping(发送一个设置了ACK标记的TCP包)请求，则返回TRUE。
 - getrpcport() 返回rpc端口号。语法如下；

 	.. code-block:: python

 		result = getrpcport(program : <program_number>,
                     protocol: IPPROTO_TCP|IPPROTO_UDP,
                     [version: <version>]);

	如果出错则返回0(如果<program_number>没有在远程主机RPC端口映射表中注册)。

