---
layout:     post
title:      "too many open files 问题记录"
subtitle:   "排查和参数调整的方法"
date:       2017-09-20
author:     "BENJAMIN"
header-img: "img/toomanyopenfileserror.jpg"
tags:
    - 服务器相关
    - linux  
    - 文件流
    - too many open files
   
---


> 生产环境一直时不时报出too many open files 错误，导致应用和接口都无法访问，记录一下修改方式。  
> 后面还需继续观察。

---


# 1.分析


---

从报错中可以看出是打开的文件过多，并且从log来看是进行http访问的时候的报错，每个socket应该都算是一次文件打开，
那么问题主要在两个点：  
1. 程序是再哪里打开了文件，没有及时关闭；
2. 系统能够打开最多的文件数是多少，怎么调整。



# 2. 程序检查
--

检查代码搜索BufferedReader，检查是否都在finally中都有调用对应的close()方法，有找到类似有问题的方法如下：

```
/**   
     * 从文件中加载私钥   
     * @param keyFileName 私钥文件名   
     * @return 是否成功 
     * @throws Exception  
     */  
    private static RSAPrivateKey loadPrivateKey(InputStream in) throws Exception{  
        try {  
            BufferedReader br= new BufferedReader(new InputStreamReader(in));  
            String readLine= null;  
            StringBuilder sb= new StringBuilder();  
            while((readLine= br.readLine())!=null){  
                if(readLine.charAt(0)=='-'){  
                    continue;  
                }else{  
                    sb.append(readLine);  
                    sb.append('\r');  
                }  
            }  
            logger.info("prikeyString:" + sb.toString());
            return  loadPrivateKey(sb.toString());  
        } catch (IOException e) {  
            e.printStackTrace();
            throw new Exception("数据读取错误");  
        } catch (NullPointerException e) {  
            e.printStackTrace();
            throw new Exception("输入流为空");  
        }  
    }  
```

应该在finaly中进行 br.close()的操作。  
后续同样检查其他关于流的代码。

# 3.服务器配置
--
linux默认每个进程可以打开的最大文件数为1024，

```
ulimit -a
``` 
可以查看

```
[root@QITV0481 ~]# ulimit -a
core file size          (blocks, -c) 0
data seg size           (kbytes, -d) unlimited
scheduling priority             (-e) 0
file size               (blocks, -f) unlimited
pending signals                 (-i) 62827
max locked memory       (kbytes, -l) 64
max memory size         (kbytes, -m) unlimited
open files                      (-n) 1024
pipe size            (512 bytes, -p) 8
POSIX message queues     (bytes, -q) 819200
real-time priority              (-r) 0
stack size              (kbytes, -s) 10240
cpu time               (seconds, -t) unlimited
max user processes              (-u) 62827
virtual memory          (kbytes, -v) unlimited
file locks                      (-x) unlimited

```

**有时候以为服务器压力很大，并发特别高，或者需要同步的数据量也特别大的时候，  
会造成需要打开很多文件或者http请求频繁，所以服务器也要相应调高来应对高并发的场景**

```
lsof -n |awk '{print $2}'|sort|uniq -c |sort -nr|more
```
可以查看所有进程打开的文件统计

```
[carapp@qitv0502 ~]$ sudo lsof -n |awk '{print $2}'|sort|uniq -c |sort -nr|more
[sudo] password for carapp:
   2156 21890
   1296 22845
    162 7378
    125 2190
     91 6681
     88 23154
     61 6215
     61 21478
     60 9043
     60 21313
     57 1939
     56 8829
     56 6213
     56 21085
     56 13681
     53 2207
     46 7568
     46 6594
     46 23991
     46 2197
     43 2094
     39 2332
     37 2283
```
可以看到21890 这个进程打开了2156这个文件，由于系统资源不断使用和释放这个值也是不停在变的


## 3.1 调整可开打文件数
加大打开的文件数

```
ulimit -n 10240
```
看资料这样修改只是用户级别的修改，重启后会失效(暂未验证)。  

系统级别修改：

```
vi /etc/sysctl.conf
```

增加：

```
#增加文件打开的数量
fs.file-max=10241

```
保存后执行启动配置

```
sysctl -p

```

另外的一些备注  
/proc 目录  
/proc 目录包括很多系统当前状态的参数  
/proc/meminfo  
/proc/cpuinfo  
/proc/sys/fs/file-max #系统总限制  
/proc/sys/fs/file-nr #整个系统目前使用的文件句柄数量  


## 3.2 调整TCP参数

仅实践了```net.ipv4.tcp_keepalive_time```，以下作为备注

```
#参数优化
#表示如果套接字由本端要求关闭，这个参数决定了它保持在FIN-WAIT-2状态的时间
net.ipv4.tcp_fin_timeout = 30
#表示当keepalive起用的时候，TCP发送keepalive消息的频度。缺省是2小时，改为300秒，原因是：当前都是图片，不需要建立长链接
net.ipv4.tcp_keepalive_time = 300
#表示开启SYN Cookies。当出现SYN等待队列溢出时，启用cookies来处理，可防范少量SYN攻击，默认为0，表示关闭
net.ipv4.tcp_syncookies = 1
#表示如果套接字由本端要求关闭，这个参数决定了它保持在FIN-WAIT-2状态的时间
net.ipv4.tcp_tw_reuse = 1
#表示开启TCP连接中TIME-WAIT sockets的快速回收，默认为0，表示关闭，当前TIME-WAIT 过多，所以开启快速回收
net.ipv4.tcp_tw_recycle = 1
net.ipv4.ip_local_port_range = 5000 65000
```

## 3.3 Tomcat配置
connectionTimeout - 网络连接超时，单位：毫秒。设置为0表示永不超时，这样设置有隐患的。通常可设置为30000毫秒。
