---
layout: post
title: WCF - TCP Sample
published: true
categories: [.NET]
tags: wcf .net c#
---
## 서버
### WCF LIB  

```
namespace WCFServerLib
{
	public interface IClientCallback
	{
		[OperationContract(IsOneWay = true)]
		void SendMessageToClient(string message);
	}
}

namespace WCFServerLib
{
	[ServiceContract(CallbackContract = typeof(IClientCallback))]
	public interface IServerService
	{
		[OperationContract(IsOneWay = true)]
		void RegistClinet(string id);

		[OperationContract]
		void TestUserData(RequestEcho reqEcho);

		[OperationContract]
		string TestSayHello();
	}

	public struct RequestEcho
	{
		public string Message;
	}
}

namespace WCFServerLib
{
	[ServiceBehavior(InstanceContextMode = InstanceContextMode.Single, ConcurrencyMode = ConcurrencyMode.Multiple)]
	public class ServerService : IServerService
	{
		ServerLogic MainLogic;

		ConcurrentDictionary<string, IClientCallback> ClientMap = new ConcurrentDictionary<string, IClientCallback>();



		public ServerService(ServerLogic mainLogic)
		{
			MainLogic = mainLogic;

			MainLogic.ForceDisconnect = this.ForceDisconnect;
			MainLogic.SendPacketClient = this.SendToClient;
			MainLogic.SendPacketAllClient = this.BrodCast;
		}

		public void RegistClinet(string id)
		{
			var ctx = OperationContext.Current;
			var client = ctx.GetCallbackChannel<IClientCallback>();
			IClientChannel channel = client as IClientChannel;

			if (ClientMap.TryAdd(id, client))
			{
				MainLogic.AddClinet(id, channel);

				// 네트워크 통신이 안될 때 호출
				//OperationContext.Current.Channel.Faulted += new EventHandler(Channel_Faulted);
				// 클라이언트의 접속이 우아하게 종료되었을 때 호출
				OperationContext.Current.Channel.Closed += new EventHandler(Channel_Closed);
			}
			else
			{
				//
			}
		}

		public void TestUserData(RequestEcho reqEcho)
		{
		}

		public string TestSayHello()
		{
			return "Hello WCF World!";
		}

		public bool SendToClient(string id, string message)
		{
			IClientCallback client = null;
			if (ClientMap.TryGetValue(id, out client) == false)
			{
				return false;
			}

			client.SendMessageToClient(message);

			return true;
		}


		void Channel_Closed(object sender, EventArgs e)
		{
			Logout((IContextChannel)sender);
		}

		void Logout(IContextChannel channel)
		{
			string sessionId = null;

			if (channel != null)
			{
				sessionId = channel.SessionId;
				var userID = MainLogic.RemoveClient(sessionId);

				if (userID != null)
				{
					IClientCallback client = null;
					ClientMap.TryRemove(userID, out client);
				}
			}
		}

		bool ForceDisconnect(string id)
		{
			IClientCallback client = null;
			if (ClientMap.TryGetValue(id, out client) == false)
			{
				return false;
			}

			IClientChannel channel = client as IClientChannel;
			((ICommunicationObject)channel).Close();
			return true;
		}

		int BrodCast(string message)
		{
			int count = 0;

			foreach (var client in ClientMap.Values)
			{
				var channel = client as IClientChannel;
				if (channel.State != CommunicationState.Opened)
				{
					continue;
				}

				client.SendMessageToClient(message);
				++count;
			}

			return count;
		}             
	}
}
```
  
  
### ServerLogic 클래스

```
namespace ServerLogicLib
{
	public class ServerLogic
	{
		ConcurrentDictionary<string, Client> ClientMap = new ConcurrentDictionary<string, Client>();
		ConcurrentDictionary<string, string> SessionIDMap = new ConcurrentDictionary<string, string>();

		public Func<string, bool> ForceDisconnect;
		public Func<string, string, bool> SendPacketClient;
		public Func<string, int> SendPacketAllClient;


		public void Init()
		{
			InnerMessageHostProgram.Init();
		}

		public void AddClinet(string id, IClientChannel clientChannel)
		{
			var client = new Client();
			ClientMap.TryAdd(id, client);
			SessionIDMap.TryAdd(clientChannel.SessionId, id);

			InnerMessageHostProgram.RegistNewUser(id);
			DevLog.Write(string.Format("[AddClinet] ID:{0}, SessionID:{1}", id, clientChannel.SessionId), LOG_LEVEL.INFO);
		}

		public string RemoveClient(string sessionID)
		{
			string id = null;
			if (SessionIDMap.TryRemove(sessionID, out id) == false)
			{
				return id;
			}

			Client tempClient = null;
			ClientMap.TryRemove(id, out tempClient);

			InnerMessageHostProgram.UnRegistUser(id);
			DevLog.Write(string.Format("[RemoveClinet] ID:{0}, SessionID:{1}", id, sessionID), LOG_LEVEL.INFO);

			return id;
		}
	}
}
```
  
  
### 호스트 프로그램

```
ServiceHost Host = null;
WCFServerLib.ServerService HostService = null;
ServerLogic MainLogic = new ServerLogic();


private void MainForm_Load(object sender, EventArgs e)
{
	InitWCF();
}

private void MainForm_FormClosing(object sender, FormClosingEventArgs e)
{
	Host.Close();
}		

void InitWCF()
{
	MainLogic = new ServerLogic();
	MainLogic.Init();

	HostService = new WCFServerLib.ServerService(MainLogic);

	// host 생성, address 지정
	Host = new ServiceHost(HostService, new Uri("net.tcp://localhost/WCF/ServerService"));
   
	// 종점 설정
	Host.AddServiceEndpoint(
		typeof(WCFServerLib.IServerService),
		new NetTcpBinding(),
		"");

	
	// 호스트 open
	Host.Open();
}
```
  
  

## 클라이언트

### WCF의 IClientCallback 구현

```
namespace TestClient
{
    class ClientCallBackHandler : WCFServerLib.IClientCallback
    {        
        public void SendMessageToClient(string message)
        {
            DevLog.Write(message);
        }

    }
}
```
  
  

### 호스트 프로그램

```
public partial class ClientForm : Form
{
	WCFServerLib.IServerService ServerProxy = null;
	
	private void ClientForm_Load(object sender, EventArgs e)
	{
		InitWCF();
	}

	private void ClientForm_FormClosing(object sender, FormClosingEventArgs e)
	{
		if (ServerProxy != null)
		{
			var channel = (System.ServiceModel.Channels.IChannel)ServerProxy;
			if(channel.State == CommunicationState.Opened)
			{
				channel.Close();
			}
		}
	}

	void InitWCF()
	{
		try
		{
			var callback = new ClientCallBackHandler();

			var factory = new DuplexChannelFactory<WCFServerLib.IServerService>(callback);
			factory.Endpoint.Binding = new NetTcpBinding();
			factory.Endpoint.Contract.ContractType = typeof(WCFServerLib.IServerService);
			factory.Endpoint.Address = new EndpointAddress(textBoxServer.Text);

			ServerProxy = factory.CreateChannel();
		}
		catch (Exception ex)
		{
			DevLog.Write(ex.Message);
		}
	}
	
	
	// 서버에 등록하기
	private void button2_Click(object sender, EventArgs e)
	{
	    if (ServerProxy == null)
		{
			return;
		}
		
		ServerProxy.RegistClinet(textBoxUserID.Text);
	}

	// 서버에 메시지 보내기
	private void button1_Click(object sender, EventArgs e)
	{
		if (ServerProxy == null)
		{
			return;
		}

		var result = ServerProxy.TestSayHello();
	}

        private void button3_Click(object sender, EventArgs e)
        {
            if (ServerProxy == null)
            {
                return;
            }

            var request = new WCFServerLib.RequestEcho 
            { 
                Message = string.Format("[{0}] 클라이언트에서 보냈음^^", DateTime.Now) 
            };
            
            ServerProxy.TestUserData(request);
        }
}
```
  
   


