---
layout: post
title:  "programming with libpcap on linux"
author: chenlian
categories: network
tags: network
excerpt: The Packet Capture library provides a high level interface to packet capture systems. All packets on the network, even those destined for other hosts, are accessible through this mechanism. It also supports saving captured packets to a "savefile", and reading packets from a "savefile".
---


* content
{:toc}
The Packet Capture library provides a high level interface to packet capture systems. All packets on the network, even those destined for other hosts, are accessible through this mechanism. It also supports saving captured packets to a "savefile", and reading packets from a "savefile".


## 简介


libpcap是Unix/Linxu 平台下的网络数据包捕获函数包，大多数网络监控软件都以它为基础。libpcap提供了系统独立的用户级别网络数据包接口，并充分考虑到应用程序的可移植性。


## libpcap函数简介


char * pcap_lookupdev(char * ebuf);


* 返回值　： 返回第一个网络设备名（不会为“lookback”回环接口）；没找到网络设备，返回NULL。
* ebuf ：出错信息，没找到网络设备，"No driver found"


int pcap_lookupnet(const char * device, bpf_u_int32 * localnet,bpf_u_int32 * netmask,char * errbuf);


* 返回值 ：出错返回 -1 ， 成功 1。
* device ： 网络设备名
* localnet : 本地网络号
* netmask ： 子网掩码
* errbuf ：出错信息


pcap_t * pcap_open_live(char * device,int snaplen,int promisc,int to_ms,char * ebuf);


* 返回值 ： 成功返回pcap_t 指针；出错，返回 NULL
* device : 打开的网络设备名
* snaplen : 转包的最大的长度
* promisc : 非0，网卡设为混杂模式
* to_ms : 嗅探的时间，单位为milliseconds，毫秒；为 0，则一直嗅探
* ebuf ：出错信息


int pcap_compile(pcap_t * p , struct bfp_program * program, const char * buf,int optimize,bpf_u_int32 mask);


* 返回值 ：　成功返回０；　出错，返回－１
* pcap_t　： 打开的会话的指针
* program : 存储 pcap_compile 的过滤的信息的指针
* optimize ： 是否充分利用标志，0 否，1 是
* mask ：网络子网掩码

int pcap_setfilter(pcap_t * p , struct bpf_program * fp);


* 返回值 ：　成功返回０；出错返回　－１　
* pcap_t　：　打开的会话的指针
* fp :  存储 pcap_compile 的过滤的信息的指针


const u_char * pcap_next(pcap_t * p,struct pcap_pkthdr *h);


* 返回值 ： 成功数据包的内存地址；出错返回0；
* p :  打开的会话的指针
* h :  pcap的数据包的通用信息的指针

int pcap_loop(pcap_t * p , int cnt, pcap_handler callback, u_char * user);


* 返回值 ： 0 读结束
* p ： 打开的会话的指针
* cnt ： 嗅探的数据包的个数。小于或等于0，则一直抓包
* callback ：　自定义的数据包处理回调函数
* user : 特殊使用，大部分设置NULL


回调函数的格式：
void (* ) (u_char * args , const struct pcap_pkthad * header,const u_char *packet);


* args :  pcap_loop()函数的最后一个参数。
* header ： pcap的数据包的通用信息的指针
* packet :  数据包的指针


在callback 中获取中packet指针，可以使用Linux 系统中的对数据包处理的函数，如处理以太网数据头，ip数据头，tcp或udp数据头，和payload 数据。


## 简单的例子


```c
#include<pcap.h>
#include<stdio.h>
#include<stdlib.h>
#include<errno.h>
#include<sys/socket.h>
#include<netinet/in.h>
#include<arpa/inet.h>
#include<net/ethernet.h>
#include<netinet/if_ether.h>
#include<netinet/ether.h>

void pcap_packet_callback_print(u_char * args,
        const struct pcap_pkthdr * header,const u_char * packet)
{
    /*
    static u_int32_t  num ;
    num++;
    //fprintf(stdout,"%d\n",num);
    printf("%d\n",num);
    */
    struct ether_header * eptr;
    eptr = (struct ether_header *) packet;

    fprintf(stdout,"ethernet header source:%s\n",ether_ntoa((struct ether_addr *)eptr->ether_shost));
    fflush(stdout);
}

int main(int argc,char * argv[])
{
    pcap_t * handle;
    char * dev;
    char errbuf[PCAP_ERRBUF_SIZE];
    struct bpf_program fp;
    char filter_exp[] = "port 80";
    bpf_u_int32 mask;
    bpf_u_int32 net;
    struct pcap_pkthdr header;
    const u_char * packet;

    /* Define the device */
    dev = pcap_lookupdev(errbuf);
    if(dev == NULL){
        fprintf(stderr,"Couldn't find default device:%s\n",errbuf);
        return 2;
    }

    /* Find the properties for the device */
    if(pcap_lookupnet(dev,&net,&mask,errbuf) == -1){
        fprintf(stderr,"Couldn't get netmask for device %s:%s\n",dev,errbuf);
        net = 0;
        mask = 0;
    }
    /* Open the session in promiscuous mode */
    handle = pcap_open_live(dev,BUFSIZ,1,1000,errbuf);
    if(handle == NULL){
        fprintf(stderr,"Couldn't open device %s:%s\n",dev,errbuf);
        return 2;
    }

    /* Compile and apply the filter */
    if(pcap_compile(handle,&fp,filter_exp,0,net) == -1){
        fprintf(stderr,"Couldn't parse filter %s:%s\n",filter_exp,pcap_geterr(handle));
        return 2;
    }

    if(pcap_setfilter(handle,&fp) == -1){
        fprintf(stderr,"Couldn't install filter %s:%s\n",filter_exp,pcap_geterr(handle));
    }

    /*Grab a packet
    packet = pcap_next(handle,&header);
    */

    printf("before pcap_loop! \n");
    pcap_loop(handle,-1,&pcap_packet_callback_print,NULL);
    /* Print its length */
    printf("Jacked a packet with length of [%d]\n",header.len);

    /* And close the session */
    pcap_close(handle);

    return 0;
}

```

