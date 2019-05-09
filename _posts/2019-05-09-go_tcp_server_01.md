---
layout: post
title: golang - 01) TCP Server 만들기
published: true
categories: [Golang]
tags: go tcp socket server
---
[출처](https://qiita.com/kawasin73/items/5d93f77fe653417449c9 )   
  
   
# Echo Server 예제 코드 
  
```
package main

import (
        "log"
        "net"
)

func handleConnection(conn *net.TCPConn) {
        defer conn.Close()

        buf := make([]byte, 4*1024)

        for {
                n, err := conn.Read(buf)
                if err != nil {
                        if ne, ok := err.(net.Error); ok {
                                switch {
                                case ne.Temporary():
                                        continue
                                }
                        }
                        log.Println("Read", err)
                        return
                }

                n, err = conn.Write(buf[:n])
                if err != nil {
                        log.Println("Write", err)
                        return
                }
        }
}

func handleListener(l *net.TCPListener) error {
        defer l.Close()
        for {
                conn, err := l.AcceptTCP()
                if err != nil {
                        if ne, ok := err.(net.Error); ok {
                                if ne.Temporary() {
                                        log.Println("AcceptTCP", err)
                                        continue
                                }
                        }
                        return err
                }

                go handleConnection(conn)
        }
}

func main() {
        tcpAddr, err := net.ResolveTCPAddr("tcp", "127.0.0.1:12345")
        if err != nil {
                log.Println("ResolveTCPAddr", err)
                return
        }

        l, err := net.ListenTCP("tcp", tcpAddr)
        if err != nil {
                log.Println("ListenTCP", err)
                return
        }

        err = handleListener(l)
        if err != nil {
                log.Println("handleListener", err)
        }
}
```
  
  
## 코드 설명
- AcceptTCP(), Read(buf) 각각의 내부 구현(libexec/src/net/tcpsock.go)을 보면, 발생할 수 있는 오류는 syscall.EINVAL, *net.OpError 중 하나이다.
- *net.OpError는 func(e *OpError) Timeout() bool 과 func (e *OpError) Temporary () bool 메소드를 가지고 있으며, 복구 가능한 오류를 쉽게 알 수 있다.
- *net.OpError는 net.Error을 충족하므로 net.Error로 캐스팅 하여 오류 유형을 판별한다.
  
  
  
## 에코 서버 흐름도  
  
![](/images/2019/golang_tcp_server_01.png)  
  