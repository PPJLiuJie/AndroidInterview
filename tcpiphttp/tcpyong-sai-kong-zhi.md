TCP使用了一个拥塞窗口和一个拥塞策略来避免拥塞，并在拥塞发生后检测和缓解拥塞。

### 1 拥塞窗口\(congestion window\)

cwnd：congestion window 拥塞窗口

在TCP的流量控制中，说明了发送方的窗口大小是由接收方的窗口大小来决定的。换言之，我们假定了只有接收方能够指示发送方应当使用多大的发送窗口。此时，我们完全忽略了另一个实体，即网络。

除了接收方之外，网络是决定发送方窗口大小的第二个实体。

拥塞窗口的大小取决于网络的拥塞程度，并且动态的在变化。发送方控制拥塞窗口的原则是：只要网络没有出现拥塞，拥塞窗口就再增大一些；只要网络出现拥塞，拥塞窗口就减小一些。

发送方有两种信息：接收方的窗口大小和拥塞窗口大小。发送方窗口的真正大小取两者之中较小的一个。

**发送方窗口大小 = min\(rwnd, cwnd\)**

### 2 拥塞策略

TCP的拥塞策略是基于三个阶段：慢开始、拥塞避免和拥塞检测。

#### 2.1 慢开始

![](/assets/0011.png)

慢开始的增大方式是：指数增大。

拥塞窗口大小（cwnd）从1个最大报文段的长度（MSS）开始。MSS的数值是在连接建立时确定的。每当有一个报文段被确认（ACK），拥塞窗口就增大1个MSS。

慢开始算法开始很慢，但按指数规律增大。假设rwnd远大于cwnd，因此发送窗口大小总是等于cwnd。发送方从cwnd=1MSS开始，这表示发送方只能发送一个报文段，当这个报文段被确认后后，cwnd增加1MSS，即cwnd=2MSS。此时，可以发送两个报文段，对于每一个报文段被确认，cwnd都增加1MSS，即cwnd=4MSS，以此类推，cwnd=8MSS，cwnd=16MSS......

#### 2.2 慢开始门限

慢开始不能无限的继续下去，一定要有一个门限来停止这个阶段。

发送方密切关注一个称为ssthresh（慢开始门限）的变量。当窗口大小达到这个门限时，慢开始阶段就结束了，并进入下一阶段：拥塞避免。

#### 2.3 拥塞避免

![](/assets/0012.png)

拥塞避免的增大方式是：加法增大。

每当**一整个窗口**中的报文段都被确认后，拥塞窗口大小才增加1。“一整个窗口”是指在一个RTT（**一个传输轮次所经历的时间其实就是往返时间RTT**）期间传输的所有报文段。

#### 2.4 拥塞检测

拥塞检测的根据就是：是否有收到报文段的确认（ACK）。

无论是在慢开始阶段还是在拥塞避免阶段，只要发送方检测到了网络拥塞，就会把ssthresh（慢开始门限）设置为出现拥塞时cwnd的一半（但不能小于2MSS），然后重新设置cwnd=1，并重新开始执行慢开始算法。

这样做的目的是要迅速减少主机发送到网络中的分组数，使得发生拥塞的路由器有足够的时间把队列中积压的分组处理完毕。

#### 如下图，用具体数值说明了上述拥塞控制的过程：![](/assets/0013.png)

### 3 快重传和快恢复

#### 3.1 快重传

快重传算法首先要求接收方每收到一个失序的报文段后就立即发出重复确认（是为了使发送方尽早知道有报文段没有到达接收方），而不要等到自己发送数据时才进行捎带确认。

![](/assets/0014.png)

接收方收到了M1和M2后都分别发出了确认。现在假定接收方没有收到M3但接着收到了M4。

显然，接收方不能确认M4，因为M4是收到的失序报文段。根据 可靠传输原理，接收方可以什么都不做，也可以在适当时机发送一次对M2的确认。

但按照快重传算法的规定，接收方应及时发送对M2的重复确认，这样做可以让发送方及早知道报文段M3没有到达接收方。发送方接着发送了M5和M6。接收方收到这两个报文后，也还要再次发出对M2的重复确认。这样，发送方共收到了接收方的四个对M2的确认，其中后三个都是重复确认。

快重传算法还规定，发送方只要一连收到三个重复确认就应当立即重传对方尚未收到的报文段M3，而不必 继续等待M3设置的重传计时器到期。

由于发送方尽早重传未被确认的报文段，因此采用快重传后可以使整个网络吞吐量提高约20%。

#### 3.2 快恢复

与快重传配合使用的还有快恢复算法，其过程有以下两个要点：

1. 当发送方连续收到三个重复确认，就执行“乘法减小”算法，把慢开始门限ssthresh减半。
2. 与慢开始不同之处是现在不执行慢开始算法（即拥塞窗口cwnd现在不设置为1），而是把cwnd值设置为 慢开始门限ssthresh减半后的数值，然后开始执行拥塞避免算法（“加法增大”），使拥塞窗口缓慢地线性增大。

![](/assets/0015.png)

以上内容摘自：

[https://github.com/LRH1993/android\_interview/blob/master/computer-networks/tcpip.md](https://github.com/LRH1993/android_interview/blob/master/computer-networks/tcpip.md)

《TCP/IP协议族》第四版

