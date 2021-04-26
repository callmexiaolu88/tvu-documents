# IPtables

IPtables -> tables -> chains -> rules

- tables and chains

  | Chains          | Built-in                           | Built-in                                                     | Built-in                                         | Built-in       | Custom           |
  | --------------- | ---------------------------------- | ------------------------------------------------------------ | ------------------------------------------------ | -------------- | ---------------- |
  |                 | **Filter**                         | **NAT**                                                      | **Mangle**                                       | **Raw**        | **[table name]** |
  | **INPUT**       | 处理来自外部的数据。               |                                                              | 表用于指定如何处理数据包。它能改变TCP头中的QoS位 |                |                  |
  | **OUTPUT**      | 处理向外发送的数据。               | 处理本机产生的数据包。                                       | 表用于指定如何处理数据包。它能改变TCP头中的QoS位 | 表用于处理异常 |                  |
  | **FORWARD**     | 将数据转发到本机的其他网卡设备上。 |                                                              | 表用于指定如何处理数据包。它能改变TCP头中的QoS位 |                |                  |
  | **PREROUTING**  |                                    | 处理刚到达本机并在路由转发前的数据包。它会转换数据包中的目标IP地址（destination ip address），通常用于DNAT(destination NAT) | 表用于指定如何处理数据包。它能改变TCP头中的QoS位 | 表用于处理异常 |                  |
  | **POSTROUTING** |                                    | 处理即将离开本机的数据包。它会转换数据包中的源IP地址（source ip address），通常用于SNAT（source NAT）。 | 表用于指定如何处理数据包。它能改变TCP头中的QoS位 |                |                  |

  the orders of table:

  **Raw—>mangle—>nat—>filter**

- Rules

  - Rules包括一个条件和一个目标(target)
  - 如果满足条件，就执行目标(target)中的规则或者特定值。
  - 如果不满足条件，就判断下一条Rules。

  **Target values(built-in):**

  - **ACCEPT** – 允许防火墙接收数据包
  - **DROP** – 防火墙丢弃包
  - **QUEUE** – 防火墙将数据包移交到用户空间
  - **RETURN** – 防火墙停止执行当前链中的后续Rules，并返回到调用链(the calling chain)中。
  - **OtherTarget** - 其它Chain

![linux iptables详解--个人笔记](http://i2.51cto.com/images/blog/201804/03/d2510f96d6c026c5d07f5421374a4b92.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

1、目的地址是本地，则发送到INPUT，让INPUT决定是否接收下来送到用户空间，流程为①--->②;

2、若满足PREROUTING的nat表上的转发规则，则发送给FORWARD，然后再经过POSTROUTING发送出去，流程为： ①--->③--->④--->⑥

3、主机发送数据包时，流程则是⑤--->⑥

![img](https://images2015.cnblogs.com/blog/1206313/201707/1206313-20170725211002763-1630915226.png)