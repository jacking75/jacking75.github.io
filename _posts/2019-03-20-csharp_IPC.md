---
layout: post
title: C# - IPC
published: true
categories: [.NET]
tags: c# .net IPC
---
### 참조에 System.Runtime.Remoting을 추가한다.
  
### IPC로 통신할 데이터 객체
  
```
public class RemoteObject : MarshalByRefObject
{
    static Queue<string> ClientMessage = new Queue<string>();
    static Queue<string> ServerMessage = new Queue<string>();


    public void ServerToClient(string msg)
    {
        ClientMessage.Enqueue(msg);
    }

    public void ClientToServer(string msg)
    {
        ServerMessage.Enqueue(msg);
    }

    public string GetClientMessage()
    {
        if(ClientMessage.Count() < 1)
        {
            return "";
        }

        var msg = ClientMessage.Dequeue();
        return msg;
    }

    public string GetServerMessage()
    {
        if (ServerMessage.Count() < 1)
        {
            return "";
        }

        var msg = ServerMessage.Dequeue();
        return msg;
    }
}
```
  
  
### IPC 통신
  
```
IpcServerChannel ServerChannel; // Server
IpcClientChannel ClientChannel; // Client
RemoteObject IPCObject;

public void InitServer(string serverName)
{
    ServerChannel = new IpcServerChannel(serverName);
    ChannelServices.RegisterChannel(ServerChannel, false);

    RemotingConfiguration.RegisterWellKnownServiceType(typeof(RemoteObject), "IPCDataManager", WellKnownObjectMode.Singleton);

    IPCObject = new RemoteObject();
}

public void InitClient(string serverName)
{
    try
    {
        ClientChannel = new IpcClientChannel();
        ChannelServices.RegisterChannel(ClientChannel, false);

        RemotingConfiguration.RegisterWellKnownClientType(typeof(RemoteObject),
                    string.Format("ipc://{0}/IPCDataManager", serverName));

        IPCObject = new RemoteObject();
    }
    catch (System.Security.SecurityException ex)
    {
    }
}


/// 서버는 클라이언트가 없어도 데이터를 보내고 받을 수 있다.
/// 그러므로 클라이언트는 받은 메시지가 시간 상 오랜된 것이라면 무시해야 한다.
/// 물론 무조건 처리해야 하는 데이터라면 다 처리한다..
public bool ServerSend(string msg)
{
    try
    {
        IPCObject.ServerToClient(msg);
        return true;
    }
    catch
    {
        return false;
    }
}

public string ServerReceive()
{
    try
    {
        var msg = IPCObject.GetServerMessage();
        return msg;
    }
    catch
    {
        return "Disconnected";
    }
}

public bool ClientSend(string msg)
{
    try
    {
        IPCObject.ClientToServer(msg);
        return true;
    }
    catch
    {
        return false;
    }
}

public string ClientReceive()
{
    try
    {
        var msg = IPCObject.GetClientMessage();
        return msg;
    }
    catch
    {
        return "Disconnected";
    }
}
```
  
  
### IPC로 주고 받는 데이터가 클래스/구조체 라면 시리얼라이즈 속성을 붙여줘야 한다.
  
```
[Serializable, StructLayout(LayoutKind.Sequential, Pack = 1)]
public struct IPCPacket
{
    public short PacketIndex;
    public string JsonFormat;
}

public class RemoteObject : MarshalByRefObject
{
    static Queue<IPCPacket> AgentMessage = new Queue<IPCPacket>();
    static Queue<IPCPacket> AppServerMessage = new Queue<IPCPacket>();
    ...
}
```
  


