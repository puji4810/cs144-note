## Check3 - TCP Sender

### Some details of TCP sender
1. every few milliseconds, call tick(arg)
2. tcpsender constructed. we have an argument to show the initial value of 
RTO(重传时间) RTO will change with time, but the initial value is the same
the initial value of RTO is stored as `initial_RTO_ms_`
3. implement retransmission timer,which can be called to start at a certain
time. The timer goes off once the RTO has elapsed. The time passing comes from
the `tick()` being called
4. sending a segment, if the timer is not running, start the timer.
**'expire'** ,means the timer will run out after a certain time.
5. all outstanding data being acked, stop the timer.
6. `tick()` called and timer has expired:
    1. retransmit the earliest segment has not been fully acked(need to 
    store outstanding segments)
    2. if the windows size is not zero:
        1. if the segment is not acked, retransmit the segment.and recored
        the number of consecutive retransmissions.
        2. double the RTO. 
    3. reset the tier and start it such that it will expire after RTO time
7. when the sender give the ackno
    1. we need to set the RTo back to the **initial value**
    2. if the sender has any outstanding segment, restart the timer
    3. resert the **consecutive retransmissions** to zero.


### Implementing the TCP Sender

1. `void push( const TransmitFunction& transmit );`  
the Tcp Sender reads the stream and send TCPMessage as many as possible,
make sure there are enough space in the window and has new bytes to read.  
it send them by calling `transmit()`  
you need to make sure every TCPMessage fits fully in the receiver's window.
and make each message as big as possible, but not bigger than `TCPConfig::MAX_PAYLOAD_SIZE`  
use `TCPSenderMessage::sequence_length()` to count the total seqno num of a segment.  
**Remember:** the SYN and FIN flags also has a seqno, they occupy space in the window.
> 如果窗口大小为零，我应该怎么做？如果接收方宣布窗口大小为零，`push` 方法应假装窗口大小为 1。发送方可能会发送一个字节，而该字节被接收方拒绝（且未被确认），但这也可以促使接收方发送一个新的确认段，在其中表明其窗口中已经腾出了更多空间。如果不这样做，发送方将永远无法得知它被允许重新开始发送数据。

> 这是针对零大小窗口情况的唯一特殊行为。你的 `TCPSender` 实现不应该实际记住一个虚假的窗口大小为 1。这种特殊情况仅存在于 `push` 方法中。另外，请注意，即使窗口大小为 1（或 20，或 200），窗口仍可能已满。“窗口已满”与“窗口大小为零”并不相同。

2. `void receive( const TCPReceiverMessage& msg );`  
receive a new message, conveying new left and right edge of window(ackno -- ackno + window size)  
TCPSender should look through the message stored, remove the fully acked(the ackno bigger than all the seqno in the segment)

3. `void tick( uint64 t ms since last tick, const TransmitFunction& transmit );`  
a certain number of milliseconds since last time the `tick()` was called.
the sender may retransmit the segment, using the `transmit()` function.

4. `TCPSenderMessage make empty message() const;`  
generate and send a zero-len message with seqno set correctly.
> `TCPSender` 应该生成并发送一个长度为零的消息，同时正确设置序列号。这在对端需要发送一个 `TCPReceiverMessage`（例如，因为它需要确认来自对端发送方的某些内容）并需要生成一个与之匹配的 `TCPSenderMessage` 时非常有用。

> **注意**：像这样的段（segment），由于不占用任何序列号，因此不需要被记录为“未完成的（outstanding）”段，也永远不会被重传。


### Q&A note
1. before sender get the window size by calling `receive()` method, 
just assume the receiver's window size is 1.
2. when we get a ackno, which ack partially some segments, we don't need to
clip off the bytes (actually we **`could`** do, but it is not necessary)
3. if the three segments 'a' 'b' 'c' are missing, we **`could`** do this
4. dont store emoty segments or retransmit them. a segment wit no seq no is
no need to stored or retransmit
