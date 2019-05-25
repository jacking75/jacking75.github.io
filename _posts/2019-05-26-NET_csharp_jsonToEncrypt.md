---
layout: post
title: C# - json 데이터 암호화, 복호화 하기
published: true
categories: [.NET]
tags: c# .net json AESEncrypt
---
AESEncrypt로 암호화/복호화 하기  
  
```
public static async Task<RESULT_T> RequestHttpAESEncry<REQUEST_T, RESULT_T>(
                                                    string address,
                                                    string reqAPI, 
                                                    string loginSeq,
                                                    string userID,
                                                    REQUEST_T reqPacket) where RESULT_T : new() 
{         
    var resultData = new RESULT_T();
    
    var api = "http://" + address + "/GameService/" + reqAPI;
       var jsonText = Newtonsoft.Json.JsonConvert.SerializeObject(reqPacket);

    var encryData = AESEncrypt(loginSeq, jsonText);
    var sendData = new REQ_DATA { LSeq = loginSeq, ID = userID, Data = encryData };
    var requestJson = Newtonsoft.Json.JsonConvert.SerializeObject(sendData);

    var content = new ByteArrayContent(Encoding.UTF8.GetBytes(requestJson));
    content.Headers.ContentType = new MediaTypeHeaderValue("application/json");
    
    var Network = new System.Net.Http.HttpClient();
    var response = await Network.PostAsync(api, content).ConfigureAwait(false);

    if (response.IsSuccessStatusCode == false)
    {
           Console.WriteLine("[Error] RequestHttpAESEncry Network");
        return resultData;
    }

    var responeString = await response.Content.ReadAsStringAsync();
    var responseJson = Newtonsoft.Json.JsonConvert.DeserializeObject<RES_DATA>(responeString);

    if (responseJson.Result != NO_ERROR)
    {
           Console.WriteLine("[Error] RequestHttpAESEncry Response Error: {0}", responseJson.Result);
        return resultData;
    }

    var decryptData = AESDecrypt(loginSeq, responseJson.Data);
    resultData = Newtonsoft.Json.JsonConvert.DeserializeObject<RESULT_T>(decryptData);
    return resultData;
}

public static async Task<RESULT_T> RequestHttp<REQUEST_T, RESULT_T>(
                                                string address,
                                                string reqAPI, 
                                                string loginSeq,
                                                string userID,
                                                REQUEST_T reqPacket) where RESULT_T : new() 
{         
    var resultData = new RESULT_T();
    
        var api = "http://" + address + "/GameService/" + reqAPI;
         var requestJson = Newtonsoft.Json.JsonConvert.SerializeObject(reqPacket);

    var content = new ByteArrayContent(Encoding.UTF8.GetBytes(requestJson));
    content.Headers.ContentType = new MediaTypeHeaderValue("application/json");
    
        var Network = new System.Net.Http.HttpClient();
    var response = await Network.PostAsync(api, content).ConfigureAwait(false);

    if (response.IsSuccessStatusCode == false)
    {
        Console.WriteLine("[Error] RequestHttpAESEncry Network");
        return resultData;
    }

    var responeString = await response.Content.ReadAsStringAsync();
    var responseJson = Newtonsoft.Json.JsonConvert.DeserializeObject<RESULT_T>(responeString);
        return responseJson;
}
        
        
#region Crypto
static string AesIV = @"!QAZ2WSX#EDC4RFV";  //16
static string AesKey = @"5TGB&7U(IK<";      // 11
public static string AesLoginDynamicKey = @"d_&^t";      // 5
    
static string AESEncrypt(string dynamincKey, string text)
{
   var comleteAesKey = dynamincKey.Substring(0, 5) + AesKey;

   System.Security.Cryptography.AesCryptoServiceProvider aes = new System.Security.Cryptography.AesCryptoServiceProvider();
   aes.BlockSize = 128;
   aes.KeySize = 128;
   aes.IV = Encoding.UTF8.GetBytes(AesIV);
   aes.Key = Encoding.UTF8.GetBytes(comleteAesKey);
   aes.Mode = CipherMode.CBC;
   aes.Padding = PaddingMode.PKCS7;

   // 문자열을 바이트형 배열로 변환
   byte[] src = Encoding.Unicode.GetBytes(text);

   // 암호화 한다
   using (ICryptoTransform encrypt = aes.CreateEncryptor())
   {
       byte[] dest = encrypt.TransformFinalBlock(src, 0, src.Length);

       // 바이트형 배열에서 Base64형식의 문자열로 변환
       return Convert.ToBase64String(dest);
   }
}

static string AESDecrypt(string dynamincKey, string text)
{
   var comleteAesKey = dynamincKey.Substring(0, 5) + AesKey;

   AesCryptoServiceProvider aes = new AesCryptoServiceProvider();
   aes.BlockSize = 128;
   aes.KeySize = 128;
   aes.IV = Encoding.UTF8.GetBytes(AesIV);
   aes.Key = Encoding.UTF8.GetBytes(comleteAesKey);
   aes.Mode = CipherMode.CBC;
   aes.Padding = PaddingMode.PKCS7;

   // Base64 형식의  문자열에서 바이트형 배열로 변환
   byte[] src = System.Convert.FromBase64String(text);

   // 복호화 한다
   using (ICryptoTransform decrypt = aes.CreateDecryptor())
   {
       byte[] dest = decrypt.TransformFinalBlock(src, 0, src.Length);
       return Encoding.Unicode.GetString(dest);
   }
}
#endregion Crypto

```
  