# 从0实现一个HTTP Web Server，记录过程


# 1、阅读《TCP网络编程》
## 接口大全

####TCP:  
#### int socket(int domain, int type, int protocol)    
return: fd | -1    params: 1、PF_INET | PF_INET6 | etc. 2、SOCK_STREAM (tcp) | SOCK_DGRAM (udp)  3、IPPROTO_TCP | IPPROTO_UDP
#### int bind(int sockfd, struct sockaddr *myaddr, socklen_t addrlen)    
return: 1 | 0    params: 1、fd  2、结构体（地址族和地址信息） 3、地址长度
#### int listen(int sockfd, int backlog)    
return: 0 | -1    params: 1、希望进入等待连接请求状态的套接字文件描述符fd  2、连接请求等待队列的长度
#### int accept(int sockfd, struct sockaddr *serv_addr, socklen_t addrlen)    
return: fd | -1    params: 1、fd 2、保存客户端地址信息的结构体（地址族和地址信息） 3、地址长度
#### int connect(int sockfd, struct sockaddr *serv_addr, socklen_t addrlen)    
return: 0 | -1    params: 1、fd 2、保存服务器地址信息的结构体（地址族和地址信息） 3、地址长度

#### unsigned short/long htons/htonl(unsigned short/long)   
字节序转换 host to network  #include <arpa/inet.h>  
return: 网络字节   params: 主机字节
#### unsigned short/long ntohs/ntohl(unsigned short/long)   
字节序转换 network to host  #include <arpa/inet.h>  
return: 主机字节   params: 网络字节

#### int inet_aton(const char *string, struct in_addr* addr)    
网络地址转换:127.232.124.79(char* | str) -> 0x4f7ce87f(int)  
return: 1 | 0    params: 1、字符串 2、结构体in_addr，内部是32位的IPv4地址 -> in_addr_t(POSIX标准下的uint32_t)
#### char* inet_ntoa(struct in_addr adr)
网络字节序整数型IPv4地址(struct in_addr) -> str  
return: 指向字符串的指针    params: 32位的IPv4地址

####UDP:  
#### ssize_t sendto(int sock, void* buff, size_t nbytes, int flags, struct sockaddr* to, socklen_t addrlen)  
UDP发送端函数  
return: 成功时返回传输的字节数 | -1    params: 1、UDP套接字fd 2、保存待传输数据的缓冲地址值 3、待传输数据长度 4、可选项参数，没有则为0 5、目的地址 6、目的地址长度  
#### ssize_t recvfrom(int sock, void* buff, size_t nbytes, int flags, struct sockaddr* from, socklen_t addrlen)  
UDP接收端函数  
return: 成功时返回传输的字节数 | -1    params: 1、UDP套接字fd 2、保存待传输数据的缓冲地址值 3、待传输数据长度 4、可选项参数，没有则为0 5、源地址 6、源地址长度     
UDP补充说明： UDP也可以使用connect来向套接字注册目标IP和端口号，然后使用sendto/recvfrom/read/write函数来传递数据。这并不意味着要与对方的UDP套接字连接，仅仅是注册了IP和port。  

#### int shutdown(int sock, int howto)  
return: 0 | -1    params: 1、需要断开的fd 2、断开方式（SHUT_RD | SHUT_WR | SHUT_RDWR）-> (断开输入流|断开输出流|I/O均断开)  

#### struct hostent* gethostbyname(const char* hostname)   
通过域名获取IP地址  
return: hostent结构体地址 | NULL指针    
struct hostent{1、char* h_name官方域名 2、char** h_aliases多个域名列表 3、int h_addrtype IP地址的地址族 4、int h_length IP地址长度 5、char** h_addr_list IP地址};
