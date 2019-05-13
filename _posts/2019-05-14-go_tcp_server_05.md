---
layout: post
title: golang - 05) TCP Server 만들기
published: true
categories: [Golang]
tags: go tcp socket server
---
[출처](https://qiita.com/kawasin73/items/5d93f77fe653417449c9 )   
  
   
## Graceful Shutdown
  
![](/images/2019/golang_tcp_server_09.png)  
![](/images/2019/golang_tcp_server_10.png)  
![](/images/2019/golang_tcp_server_11.png)  
![](/images/2019/golang_tcp_server_12.png)  
![](/images/2019/golang_tcp_server_13.png)  
  
  
main.go  
```
package main

import (
    "context"
    "log"
    "os"
    "os/signal"
    "syscall"

    "github.com/dmmlabo/tcpserver_go/tcp5/server"
)

func main() {
    chSig := make(chan os.Signal, 1)
    // Ignore all signals
    signal.Ignore()
    signal.Notify(chSig, syscall.SIGINT, syscall.SIGTERM, syscall.SIGQUIT, syscall.SIGHUP)

    host := loadConf()
    svr := server.NewServer(context.Background(), host)
    err := svr.Listen()

    if err != nil {
        log.Fatal("Listen()", err)
    }
    log.Println("Server Started")

    for {
        select {
        case sig := <-chSig:
            switch sig {
            case syscall.SIGINT, syscall.SIGTERM:
                log.Println("Server Shutdown...")
                svr.Shutdown()

                svr.Wg.Wait()
                <-svr.ChClosed
                log.Println("Server Shutdown Completed")
            case syscall.SIGQUIT:
                log.Println("Server Graceful Shutdown...")
                svr.GracefulShutdown()

                svr.Wg.Wait()
                <-svr.ChClosed
                log.Println("Server Graceful Shutdown Completed")
            case syscall.SIGHUP:
                log.Println("Server Restarting...")

                host = loadConf()

                svr, err = svr.Restart(context.Background(), host)
                if err != nil {
                    log.Fatal(err)
                }
                log.Println("Server Restarted")
                continue
            default:
                panic("unexpected signal has been received")
            }
        case <-svr.AcceptCtx.Done():
            log.Println("Server Error Occurred")
            svr.Wg.Wait()
            <-svr.ChClosed
            log.Println("Server Shutdown Completed")
        }
        return
    }
}

var first = true

func loadConf() string {
    // TODO: load config from file or env
    if first {
        first = false
        return "127.0.0.1:12345"
    } else {
        return "127.0.0.1:12346"
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
    AcceptCtx context.Context
    errAccept context.CancelFunc
    Wg        sync.WaitGroup
    ChClosed  chan struct{}
    ctx       *contexts
    chCtx     chan *contexts
}

type contexts struct {
    ctxShutdown context.Context
    shutdown    context.CancelFunc
    ctxGraceful context.Context
    gshutdown   context.CancelFunc
}

func NewServer(parent context.Context, addr string) *Server {
    ctx := newContext(parent)
    acceptCtx, errAccept := context.WithCancel(context.Background())
    chClosed := make(chan struct{})
    chCtx := make(chan *contexts, 1)
    return &Server{
        addr:      addr,
        AcceptCtx: acceptCtx,
        errAccept: errAccept,
        ChClosed:  chClosed,
        ctx:       ctx,
        chCtx:     chCtx,
    }
}

func newContext(parent context.Context) *contexts {
    ctxShutdown, shutdown := context.WithCancel(parent)
    ctxGraceful, gshutdown := context.WithCancel(context.Background())
    return &contexts{
        ctxShutdown: ctxShutdown,
        shutdown:    shutdown,
        ctxGraceful: ctxGraceful,
        gshutdown:   gshutdown,
    }
}

func (s *Server) Shutdown() {
    select {
    case <-s.ctx.ctxShutdown.Done():
        // already shutdown
    default:
        s.ctx.shutdown()
        s.listener.Close()
    }
}

func (s *Server) GracefulShutdown() {
    select {
    case <-s.ctx.ctxGraceful.Done():
        // already shutdown
    default:
        s.ctx.gshutdown()
        s.listener.Close()
    }
}

func (s *Server) Restart(parent context.Context, addr string) (*Server, error) {
    if addr == s.addr {
        // update contexts. not close listener
        prevCtx := s.ctx
        s.ctx = newContext(parent)
        select {
        case <-s.chCtx:
            // clear s.chCtx if previous contexts have not been popped
        default:
        }
        s.chCtx <- s.ctx
        prevCtx.gshutdown()
        return s, nil
    } else {
        // create new listener
        nextServer := NewServer(parent, addr)
        err := nextServer.Listen()
        if err != nil {
            return nil, err
        }
        s.GracefulShutdown()
        s.Wg.Wait()
        <-s.ChClosed
        return nextServer, nil
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

    ctx := s.ctx

    for {
        conn, err := s.listener.AcceptTCP()

        select {
        case ctx = <-s.chCtx:
            // update ctx if changed
        default:
        }

        if err != nil {
            if ne, ok := err.(net.Error); ok {
                if ne.Temporary() {
                    log.Println("AcceptTCP", err)
                    continue
                }
            }
            if listenerCloseError(err) {
                select {
                case <-ctx.ctxShutdown.Done():
                    return
                case <-ctx.ctxGraceful.Done():
                    return
                default:
                    // fallthrough
                }
            }

            log.Println("AcceptTCP", err)
            s.errAccept()
            return
        }

        c := newConn(s, ctx, conn)
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
    "sync"
)

type Conn struct {
    svr       *Server
    ctx       *contexts
    conn      *net.TCPConn
    ctxRead   context.Context
    stopRead  context.CancelFunc
    ctxWrite  context.Context
    stopWrite context.CancelFunc
    sem       chan struct{}
    wg        sync.WaitGroup
}

func newConn(svr *Server, ctx *contexts, tcpConn *net.TCPConn) *Conn {
    ctxRead, stopRead := context.WithCancel(context.Background())
    ctxWrite, stopWrite := context.WithCancel(context.Background())
    sem := make(chan struct{}, 1)
    return &Conn{
        svr:       svr,
        ctx:       ctx,
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
    case <-c.ctx.ctxShutdown.Done():
    case <-c.svr.AcceptCtx.Done():
    case <-c.ctx.ctxGraceful.Done():
        c.conn.CloseRead()
        c.wg.Wait()
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
        c.wg.Add(1)
        go c.handleEcho(wBuf)
    }
}

func (c *Conn) handleEcho(buf []byte) {
    defer c.wg.Done()
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
                c.stopWrite()
            }
            return
        }
    }
}
```
  
  
