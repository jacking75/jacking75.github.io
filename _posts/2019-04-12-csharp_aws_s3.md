---
layout: post
title: C# - AWS S3
published: true
categories: [.NET]
tags: c# .net aws s3
---
## 개요
- [AWS 공식 문서](https://docs.aws.amazon.com/ja_jp/AmazonS3/latest/dev/RetrievingObjectUsingNetSDK.html)
  
  
  
## S3 객체 만들기
  
```
var config = new AmazonS3Config();
config.RegionEndpoint = RegionEndpoint.APNortheast1;
var s3Client = AWSClientFactory.CreateAmazonS3Client("XXXXXX", "YYYYYYY", config);
```
  
  
  
## 버킷 안의 리스트 보기
  
```
try
{
    var config = new AmazonS3Config();
    config.RegionEndpoint = Amazon.RegionEndpoint.APNortheast1;
    var s3Client = Amazon.AWSClientFactory.CreateAmazonS3Client("XXXXXX", "YYYYYYY", config);

    var listObjectRequest = new ListObjectsRequest();
    listObjectRequest.BucketName = "MMM3dev";
    var response = s3Client.ListObjects(listObjectRequest);

    foreach (var item in response.S3Objects)
    {
        item.Dump();
        //sr.WriteLine("Object Name: " + item.Key);
    }
}
catch (Exception ex)
{
    ex.Dump();
}
```
  
  
  
## 디렉토리에 파일 올리기(디렉토리가 없으면 만든다)
  
```
try
{
    var config = new AmazonS3Config();
    config.RegionEndpoint = Amazon.RegionEndpoint.APNortheast1;
    var s3Client = Amazon.AWSClientFactory.CreateAmazonS3Client("XXXXXX", "YYYYYYY", config);

    var request = new PutObjectRequest()
    {
        BucketName = "MMM3dev",
        Key = "MMM3ServerExtendedModule/FCMChattingServer.exp", // MMM3ServerExtendedModule/ 이게 디렉토리가 된다
        AutoCloseStream = true,
        CannedACL = S3CannedACL.Private,
        InputStream = new FileStream(@"D:\Temp\MMM3ServerExtendedModule\FCMChattingServer.exp", FileMode.Open)
    };

    s3Client.PutObject(request);
}
catch (Exception ex)
{
    ex.Dump();
}
```
  
  
  
## 디렉토리만 만들기
  
```
try
{
    var config = new AmazonS3Config();
    config.RegionEndpoint = Amazon.RegionEndpoint.APNortheast1;
    var s3Client = Amazon.AWSClientFactory.CreateAmazonS3Client("XXXXXX", "YYYYYYY", config);

    var request = new PutObjectRequest()
    {
        BucketName = "MMM3dev",
        Key = "MMM3ServerExtendedModule2/",
        CannedACL = S3CannedACL.Private,
        ContentBody = "MMM3ServerExtendedModule2/"
    };

    s3Client.PutObject(request);
}
catch (Exception ex)
{
    ex.Dump();
}
```
  
  
  
## 파일 다운로드(디렉토리에 있는 파일을 다운로드. 디렉토리를 만들면서)
  
```
try
{
    var config = new AmazonS3Config();
    config.RegionEndpoint = Amazon.RegionEndpoint.APNortheast1;
    var s3Client = Amazon.AWSClientFactory.CreateAmazonS3Client("XXXXXX", "YYYYYYY", config);

    var keyName = "MMM3ServerExtendedModule/FCMChattingServer.exp";
    GetObjectRequest request = new GetObjectRequest
    {
        BucketName = "MMM3dev",
        Key = keyName
    };

    using (GetObjectResponse response = s3Client.GetObject(request))
    {
        string dest = Path.Combine(Environment.GetFolderPath(Environment.SpecialFolder.Desktop), keyName);
        if (!File.Exists(dest))
        {
            response.WriteResponseStreamToFile(dest);
        }
    }
}
catch (Exception ex)
{
    ex.Dump();
}
```
  