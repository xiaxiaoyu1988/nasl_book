附录
===============================================
.. toctree::
   :maxdepth: 2

.. _KB:

知识库(KB)
-----------------------------------------------

知识库，是一个包含脚本测试结果的键值对集合。使用script_dependencies(), get_kb_item(), set_kb_item()函数，可以优化你的脚本，避免做重复的事情。

这里是知识库的总结：

KB 的条目可能有多个值。比如，想象一下，远程主机上起了两个FTP服务，分别监听在21和2100端口上。那么Servces/ftp的值等于21和2100.如果这样的话，这个脚本会被执行两次，第一次，get_kb_item("Services/ftp")返回21，第二次返回2100。但这个行为是自动的，脚本中并不需要关系，只需要知道每次只会返回一个值。即使在真实条件下，也是这样，nessusd会控制整个流程。

并不是所有的keys都是有用的。有些我从来没用过。相对而言，多放点参数在里面总比没地方设置值要好。(**到这也基本快翻译完成了，下面的keys的含义就不翻译了，相信大家也都能看懂**)
 - Host/OS
	Defined in : queso.nasl and nmap_wrapper.nasl
	Type : string
	Meaning : Remote operating system type
 - Host/dead
	Defined in : ping_host.nasl and all the DoS plugins Type : boolean
	Meaning : The remote host is dead. If you set this item, then nessusd will interrupt the test of the host.
 - Services/www
	Defined in : find_service.nes Type : port number
	Meaning : port on which a web server is running. Returns 0 if no web server has been found.
 - Services/auth
	Defined in : find_service.nes Type : port number
	Meaning : port on which an identd server is running. Returns 0 if no such server has been found
 - Services/echo
	Defined in : find_service.nes Type : port number
	Meaning : port on which 'echo' is running. Returns 0 if no such service has been found
 - Services/finger
	Defined in : find_service.nes Type : port number
	Meaning : port on which a finger server is running. Returns 0 if no such server has been found
 - Services/ftp
	Defined in : find_service.nes Type : port number
	Meaning : port on which an ftp server is running. Returns 0 if no such server has been found
 - Services/smtp
	Defined in : find_service.nes Type : port number
	Meaning : port on which an SMTP server is running. Returns 0 if no such server has been found
 - Services/ssh
	Defined in : find_service.nes Type : port number
	Meaning : port on which an SSH server is running. Returns 0 if no such server has been found
 - Services/http_proxy
	Defined in : find_service.nes Type : port number
	Meaning : port on which an HTTP proxy is running. Returns 0 if no such server has been found
 - Services/imap
	Defined in : find_service.nes Type : port number
	Meaning : port on which an imap server is running. Returns 0 if no such server has been found
 - Services/pop1
	Defined in : find_service.nes Type : port number
	Meaning : port on which a POP-1 server is running. Returns 0 if no such server has been found
 - Services/pop2
	Defined in : find_service.nes Type : port number
	Meaning : port on which a POP-2 server is running. Returns 0 if no such server has been found
 - Services/pop3
	Defined in : find_service.nes Type : port number
	Meaning : port on which a POP-3 server is running. Returns 0 if no such server has been found
 - Services/nntp
	Defined in : find_service.nes Type : port number
	Meaning : port on which an NNTP server is running. Returns 0 if no such server has been found
 - Services/linuxconf
	Defined in : find_service.nes Type : port number
	Meaning : port on which a linuxconf server is running. Returns 0 if no such server has been found
 - Services/swat
	Defined in : find_service.nes Type : port number
	Meaning : port on which a SWAT server is running. Returns 0 if no such server has been found
 - Services/wild_shell
	Defined in : find_service.nes Type : port number
	Meaning : port on which a shell is open to the world (usually a bad thing). Returns 0 if no such server has been found
 - Services/telnet
	Defined in : find_service.nes Type : port number
	Meaning : port on which a telnet server is running. Returns 0 if no such server has been found
 - Services/realserver
	Defined in : find_service.nes Type : port number
	Meaning : port on which a RealServer server is running. Returns 0 if no such server has been found
 - Services/netbus
	Defined in : find_service.nes Type : port number
	Meaning : port on which a NetBus server is running (usually not a good thing). Returns 0 if no such server has been found
 - bind/version
	Defined in : bind_version.nasl Type : string
	Meaning : version of the remote BIND daemon
 - rpc/bootparamd
	Defined in : bootparamd.nasl Type : string
	Meaning : The bootparam RPC service is running
 - Windows compatible
	Defined in : ca_unicenter_file_transfer_service.nasl, ca_unicenter_transport_service.nasl, mssqlserver_detect.nasl and windows_detect.nasl Type : boolean value
	Meaning : The remote host appears to be running a Windows-compatible operating system (this test is only done regarding the number of the opened-ports)
 - finger/search.\*\*\@host
	Defined in : cfinger_search.nasl Type : boolean value
	Meaning : The finger daemon dumps the list of users if the query .** is made
 - finger/0\@host
	Defined in : finger_0.nasl Type : boolean value
	Meaning : The finger daemon dumps a list of users if the query 0 is made
 - finger/.\@host
	Defined in : finger_dot.nasl Type : boolean value
	Meaning : The finger daemon dumps a list of users if the query . is made
 - finger/user\@host1\@host2
	Defined in : finger_0.nasl Type : boolean value
	Meaning : The finger daemon is vulnerable to a redirection attack
 - www/frontpage
	Defined in : frontpage.nasl Type : boolean value
	Meaning : The remote web server is running frontpage extensions
 - ftp/anonymous
	Defined in : ftp_anonymous.nasl Type : boolean value
	Meaning : The remote FTP server accepts anonymous logins
 - ftp/root_via_cwd
	Defined in : ftp_cwd_root.nasl Type : boolean value
	Meaning : It is possible to gain root on the remote FTP server using the CWD ~ bug (see CVE-1999-0082)
 - ftp/microsoft
	Defined in : ftp_overflow.nasl Type : boolean value
	Meaning : The remote server is a Microsoft FTP server, which closes the connection whenever a too long argument is issued.
 - ftp/false_ftp
	Defined in : ftp_overflow.nasl Type : boolean value
	Meaning : the remote FTP server is either protected by tcp wrappers or the FTP port is open but closes the connection

nasl工具
-----------------------------------------------

现在libnasl拥有了独立的解释器。使用man nasl获取更多详细信息。

.. figure:: ..\_static\nasl.png