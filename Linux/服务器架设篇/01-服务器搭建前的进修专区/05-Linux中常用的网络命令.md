### q5.1 设置网络参数的命令

***

+ ifconfig：查询、设置网卡与IP网络等相关参数。
+ ifup，ifdown：这是两个script文件，其作用是通过更简单的方式来启动与关闭网络接口。
+ route：查看、配置路由表。
+ ip：整合式的命令、可以直接修改上述提到的功能。



1.手动/自动配置IP参数与启动/关闭网络接口：ifconfig、ifup、ifdown

+ ifconfig {interface} {up|down}：查看与启动接口

+ ifconig interface {options}：设置与修改接口

  + interface：网卡接口名称，包括eth0，eth1，ppp0等等
  + options：可以使用的参数，包括：
    + up，down：启动（up）或关闭（down）该网络接口（不涉及任何参数）
    + mtu：可以设置不同的MTU数值，例如mtu 1500 （单位为byte）
    + netmask：子网掩码
    + broadcast：广播地址

+ 查看所有的网路接口

  ```shell
  ifconfig
  ```

+ 查看eth0这个网卡的网络接口

```shell
ifconfig eth0
```

+ 暂时修改网络接口，给予eth0一个192.168.100.100/24的参数

```shell
ifconfig eth0 192.168.100.100
不加任何参数，则系统会依照该IP所在的class范围，自动的计算出netmask以及network，broadcast等IP参数

ifconfig eth0 192.168.100.100 netmask 255.255.255.128 mtu 8000
```

+ 在一个网卡上设置多个IP

```shell
ifconfig eth0:0 192.168.50.50
设备名称是eth0:0。那就是在该实体网卡上，再仿真一个网络接口。

ifconfig eth0:0 down
关闭eth0:0这个接口

ifconfig eth1 up
用默认值启动eth1
```

+ 将手动的处理全部取消，使用原有的设置值重置网络参数：

  ```shell
  /etc/init.d/network restart
  使刚刚设置的数据全部失效，会以ifcfg-ethX的设置为主
  ```



+ ifup、ifdown（配置文件/etc/sysconfig/network-scripts/ifcfg-ethx）
+ ifup {interface}
+ ifdown {interface}
  + 如果以ifconfig eth0的方式来设置或者是修改了网路接口后，那就无法再以ifdown eth0的方式来关闭了
  + 因此使用ifconfig修改完毕后，应该要用ifconfig eth0 down才能够关闭该接口



2.修改路由：route

+ route [-nee]
+ route add [-net|-host] [网络或主机] netmask [mask] [gw|dev]
+ route del [-net|-host] [网络或主机] netmask [mask] [gw|dev]
  + -n：不要使用通信协议或主机名，直接使用IP或port number；
  + -ee：显示更详细的信息
  + -net：表示后面接的路由为一个网络
  + -host：表示后面接的为连接到单部主机的路由
  + netmask：与网络有关，可以设置netmask决定网络的大小
  + gw：gateway的简写，后续接的是IP的数值，与dev不同
  + dev：如果只是指定由哪一块网卡连接出去，则使用这个设置，后面接eth0等
+ 单纯查看路由状态

```shell
route -n
route
```

+ 路由的增加与删除

```shell
route del -net 169.254.0.0 netmask 255.255.0.0 dev eth0
注意删除的时候，需要将路由表上面出现的信息都写入
```

```shell
route add -net 192.168.100.0 netmask 255.255.255.0 dev eth0
注意这个路由的设置必须要能够与你的网络互通

route add default gw 192.168.1.250
增加默认路由的方法。
```

+ 新的环境内的主机，不想更改原系统配置文件的情况下

  ```shell
  ifconfig eth0 192.168.1.100
  route add default gw 192.168.1.254
  /etc/init.d/network restart
  ```

+ 需知道的信息
+ Destination、Genmask
  
  + 这两个参数就分别是network与netmask了。所以它们就组合成为一个完整的网络了。
+ Gateway
  + 该网络是通过哪个Gateway连接出去的？
  + 如果显示0.0.0.0表示该路由是直接由本机传送，也就是可以通过局域网的MAC直接发送。
  + 如果显示IP的话，表示该路由需要经过路由器（网关）的帮忙才能够发送出去。
+ Falgs：总共有多个标志，代表的意义如下。
  + U（route is up）：该路由是启动的。
  + H（target is a host）：目标是一台主机（IP）而非网络。
  + G（use gateway）：需要通过外部的主机来传递数据包
  + R（reinstate route for dynamic routing）：使用动态路由时，恢复路由信息的标志。
  + D（dynamically installed by daemon or redirect）：动态路由
  + M（modified from routing daemon or redirect）：路由已经被修改了。
  + ！（reject route）：这个路由将不会被接受（用来阻止不安全的网络）
+ Iface：这个路由传递数据包的接口

+ 路由排列顺序：依序是由小网络，逐渐到大网络，最后则是默认路由（0.0.0.0/0.0.0.0）



3.网络参数综合命令：ip

+ ip [option] [动作] [命令]
  + option：设置的参数，主要有：
  + -s：显示出设备的统计数据（statistics），例如接收数据包的总数等。
  + 操作：也就是可以针对哪些网络参数进行操作，包括MTU、MAC地址等
  + addr/address：关于额外的IP协议，例如多IP的实现等
  + route：与路由有关的相关设置



+ ip [-s] link show：单纯地查看该设备的相关信息

+ ip link set [device] [动作与参数]

  + 选项与参数：
  + show：仅显示出这个设备的相关属性，如果加上-s会显示更多统计数据
  + set：可以开始设置项目，device指的是eth0、eth1等设备名称
  + 动作与名称：
  + up｜down：启动或关闭某个接口其他参数使用默认的以太网
  + address：如果这个设备可以更改MAC的话，用这个参数修改
  + name：给予这个设备一个特殊的名字
  + mtu：就是最大传输单元

+ 显示本机所有接口的信息

  + ```shell
    ip link show
    ```

  + ```shell
    ip -s link show eth0
    ```

+ 启动关闭与配置设备的相关信息

  + ```shell
    ip link set eth0 up
    ```

  + ```shell
    ip link set eth0 down
    ```

  + ```shell
    ip link set eth0 mtu 1000
    ```

+ 修改网卡名称，MAC等参数

  + ```shell
    ip link set eth0 name gld
    注意需要先关闭这个接口
    ```

  + ```shell
    ip link set eth0 address aa:aa:aa:aa:aa:aa
    ```



+ 关于额外IP的相关设定：ip address
+ ip address show

+ ip address [add|del] [IP参数] [dev 设备名] [相关参数]
  + 选项与参数：
  + show：仅显示接口的IP信息
  + add｜del：进行相关参数的增加（add）或删除）（del）设置，主要有：
    + IP参数：主要就是网络的蛇者，例如192.168.100.100/24之类的设置
    + dev：这个IP参数所有设置的接口，例如eth0、eth1等
  + 相关参数：
    + broadcast：设置广播地址，如果设置值是+表示“让系统自动计算”
    + label：也就是这个设备的别名，例如eth0:0
    + scope：这个选项的参数，通常是这几个大类：
      + global：允许来自所有来源的连接
      + site：仅支持IPv6，仅允许本主机的连接
      + link：仅允许本设备自我连接
      + host：仅允许本主机内部的连接

+ 显示出所有接口的IP参数：

  + ```shell
    ip address show
    ```

+ 添加一个接口，名称假设为eth0:gld

  + ```shell
    ip address add 192.168.50.50/24 broadcast + dev eth0 label eth0:gld
    ```

+ 将刚刚的接口删除

  + ```shell
    ip address del 192.168.50.50/24 dev eth0
    ```

  

3.关于路由的相关设定：ip route  

+ ip route show
+ ip route [add|del] [IP或网络号] [via gateway] [dev 设备]
  + 选项与参数：
  + show：单纯的显示出路由表，也可以使用list
  + add｜del：添加或删除路由
    + IP或网络：可使用192.168.50.50/24之类的网络号或者是单纯的IP地址
    + via：从哪个gateway出去，不一定需要
    + dev：有哪个设备连出去，这就需要了
    + mtu：可以额外的设置MTU的数值



+ 显示出当前的路由信息

  + ```shell
    ip route show
    ```

+ 添加路由，主要是本机直接可沟通的网络

  + ```shell
    ip route add 192.168.5.0/24 dev eth0
    ```

+ 增加可以通往外部的路由，需通过router

  + ```shell
    ip route add 192.168.10.0/24 via 192.168.5.100 dev eth0
    ```

+ 添加默认路由

  + ```shell
    ip route add default via 192.168.1.254 dev eth0
    ```

+ 删除路由

  + ```shell
    ip route del 192.168.10.0/24
    ip route del 192.168.5.0/24
    ```



4.无线网络：iwlist，iwconfig

5.DHCP客户端命令：dhclient

+ ```shell
  dhclient eth0
  ```



### 5.2 网络排错与查看命令

***

1.两台主机的两点沟通：ping

+ ping [选项与参数] IP
  + -c 数值：后面接的是执行ping的次数，例如-c 5
  + -n：在输出数据时不进行IP与主机名的反查，直接使用IP输出（速度较快）
  + -s 数值：发送出去的ICMP数据包大小，默认为56bytes，不过可以放大此数值
  + -t 数值：TTL的数值，默认是255，每经过一个节点就会少1
  + -W 数值：等待响应对方主机的秒数
  + -M [do|dont]：主要在检测网络的MTU数值大小，两个常见的项目是：
    + do：代表传送一个DF（Don‘t Fragment）标志，让数据包不能重新拆包与打包
    + dont：代表不要传送DF标志，表示数据包可以在其他主机上拆包与打包
+ 



2.两主机间各节点分析：traceroute

+ traceroute [选项与参数] IP
  + -n：可以不必进行主机的名称解析，单纯使用IP，速度较快
  + -U：使用UDP的port33434来进行检测，这是默认的检测协议
  + -I：使用ICMP的方式来进行检测
  + -T：使用TCP来进行检测，一般使用port 80测试
  + -w：若对方主机在几秒钟内没有回应就声明不通...默认是5秒
  + -p 端口号：若不想使用UDP与TCP的默认端口来检测，可在此改变端口号
  + -i 设备：用在比较复杂的环境，如果网络接口很多很复杂时，才会用到这个参数；



3.查看本机的网络连接与后门：netstat

+ netstat -[rn]

+ netstat -[antulpc]

  + -r：列出路由表（route table），功能如同route这个命令
  + -n：不使用主机名与服务名称，使用IP与port number，如同route -n与网络接口有关的参数
  + -a：列出所有的连接状态，包括tcp/udp/unix socket等
  + -t：仅列出TCP数据包的连接
  + -u：仅列出UDP的数据包连接
  + -l：仅列出已在Listen（监听）的服务的网络状态
  + -p：列出PID与Program的文件名
  + -c：可以设置几秒钟后自动更新一次，例如-c 5为每5s更新一次网络状态的显示

+ 列出当前的路由表状态，且以IP及port number进行显示

  + ```shell
    netstat -rn
    ```

+ 列出当前的所有网络连接状态，使用IP与port number

  + ```shell
    netstat -an
    ```

+ 显示出目前已经启动的网络服务

  + ```shell
    netstat -tulnp
    ```

+ 查看本机上所有的网络连接状态

  + ```shell
    netstat -atunp
    ```



4.检测主机名与IP的对应：host、nslookup

+ host [-a] hostname [server]

  + -a：列出该主机详细的各项主机名设置数据
  + [server]：可以使用不是有/etc/resolv.conf 文件定义的DNS服务器IP来查询

+ 列出www.yahoo.com

  + ```shell
    host www.yahoo.com
    ```



+ nslookup [-query=[type]] [hostname|IP]

  + -query=type：查询的类型，除了传统的IP与主机名对应外，DNS还有很多信息，包括mx、cname等

+ 找出www.google.com的IP

  + ```shell
    nslookup www.google.com
    ```

+ 找出168.95.1.1的主机名

  + ```shell
    nslookup 168.95.1.1
    ```



### 5.3 远程连接命令与即时通信软件

***

1.终端机与BBS连接：telnet

+ telnet本身的数据在传输过程中使用的是明文

+ telnet [host|IP [port]]

+ 连接到当前热门的PTT BBS 站点ptt.cc

  + ```shell
    yum install telnet
    telnet ptt.cc
    ```

+ 检测本地主机的110这个port是否正确启动

  + ```shell
    telnet localhost 110
    ```



2.FTP连接软件：ftp、lftp

+ ftp：用于处理FTP服务器的下载数据。

+ ftp [host|IP] [port]

  + ```shell
    ftp ftp.ksu.edu.tw
    连接到昆山科大去看看
    anonymous
    help帮助命令
    ```

+ lftp（自动化脚本）

+ lftp [-p port] [-u user[,pass]] [host|IP]

+ lftp -f filename

+ lftp -c "commands"

  + -p：后面可以直接接上远程FTP主机提供的port
  + -u：后面则是接上账号与密码，就能够连接上远程主机了，没有加账号密码则使用anonymous尝试匿名登录
  + -f：可以将命令写入脚本中，这样可以帮助进行shell script的自动处理
  + -c：后面直接加上所需要的命令



### 5.4 文字接口网页浏览

***

1.文字浏览器：links

+ links [options] [URL]

  + -anonymous [0|1]：是否使用匿名登录的意思
  + -dump [0|1]：是否将网页的数据直接输出到standard out 而非 links软件功能
  + -dump_charset：后面接想要通过dump输出到屏幕的语系编码，简体中文使用 cp936

+ 浏览Linux Kernel网站

  + ```shell
    links http://www.kernel.org
    ```

+ 常用功能按键

  + ```shell
    h:history,曾经浏览过的URL就显示到画面中。
    g:Goto URL,按g后输入网页地址（URL），如http://www.abc.edu/等。
    d:download,将该链接数据下载到本机成为文件。
    q:Quit,离开links这个软件。
    o:Option,进入功能参数的设置值修改中，最终可写入/.elinks/elinks.conf中。
    Ctrl+C:强迫中止links的执行。
    ```



+ 通过links将www.yahoo.com的网页内容整个抓下来存储

  + ```shell
    links -dump http://www.yahoo.com > yahoo.html
    ```



2.文字接口下载器：wget

+ wget [option] [网址]
  + --http-user=username
  + --http-password=password
  + --quit：不要显示wget在捕获数据时候的显示信息

+ 可通过proxy的帮助来下载，通过修改/etc/wgetrc来设置你的代理服务器

  ```shell
  vim /etc/wgetrc
  http_proxy = http://proxy.ksu.edu.tw:3128/
  use_proxy = on
  ```



### 5.5 数据包捕获功能

***

1.文字接口数据包捕获器：tcpdump

+ 监听软件，黑客软件

+ 可以分析数据包的流向，连数据包的内容也可以进行监听
+ tcpdump必须使用root的身份执行
+ tcpdump [-AennaX] [-i 接口] [-w 存储文件名] [-c 次数] [-r 文件] [所要摘取的数据包数据格式]
  + -A：数据包的内容以ASCII显示，通常用来抓取WWW的网页数据包数据
  + -e：使用数据链路层（OSI第二层）的MAC数据包数据来显示
  + -nn：直接以IP及port number显示，而非主机名与服务名称
  + -q：仅列出较为简短的数据包信息，每一行的内容比较精简
  + -X：可以列出十六进制（hex）以及ASCII的数据包内容，对于监听数据包内容很有用
  + -i：后面接要监听的网络接口，例如eth0、lo、ppp0等的界面
  + -w：如果你要将监听所得的数据包数据存储下来，用这个参数就对了，后面接文件名
  + -r：从后面接的文件将数据包数据读出来。这个文件是已经存在的文件，并且这个文件是由-w所制作出来的
  + -c：监听的数据包数，如果没有这个参数，tcpdump会持续不断的监听，直到用户输入ctrl+c为止
+ 所欲捕获的数据包数据格式：我们可以针对某些通信协议或者是IP来源进行数据包捕获，那就可以简化输出的结果，并取得最有用的信息。常见的表示方法有：
  + ‘host foo’、‘host 127.0.0.1’：针对单台主机来进行数据包捕获
  + ‘net 192.168’：针对某个网络来进行数据包的捕获
  + ‘src host 127.0.0.1’ ‘dst net 192.168’：同时加上来源（src）或目标（dst）限制
  + ‘tcp port 21’：还可以针对通信协议检测，如tcp、udp、arp、ether等
  + 还可以利用and与or来进行数据包数据的整合显示



+ 以IP与port number获取eth0这个网卡上的数据包，持续3s

  + ```shell
    tcpdump -i eth0 -nn
    ```

+ 只取出port21的连接数据包

  + ```shell
    tcpdump -i eth0 -nn port 21
    ```

+ 数据包运作的过程

  + ```shell
    先开一个终端窗口输入
    tcpdump -i lo -nn
    
    再开一个窗口来对本机登录
    ssh localhost
    ```

+ 如何使用tcpdump 监听来自eth0网卡且通信协议为port22，目标数据来源为192.168.1.101的数据包数据？

  + ```shell
    tcpdump -i eth0 -nn 'port 22 and src host 192.168.1.101'
    ```



2.图形接口数据包捕获器：wireshark

+ 文字接口的wireshark以及图形接口的wireshark-gnome软件

+ ```
  yum install wireshark wireshark-gnome
  ```



3.任意启动TCP/UDP数据包的端口连接：nc、netcat

+ nc可以用来作为某些服务的检测，因为它可以连接到某个port来进行通信

+ 还可以自行启动一个port来监听其他用户的连接。

+ 如果在编译nc软件的使用给予“GAPING_SECURITY_HOME”参数的话，还可以用来取得客户端的bash。

+ 有的系统将执行文件nc改名为netcat了

+ nc [-u] [IP|host] [port]

+ nc -l [IP|host] [port]

  + -l：作为监听之用，也就是打开一个port来监听用户的连接
  + -u：不使用TCP而是使用UDP作为连接的数据包状态

+ 与telnet类似，连接本地端的port 25 查阅相关信息

  + ```shell
    nc localhost 25
    ```

+ 激活一个port 20000来监听用户的连接需求

  + ```shel
    nc -l localhost 20000 &
    netstat -tlunp | grep nc
    ```

  + 再开另外一个终端机，也利用nc来连接服务器，并且输入一些命令看看

  + ```shell
    nc localhost 20000
    ```

  + 客户端输入的文字会同时出现在服务端



