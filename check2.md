## Check2 - TCP Receiver

### Some details of TCP receiver

receiver 需要告知 
- ackno(acknowledgment number) 给 sender
- windows size 滑动窗口大小，the available capacity of the output ByteStream。窗口大小表示接收方的缓冲区还有多少空间可以存储新的数据。

> Together, the ackno and window size describe describes the receiver’s window: a range of
indexes that the TCP sender is allowed to send.

可接受范围即为[ackno, ackno + windows size)

- 以及三个难点：
  - seqno只有32位，stream有可能超出32位的大小。当seqno来到 index_(2^32-1) ,next byte就是index_0。
  - TCP的seqno是一个随机值Initial Sequence Number(ISN,represents the zero point of SYN) 字节流中，第一个字节就是ISN+1(mod 2^32),依次往后,bala bala
  - SIN和FIN位也会占据seqno
- SYN 标志的序列号是 **ISN**。数据流中的第一个字节的序列号是 **ISN + 1**。
- FIN 标志的序列号紧接在最后一个数据字节之后。假设最后一个数据字节的序列号是 `N`，那么 FIN 标志的序列号就是 **N + 1**。
> 注意，在数据流中，索引是从0开始，需要使用 `abs_seqno - 1` 计算。

### receive():
  message.SYN --->  记录ISN，后面就是payload
  message.FIN --->  结束，需要记录 然后传给reassamber的时候就是is_last_substring=True
  message.RST --->  有错误 连接aborted

