-   正如其他人所讲，[内网IP地址](https://www.zhihu.com/search?q=%E5%86%85%E7%BD%91IP%E5%9C%B0%E5%9D%80&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A137182234%7D)、外网IP地址这个概念并不是固定的，而是相对的。如果用私有IP、[公网IP](https://www.zhihu.com/search?q=%E5%85%AC%E7%BD%91IP&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A137182234%7D)或者局域网IP、互联网IP来理解就容易多了。  
    
-   看[网络习惯](https://www.zhihu.com/search?q=%E7%BD%91%E7%BB%9C%E4%B9%A0%E6%83%AF&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A137182234%7D)书籍无法理解很多原因是因为[教科书](https://www.zhihu.com/search?q=%E6%95%99%E7%A7%91%E4%B9%A6&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A137182234%7D)太古老，不与时俱进造成的。几乎所有的教科书都会告诉大家私有IP有3种：A类10.0.0.0～10.255.255.255，B类172.16.0.0～172.31.255.255，C类192.168.0.0～192.168.255.255。但事实上远远不止。下面有详细的列表。  
    

```text
Address Block                    Name                              RFC                       
0.0.0.0/8                        "This host on this network"       [RFC1122], section 3.2.1.3
10.0.0.0/8                       Private-Use                       [RFC1918]                 
100.64.0.0/10                    Shared Address Space              [RFC6598]                 
127.0.0.0/8                      Loopback                          [RFC1122], section 3.2.1.3
169.254.0.0/16                   Link Local                        [RFC3927]                 
172.16.0.0/12                    Private-Use                       [RFC1918]                 
192.0.0.0/24[2]                  IETF Protocol Assignments         [RFC6890], section 2.1    
192.0.0.0/29                     IPv4 Service Continuity Prefix    [RFC7335]                 
192.0.0.8/32                     IPv4 dummy address                [RFC7600]                 
192.0.0.9/32                     Port Control Protocol Anycast     [RFC-ietf-pcp-anycast-08] 
192.0.0.170/32, 192.0.0.171/32   x/DNS64 Discovery             [RFC7050], section 2.2    
192.0.2.0/24                     Documentation (TEST-NET-1)        [RFC5737]                 
192.31.196.0/24                  AS112-v4                          [RFC7535]                 
192.52.193.0/24                  AMT                               [RFC7450]                 
192.88.99.0/24                   Deprecated (6to4 Relay Anycast)   [RFC7526]                 
192.168.0.0/16                   Private-Use                       [RFC1918]                 
192.175.48.0/24                  Direct Delegation AS112 Service   [RFC7534]                 
198.18.0.0/15                    Benchmarking                      [RFC2544]                 
198.51.100.0/24                  Documentation (TEST-NET-2)        [RFC5737]                 
203.0.113.0/24                   Documentation (TEST-NET-3)        [RFC5737]                 
240.0.0.0/4                      Reserved                          [RFC1112], section 4      
255.255.255.255/32               Limited Broadcast                 [RFC919], section 7 
```

-   如上表，[运营商](https://www.zhihu.com/search?q=%E8%BF%90%E8%90%A5%E5%95%86&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A137182234%7D)给你的100.64.*.*也是[私有地址](https://www.zhihu.com/search?q=%E7%A7%81%E6%9C%89%E5%9C%B0%E5%9D%80&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A137182234%7D)（因为地主家也没有余量啊，以前大家大家共享一个地址池，有随机的[公网地址](https://www.zhihu.com/search?q=%E5%85%AC%E7%BD%91%E5%9C%B0%E5%9D%80&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A137182234%7D)，现在公网地址更加紧张，运营商只给客户分配私网地址，然后nat后大家共享一个公网地址），《盗[梦空间](https://www.zhihu.com/search?q=%E6%A2%A6%E7%A9%BA%E9%97%B4&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A137182234%7D)》看过吧，[二重梦境](https://www.zhihu.com/search?q=%E4%BA%8C%E9%87%8D%E6%A2%A6%E5%A2%83&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A137182234%7D)，你用路由器上网也就是二重局域网。也就是说：如果内网、外网是指**私有地址与公网地址的话，那么100.64.0.30和192.168.1.101都是内网IP，你没有外网IP**。如果内网、外网是**相对你路由器来讲，那么100.64.0.30是外网IP，192.168.1.101是内网IP**。  
    

1.  问题1：不用家庭路由器，直接电脑分配到100.64.0.30，那么理论上100.64.0.1-100.64.0.254的确可以直接通讯，无需运营商路由。但实际上是通过各种技术做了隔离的。你无法访问他们。
2.  问题2：因为你路由器获取的地址仍然是私网地址，所以你在外部连路由器都无法访问。除非在运营商的设备上做[端口映射](https://www.zhihu.com/search?q=%E7%AB%AF%E5%8F%A3%E6%98%A0%E5%B0%84&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A137182234%7D)才能访问到路由器。