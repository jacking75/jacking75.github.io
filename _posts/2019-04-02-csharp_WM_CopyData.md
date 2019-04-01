---
layout: post
title: C# - WM_COPY_DATA
published: true
categories: [.NET]
tags: c# .net WM_COPY_DATA win32api
---
## 참고 글
- http://www.codeproject.com/KB/cs/ipc_wmcopy.aspx 
- http://www.codeproject.com/KB/threads/InterprocessCommunicator.aspx 
- http://www.codeproject.com/KB/cs/wm_copydata_use.aspx 
- MessageWindow  http://www.codeproject.com/KB/threads/IPCBetManagdUnmanagd.aspx
  
  
  
## 예제 코드
  
```
const int WM_COPYDATA = 0x4A;
[DllImport("user32.dll", CharSet = CharSet.Auto)]
public static extern IntPtr SendMessage(IntPtr hwnd, int msg, IntPtr wparam, IntPtr lparam);
................
//Process[] AppServerProcess = Process.GetProcessesByName("LoginServer");
//.................
//SendMessage(AppServerProcess[0].MainWindowHandle, WM_COPYDATA, IntPtr.Zero, iPtrForCdsMsg);

COPYDATASTRUCT cds;
cds.dwData = srcHandle;
str = str + '\0';
cds.cbData = str.Length + 1;
cds.lpData = Marshal.AllocCoTaskMem(str.Length);
cds.lpData = Marshal.StringToCoTaskMemAnsi(str);
IntPtr iPtr = Marshal.AllocCoTaskMem(Marshal.SizeOf(cds));
Marshal.StructureToPtr(cds, iPtr, true);

// send to the MFC app
SendMessage(destHandle, WM_COPYDATA, IntPtr.Zero, iPtr);

// Don't forget to free the allocated memory 
Marshal.FreeCoTaskMem(cds.lpData);
Marshal.FreeCoTaskMem(iPtr);


//위 방식의 경우 보낼 때 마다 동적 할당을 하므로 메모리가 심하게 조각 날 수도 있다. 그러므로 한번 할당한 것을 계속 사용 하도록 한다.
public struct COPYDATASTRUCT
{
   public IntPtr dwData;
   public int cbData;
   public IntPtr lpData;
}

COPYDATASTRUCT  CdsMsg;
IntPtr iPtrForCdsMsg;


public ServerCommunication()
{
  CdsMsg.dwData = (IntPtr)12;
  CdsMsg.lpData = Marshal.AllocCoTaskMem(1024);
  iPtrForCdsMsg = Marshal.AllocCoTaskMem(Marshal.SizeOf(CdsMsg));
}

~ServerCommunication()
{
   Marshal.FreeCoTaskMem(CdsMsg.lpData);
   Marshal.FreeCoTaskMem(iPtrForCdsMsg);
}

public void ReceiveMsg(System.Windows.Forms.Message m)
{
   COPYDATASTRUCT cds = new COPYDATASTRUCT();
   cds = (COPYDATASTRUCT)Marshal.PtrToStructure(m.LParam, typeof(COPYDATASTRUCT));
   if (cds.cbData > 0)
   {
      byte[] data = new byte[cds.cbData];
      Marshal.Copy(cds.lpData, data, 0, cds.cbData);
      Encoding unicodeStr = Encoding.ASCII;
      char[] myString = unicodeStr.GetChars(data);
      string returnText = new string(myString);
      int iMsgIndex = (int)cds.dwData;
      System.Windows.Forms.MessageBox.Show("ACK Received: " + returnText + iMsgIndex.ToString());
      m.Result = (IntPtr)1;
   }
}

public void SendMsg()
{
   Process[] AppServerProcess = Process.GetProcessesByName("LoginServer");

   if (AppServerProcess.Length < 1)
   {
      return;
   }

   string str = "Hello" + "\0";

   CdsMsg.dwData = AppServerProcess[0].MainWindowHandle;
   CdsMsg.cbData = str.Length;
   CdsMsg.lpData = Marshal.StringToCoTaskMemAnsi(str);
   Marshal.StructureToPtr(CdsMsg, iPtrForCdsMsg, true);
   SendMessage(AppServerProcess[0].MainWindowHandle, WM_COPYDATA, IntPtr.Zero, iPtrForCdsMsg);
}

protected override void WndProc(ref Message m)
{
   switch (m.Msg)
   {
   case WM_COPYDATA:
      ServerComm.ReceiveMsg(m);
      break;

   default:
      // let the base class deal with it
      base.WndProc(ref m);
      break;
   }
}
``` 
  

