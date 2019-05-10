---
layout: post
title: golang - 02) TCP Server 만들기
published: true
categories: [Golang]
tags: go tcp socket server
---
[출처](https://qiita.com/kawasin73/items/5d93f77fe653417449c9 )   
  
   
# 서버 종료하기
  
```
package main

import (
    "context"
    "log"
    "net"
    "os"
    "os/signal"
    "strings"
    "sync"
    "syscall"
)

const (
    listenerCloseMatcher = "use of closed network connection"
)

func handleConnection(conn *net.TCPConn, serverCtx context.Context, wg *sync.WaitGroup) {
    defer func() {
        conn.Close()
        wg.Done()
    }()

    readCtx, errRead := context.WithCancel(context.Background())

    go handleRead(conn, errRead)

    select {
    case <-readCtx.Done():
    case <-serverCtx.Done():
    }
}

func handleRead(conn *net.TCPConn, errRead context.CancelFunc) {
    defer errRead()

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

func handleListener(l *net.TCPListener, serverCtx context.Context, wg *sync.WaitGroup, chClosed chan struct{}) {
    defer func() {
        l.Close()
        close(chClosed)
    }()
    for {
        conn, err := l.AcceptTCP()
        if err != nil {
            if ne, ok := err.(net.Error); ok {
                if ne.Temporary() {
                    log.Println("AcceptTCP", err)
                    continue
                }
            }
            if listenerCloseError(err) {
                select {
                case <-serverCtx.Done():
                    return
                default:
                    // fallthrough
                }
            }

            log.Println("AcceptTCP", err)
            return
        }

        wg.Add(1)
        go handleConnection(conn, serverCtx, wg)
    }
}

func listenerCloseError(err error) bool {
    return strings.Contains(err.Error(), listenerCloseMatcher)
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

    sigChan := make(chan os.Signal, 1)
    // Ignore all signals
    signal.Ignore()
    signal.Notify(sigChan, syscall.SIGINT)

    var wg sync.WaitGroup
    chClosed := make(chan struct{})

    serverCtx, shutdown := context.WithCancel(context.Background())

    go handleListener(l, serverCtx, &wg, chClosed)

    log.Println("Server Started")

    s := <-sigChan

    switch s {
    case syscall.SIGINT:
        log.Println("Server Shutdown...")
        shutdown()
        l.Close()

        wg.Wait()
        <-chClosed
        log.Println("Server Shutdown Completed")
    default:
        panic("unexpected signal has been received")
    }
}
```
  
  
## 코드 설명
- SIGINT로 서버를 종료 시킨다.
- SIGINT은 ‘Ctrl+C’ 로 발생 시킨다.
  
![](/images/2019/golang_tcp_server_02.png)  
  
  
  
## 서버 흐름도  
  
![](/images/2019/golang_tcp_server_03.png)  
![](/images/2019/golang_tcp_server_04.png)  
![](/images/2019/golang_tcp_server_05.png)  
![](/images/2019/golang_tcp_server_06.png)  
  