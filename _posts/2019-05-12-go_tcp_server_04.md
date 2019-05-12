---
layout: post
title: golang - 04) TCP Server 만들기
published: true
categories: [Golang]
tags: go tcp socket server
---
[출처](https://qiita.com/kawasin73/items/5d93f77fe653417449c9 )   
  
   
## 비동기 Write
  
![](/images/2019/golang_tcp_server_08.png)  
  
main.go  
```
package main

import (
    "context"
    "log"
    "os"
    "os/signal"
    "syscall"

    "github.com/dmmlabo/tcpserver_go/tcp4/server"
)

func main() {
    sigChan := make(chan os.Signal, 1)
    // Ignore all signals
    signal.Ignore()
    signal.Notify(sigChan, syscall.SIGINT)

    svr := server.NewServer(context.Background(), "127.0.0.1:12345")

    err := svr.Listen()

    if err != nil {
        log.Fatal("Listen()", err)
    }

    log.Println("Server Started")

    select {
    case sig := <-sigChan:
        switch sig {
        case syscall.SIGINT:
            log.Println("Server Shutdown...")
            svr.Shutdown()

            svr.Wg.Wait()
            <-svr.ChClosed
            log.Println("Server Shutdown Completed")
        default:
            panic("unexpected signal has been received")
        }
    case <-svr.AcceptCtx.Done():
        log.Println("Server Error Occurred")
        svr.Wg.Wait()
        <-svr.ChClosed
        log.Println("Server Shutdown Completed")
    }
}
```
  
  
server 패키지  
server.go  
```
package server

import (
    "context"
    "log"
    "net"
    "strings"
    "sync"
)

const (
    listenerCloseMatcher = "use of closed network connection"
)

type Server struct {
    addr      string
    listener  *net.TCPListener
    ctx       context.Context
    shutdown  context.CancelFunc
    AcceptCtx context.Context
    errAccept context.CancelFunc
    Wg        sync.WaitGroup
    ChClosed  chan struct{}
}

func NewServer(parent context.Context, addr string) *Server {
    ctx, shutdown := context.WithCancel(parent)
    acceptCtx, errAccept := context.WithCancel(context.Background())
    chClosed := make(chan struct{})
    return &Server{
        addr:      addr,
        ctx:       ctx,
        shutdown:  shutdown,
        AcceptCtx: acceptCtx,
        errAccept: errAccept,
        ChClosed:  chClosed,
    }
}

func (s *Server) Shutdown() {
    select {
    case <-s.ctx.Done():
        // already shutdown
    default:
        s.shutdown()
        s.listener.Close()
    }
}

func (s *Server) Listen() error {
    tcpAddr, err := net.ResolveTCPAddr("tcp", s.addr)
    if err != nil {
        return err
    }

    l, err := net.ListenTCP("tcp", tcpAddr)
    if err != nil {
        return err
    }
    s.listener = l

    go s.handleListener()
    return nil
}

func (s *Server) handleListener() {
    defer func() {
        s.listener.Close()
        close(s.ChClosed)
    }()
    for {
        conn, err := s.listener.AcceptTCP()
        if err != nil {
            if ne, ok := err.(net.Error); ok {
                if ne.Temporary() {
                    log.Println("AcceptTCP", err)
                    continue
                }
            }
            if listenerCloseError(err) {
                select {
                case <-s.ctx.Done():
                    return
                default:
                    // fallthrough
                }
            }

            log.Println("AcceptTCP", err)
            s.errAccept()
            return
        }

        c := newConn(s, conn)
        s.Wg.Add(1)
        go c.handleConnection()
    }
}

func listenerCloseError(err error) bool {
    return strings.Contains(err.Error(), listenerCloseMatcher)
}
```  
  
conn.go
```
package server

import (
    "context"
    "log"
    "net"
)

type Conn struct {
    svr       *Server
    conn      *net.TCPConn
    ctxRead   context.Context
    stopRead  context.CancelFunc
    ctxWrite  context.Context
    stopWrite context.CancelFunc
    sem       chan struct{}
}

func newConn(svr *Server, tcpConn *net.TCPConn) *Conn {
    ctxRead, stopRead := context.WithCancel(context.Background())
    ctxWrite, stopWrite := context.WithCancel(context.Background())
    sem := make(chan struct{}, 1)
    return &Conn{
        svr:       svr,
        conn:      tcpConn,
        ctxRead:   ctxRead,
        stopRead:  stopRead,
        ctxWrite:  ctxWrite,
        stopWrite: stopWrite,
        sem:       sem,
    }
}

func (c *Conn) handleConnection() {
    defer func() {
        c.stopWrite()
        c.conn.Close()
        c.svr.Wg.Done()
    }()

    go c.handleRead()

    select {
    case <-c.ctxRead.Done():
    case <-c.svr.ctx.Done():
    case <-c.svr.AcceptCtx.Done():
    }
}

func (c *Conn) handleRead() {
    defer c.stopRead()

    buf := make([]byte, 4*1024)

    for {
        n, err := c.conn.Read(buf)
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

        wBuf := make([]byte, n)
        copy(wBuf, buf[:n])
        go c.handleEcho(wBuf)
    }
}

func (c *Conn) handleEcho(buf []byte) {
    // do something

    // write
    select {
    case <-c.ctxWrite.Done():
        return
    case c.sem <- struct{}{}:
        defer func() { <-c.sem }()
        for {
            n, err := c.conn.Write(buf)
            if err != nil {
                if nerr, ok := err.(net.Error); ok {
                    if nerr.Temporary() {
                        buf = buf[n:]
                        continue
                    }
                }
                log.Println("Write error", err)
                // write error
                c.stopRead()
            }
            return
        }
    }
}
```
  
  
