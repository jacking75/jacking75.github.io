---
layout: post
title: golang - 브로드캐스팅 하는 에코 서버 예제 코드
published: true
categories: [Golang]
tags: go tcp socket server
---
## 서버
  
```
package main

import (
    "bufio"
    "fmt"
    "log"
    "net"
    "os"
)

func main() {
    if len(os.Args) != 2 {
        log.Fatalf("Usage: %s <port>\n", os.Args[0])
    }

    port := os.Args[1]
    server := NewServer(port)
    server.ListenAndServe()
}

type message string

type Server struct {
    port         string
    clients      []*Client
    addClient    chan *Client
    removeClient chan *Client
    broadcast    chan message
}

func NewServer(port string) *Server {
    return &Server{
        port,
        make([]*Client, 0),
        make(chan *Client),
        make(chan *Client),
        make(chan message),
    }
}

func (s *Server) ListenAndServe() error {
    l, err := net.Listen("tcp", ":"+s.port)
    if err != nil {
        return err
    }
    defer l.Close()

    go s.Serve()

    for {
        conn, err := l.Accept()
        if err != nil {
            log.Printf("Error: %s\n", err)
        }
        c := NewClient(conn, s)

        s.addClient <- c

        go c.Read()
        go c.Write()
    }

}

func (s *Server) Serve() {
    for {
        select {
        case c := <-s.addClient:
            log.Println("Join: " + c.conn.RemoteAddr().String())
            s.clients = append(s.clients, c)
        case c := <-s.removeClient:
            for i := range s.clients {
                if s.clients[i] == c {
                    s.clients = append(s.clients[:i], s.clients[i+1:]...)
                    log.Println("Leave: " + c.conn.RemoteAddr().String())
                    break
                }
            }
        case m := <-s.broadcast:
            for _, c := range s.clients {
                log.Printf("Broadcast (%s): \"%s\"\n", c.conn.RemoteAddr(), m)
                c.channel <- m
            }
        }
    }
}

type Client struct {
    conn    net.Conn
    server  *Server
    channel chan message
    done    chan bool
}

func NewClient(conn net.Conn, server *Server) *Client {
    return &Client{
        conn,
        server,
        make(chan message),
        make(chan bool),
    }
}

func (c *Client) Read() {
    scanner := bufio.NewScanner(c.conn)
LOOP:
    for scanner.Scan() {
        select {
        case <-c.done:
            break LOOP
        default:
            m := scanner.Text()
            log.Printf("Receive (%s): \"%s\"\n", c.conn.RemoteAddr(), m)
            c.server.broadcast <- message(m)
        }
    }

    if err := scanner.Err(); err != nil {
        log.Printf("Error (read): %s\n", err)
    }
    c.server.removeClient <- c
    c.done <- true
}

func (c *Client) Write() {
LOOP:
    for {
        select {
        case <-c.done:
            break LOOP
        case m := <-c.channel:
            log.Printf("Send (%s): \"%s\"\n", c.conn.RemoteAddr(), m)
            _, err := fmt.Fprintln(c.conn, m)
            if err != nil {
                log.Printf("Error (write): %s\n", err)
                break LOOP
            }
        }
    }

    c.server.removeClient <- c
    c.done <- true
    return
}
```
  
  
  
## Client
    
```
package main

import (
    "net"
    "os"
)

const (
    RECV_BUF_LEN = 1024
)

func main() {
    if len(os.Args) == 1 {
        println("need request parameter")
        os.Exit(1)
    }
    echo_contents := os.Args[1]
    tcp_addr, err := net.ResolveTCPAddr("tcp", "localhost:6666")
    if err != nil {
        println("error tcp resolve failed", err.Error())
        os.Exit(1)
    }
    tcp_conn, err := net.DialTCP("tcp", nil, tcp_addr)
    SendEcho(tcp_conn, echo_contents)

    echo := GetEcho(tcp_conn)
    println("echo: ", string(echo))
    println("receive success")
    tcp_conn.Close()
}

func SendEcho(conn *net.TCPConn, msg string) {
    _, err := conn.Write([]byte(msg))
    if err != nil {
        println("Error send request:", err.Error())
    } else {
        println("Request sent")
    }
}

func GetEcho(conn *net.TCPConn) string {
    buf_recever := make([]byte, RECV_BUF_LEN)
    _, err := conn.Read(buf_recever)
    if err != nil {
        println("Error while receive response:", err.Error())
        return ""
    }
    return string(buf_recever)
}
```  
  
