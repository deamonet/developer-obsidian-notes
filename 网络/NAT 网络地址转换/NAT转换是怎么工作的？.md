1.  **  
    RT** 收到报文，建立 10.0.0.1:1024 到 219.134.180.11:2001 的映射关系，转换源 IP 地址和 TCP 端口。根据目的端口 21 ，RT 识别出这是一个 FTP 报文，因此还要检查应用层数据，发现[原始数据](https://www.zhihu.com/search?q=%E5%8E%9F%E5%A7%8B%E6%95%B0%E6%8D%AE&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A1917791148%7D)为 “ IP = 10.0.0.1 port=5001 ”，于是为 Data 通道 10.0.0.1:5001 建立第二个映射关系：10.0.0.1:5001 到 219.134.180.11:2002 ，转换后的报文源地址和端口为 219.134.180.11:2001 ，目的地址和端口不变，携带数据为 “ IP = 219.134.180.11 port=2002 ”。

  

![](media/v2-fbddff3f2a5728e33919b49b9e05e67b_1440w.jpg.png)

  

1.  **Server** 收到报文，向 A 回应 command okay 报文，FTP Control 通道建立成功。同时 Server 根据应用层数据确定 A 的 Data 通道网络参数为 219.134.180.11:2002 。  
    
2.  **A** 需要从 FTP 服务器下载文件，于是发起文件请求报文。Server 收到请求后，发起 Data 通道建立请求，IP 报文的源地址和端口为 61.144.249.229:20 ，目的地址和端口为 219.134.180.11:2002，并携带 FTP 数据。  
    

  

![](media/v2-9b4e1d736f66e0d557fbfc17bfb7c552_1440w.jpg)

[发布于 2021-06-01 22:50](https://www.zhihu.com/question/31332694/answer/1917791148)

真诚赞赏