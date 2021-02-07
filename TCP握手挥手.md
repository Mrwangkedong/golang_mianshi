## TCP/IP

#### TCP报文格式

<img src="C:\Users\Cristiano-Ronaldo\AppData\Roaming\Typora\typora-user-images\image-20210128181030976.png" alt="image-20210128181030976" style="zoom:80%;" />



#### 三次握手

<img src="C:\Users\Cristiano-Ronaldo\AppData\Roaming\Typora\typora-user-images\image-20210128182007816.png" alt="image-20210128182007816" style="zoom:80%;" />



#### 四次挥手

<img src="C:\Users\Cristiano-Ronaldo\AppData\Roaming\Typora\typora-user-images\image-20210128184727058.png" alt="image-20210128184727058" style="zoom:80%;" />

![img](https://images2015.cnblogs.com/blog/964016/201608/964016-20160829222234683-593863018.png)





#### 为什么TCP是四次挥手而不是三次

**tcp是全双工通信，服务端和客服端都能发送和接收数据。**

**==tcp在断开连接时，需要服务端和客服端都确定对方将不再发送数据。==**

（1）第一次挥手

​     因此当主动方发送断开连接的请求（即FIN报文）给被动方时，仅仅代表==主动方不会再发送**数据报文**==了，但主动方仍可以接收数据报文。

​    （2）第二次挥手

​     ==被动方此时有可能还有相应的数据报文需要发送==，因此需要先发送ACK报文，告知主动方“我知道你想断开连接的请求了”。这样主动方便不会因为没有收到应答而继续发送断开连接的请求（即FIN报文）。

   （3）第三次挥手

​    ==被动方在处理完数据报文后==，便发送给主动方FIN报文；这样可以保证数据通信正常可靠地完成。发送完FIN报文后，被动方进入LAST_ACK阶段（超时等待）。

   （4）第四挥手

​    如果主动方及时发送ACK报文进行连接中断的确认，这时被动方就直接释放连接，进入可用状态

#### 为什么TIME_WAIT状态需要经过2MSL(最大报文段生存时间)才能返回到CLOSE状态

​	MSL指的是数据包在网络中的最大生存时间。虽然按道理，四个报文都发送完毕，我们可以直接进入CLOSE状态了，但是我们==必须假象网络是不可靠的==，有可以最后一个ACK丢失。所以TIME_WAIT状态就是用来重发可能丢失的ACK报文。

​	避免server端再次发送Fin

#### 三次握手发生在那个API上

socket，connect，send，recv

bind，listen，accept，recv，send，close

**client：**

socket，connect，read/recv，close

**server：**

socket，bind，listen，Accept，send/write，close



read/recv、write/send

都是系统调用：read  --> do_read() --> recv()

socket  --> (fd,tcp)



**滑动窗口**

> 早期的[网络通信](https://baike.baidu.com/item/网络通信)中，通信双方不会考虑网络的拥挤情况直接发送数据。由于大家不知道[网络拥塞](https://baike.baidu.com/item/网络拥塞)状况，同时发送数据，导致中间节点阻塞掉包，谁也发不了数据，所以就有了[滑动窗口机制](https://baike.baidu.com/item/滑动窗口机制)来解决此问题

[滑动窗口协议](https://baike.baidu.com/item/滑动窗口协议)是用来改善吞吐量的一种技术，即容许发送方在接收任何应答之前传送附加的包。接收方告诉发送方在某一时刻能送多少包（称窗口尺寸）。







**慢启动**

**拥塞算法**

**RTT**



