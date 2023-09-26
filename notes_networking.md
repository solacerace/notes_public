

- [1. TCP v/s UDP latency](#1-tcp-vs-udp-latency)
- [2. Epoll](#2-epoll)
  - [2.1. Epoll advanatges](#21-epoll-advanatges)
  - [2.2. EPoll loop based server and Client](#22-epoll-loop-based-server-and-client)
  - [2.3. EPoll Events to know and listen for](#23-epoll-events-to-know-and-listen-for)
  - [2.3. EPoll Read/Write errno](#23-epoll-readwrite-errno)
  - [2.2. EPoll Level Triggered vs Edge Triggered](#22-epoll-level-triggered-vs-edge-triggered)
  - [2.3. EPoll Timer loop how to](#23-epoll-timer-loop-how-to)
  - [2.3. Simple EPoll Server Single Threaded](#23-simple-epoll-server-single-threaded)
- [TCP Server Client](#tcp-server-client)
- [UDP Server Client](#udp-server-client)
- [EPOLL Based Server client](#epoll-based-server-client)


# 1. TCP v/s UDP latency



# 2. Epoll 
## 2.1. Epoll advanatges

## 2.2. EPoll loop based server and Client

## 2.3. EPoll Events to know and listen for

EpollErrors to listen for
EPOLLERR| EPOLLRDHUP | EPOLLHUP | 

EpollIn connection: EPOLLIN 
EpollOut connection: EPOLLOUT

## 2.3. EPoll Read/Write errno

EAGAIN, EWOULDBLOCK, ECONNRESET, EINTR

```



// check if the event is for writing


if (events & ( EPOLLERR| EPOLLRDHUP | EPOLLHUP ))
{
    // Error event
    // restart connection
    return;
}

if (events & EPOLLOUT)
{
    // TBD - when to check EAGAIN?
    cout << "Writtable" ;
    writeLen = send(fd, buffer.out_pos, buff.size());
}

if (events & EPOLLIN)
{
    // TBD - when to check EAGAIN?
    cout << "Readable" ;
}


do
{
    len = read(fd, inbuffer.in_pos(), sizeofBuffer);
} while (-1 == len && EINTR == errno)

if (len > 0)
{
    // all good 
    inbuffer.incr_pos(len);
}
else if ( (len ==0) || ((len == -1) && (ECONNRESET == errno) )
{
    // Read Error - client disconnected - close the connection
}
else if ( (len == -1) && (EAGAIN == errno || errno == EWOULDBLOCK) )
{
    // all good - nothing to read - start waiting
}

```

## 2.2. EPoll Level Triggered vs Edge Triggered

## 2.3. EPoll Timer loop how to

1. Create a timer_fd
2. Add to the epoll_ctl and register EPOLLIN
3. When the event is generated for timer_fd - read the data - so that it knows the event is actioned.
4. Do the required task - if any
5. go back to the epoll_wait loop.

## 2.3. Simple EPoll Server Single Threaded

1. Listen for incoming connection or incoming read
2. When read.. read the data and write back to the socket. If Write fails or not enough space in buffer. Go back and listen but this time register the EPOLLOUT so that as soon as the write is ready you will get callback.
3. When you get the callback from the epoll_wait check if its for EPOLLOUT.. then send the remaining data.
4. If the callback is to read the data, then read the data.
5. Set the mask again based on if you want to listen for read or write data. 



# TCP Server Client
![Alt text](image.png)


# UDP Server Client

https://stackoverflow.com/questions/23068905/is-bind-necessary-if-i-want-to-receive-data-from-a-server-using-udp


![Alt text](image-1.png)


# EPOLL Based Server client

https://stackoverflow.com/questions/66916835/c-confused-by-epoll-and-socket-fd-on-linux-systems-and-async-threads


![Alt text](image-2.png)

Addition to the diagram
- When recv receives 0 len data. The client connection needs to be closed.
- 

