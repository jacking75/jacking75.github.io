---
layout: post
title: C# - SmtpClient로 메일 보내기
published: true
categories: [.NET]
tags: c# .net SmtpClient
---
닷넷의 SmtpClient 클래스를 사용하여 메일을 보내는 방법이다.  
MSDN의 예제는 빠진 부분이 많아서 그걸 사용하면 제대로 되지 않았다.    
삽질하다가 구글링으로 알아냈다.  
아래 예제는 구글의 Gmail을 사용하는 것으로 했다.  
Gmail의 주소는 smtp.gmail.com 이다.  
포트번호는 587을 사용한다.  
구글에서는 465도 사용 할수 있다고 하지만 사용하면 연결이 되지 않았다.  
  
```
SmtpClient client = new SmtpClient("smtp.gmail.com", 587);
client.UseDefaultCredentials = false; // 시스템에 설정된 인증 정보를 사용하지 않는다.
client.EnableSsl = true;  // SSL을 사용한다.
client.DeliveryMethod = SmtpDeliveryMethod.Network; // 이걸 하지 않으면 Gmail에 인증을 받지 못한다.
client.Credentials = new System.Net.NetworkCredential("구글 아이디", "패스워드");
           
MailAddress from = new MailAddress("jacking12343@gmail.com","최흥배",         System.Text.Encoding.UTF8);
MailAddress to = new MailAddress("jacking@dyon.co.kr");
                       
MailMessage message = new MailMessage(from, to);

message.Body = "This is a test e-mail message sent by an application. ";
string someArrows = new string(new char[] { '\u2190', '\u2191', '\u2192', '\u2193' });
message.Body += Environment.NewLine + someArrows;
message.BodyEncoding = System.Text.Encoding.UTF8;
message.Subject = "test message 2" + someArrows;
message.SubjectEncoding = System.Text.Encoding.UTF8;

try
{
     // 동기로 메일을 보낸다.
     client.Send(message);
               
      // Clean up.
      message.Dispose();
}
catch (Exception ex)
{
      MessageBox.Show(ex.ToString());
}
```
  