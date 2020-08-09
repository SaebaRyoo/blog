
### TCP/IP的三次握手四次挥手
SYN 表示建立连接，
FIN 表示关闭连接，
ACK 表示响应，
PSH 表示有
DATA 数据传输，
RST 表示连接重置。

SYN_SENT 状态表示请求连接
SEND_RCVD 服务端被动打开后,接收到了客户端的SYN并且发送了ACK时的状态
ESTABLISHED 的意思是建立连接。表示两台机器正在通信。
CLOSE_WAIT 对方主动关闭连接或者网络异常导致连接中断，这时我方的状态会变成CLOSE_WAIT 此时我方要调用close()来使得连接正确关闭
TIME_WAIT  我方主动调用close()断开连接，收到对方确认后状态变为TIME_WAIT

RST表示连接重置。
**三次握手**
1. 客户端发起SYN_SENT, 发送SYN=1、seq=k（seq为随机数）请求连接
2. 服务器确认SEND_RCVD， 并针对客户端的SYN进行应答，发送SYN=1、 ACK=1、ack=k+1、seq=j（seq为随机数），建立请求连接；
3. 客户端ESTABLISHED,并针对服务器SYN进行应答,发送ACK=1， ack=j+1。服务器接收到ACK，进入ESTABLISHED 状态，连接建立完成

**四次挥手**
1. 客户端状态进入FIN_WAIT=1，发送FIN=M，用来关闭客户端到服务器的数据传送
2. 服务器针对客户端的FIN应答，发送ack=M+1，状态改为CLOSE_WAIT，表示服务器已经接收到关闭请求，但是没有做好准备，然后客户端状态变为FIN_WAIT=2；
3. 服务器发送FIN=N，表示服务器确定数据已经发送完毕。告诉客户端，已准备好关闭连接。服务器状态改为LAST_ACK。
4. 客户端接收FIN_N报文后，知道可以关闭连接，但它爬服务器不知道要关闭，所以还是需要发送ACK=1、ack=N+1。然后进入TIME_WAIT，如果服务端收到ACK后，则就知道可以断开了。客户端等待2MSL 后依然没收到回复，则服务器正常关闭

