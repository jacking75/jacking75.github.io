---
layout: post
title: C# - AWS SNS
published: true
categories: [.NET]
tags: c# .net aws sns
---
## AWS SNS 프로그래밍
- 디바이스 토큰: 애플리케이션의 식별자. 각 플래폼이 발행.
- Endpoint Arn: 디바이스 토큰으로 생성하는 모바일 엔드포인터. SNS에서는 이 모바일 포인트를 사용하여 push를 보낸다.
- 가격: 월 100만 요청은 무료. 100만을 초과하면 100만 요청마다 0.5$
- 안드로이드에서 디바이스 토큰이 바뀌는 경우
    - 앱을 재 설치 할 때
    - '데이터 삭제' 되었을 때
    - 앱을 버전업 했을 때  
- 디바이스 토큰이 바뀌면 endpoint가 disable이 된다. 그러므로 주기적으로 조사해야 한다.
    - SNS에서 push 통지가 실패하면 endpoint가 disable이 된다.
    - 한번 disable이 된 endpoint는 이후 SNS에 push 요청을 해도 보내지 않는다.
- 한 topic 당 1000만개까지 서브스크립션 등록 가능하고, 한 계정단 topic 수는 3000개까지. 확장하고 싶다면 신청 해야 한다.\
    - https://aws.amazon.com/ko/sns/faqs/#limits-restrictions
- Q: SNS 모바일 푸시는 알림 서비스의 토큰 피드백을 어떻게 처리합니까?
    - SNS 모바일 푸시는 귀하를 대신해 토큰 피드백을 자동으로 처리하며 소비하도록 선택한 주제에 게시된 이벤트를 통해 해당 피드백 정보를 노출합니다. 이러한 접근은 푸시 알림을 보내는 운영 부담을 줄이고 알림이 전달되는 속도와 안정성을 최대화합니다. APNS 및 GCM 같은 푸시 알림 서비스는 만료되거나 새로운 토큰으로 대체된 토큰에 대한 피드백을 제공합니다. 특정 토큰이 새로운 토큰으로 대체되면 SNS는 관련 Endpoint를 자동으로 업데이트하고 이벤트를 통해 알립니다. 사용자가 앱을 삭제하여 특정 토큰이 만료되는 경우 SNS는 end-point를 비활성화하고 이벤트를 통해 알립니다. SNS를 사용하여 푸시 알림을 보내기 위해 엄격하게 피드백 알림을 소비할 필요는 없지만 광범위한 사용 사례에 따라 그렇게 처리하도록 선택할 수 있습니다.
  
  
  
## 사용 준비
- SNS에서 콘솔에서 Application 항목에서 "Create Platform Application" 한다.
    - 안드로이드, iOS 각각 만들어야 한다.
- Topics을 만든다(그룹 푸쉬를 보내기 위해)
    - Subscription 항목은 따로 설정하지 않아도 된다.
    - Topic에 엔드포인트를 추가하면 자동으로 Subscription에 추가된다.
  
  
  
## Application에 추가하기(엔드 포인트를 얻을 수 있다)
  
```
var Endpoint = "ap-northeast-1";
var AwsAccessKey = "unknow";
var AwsSecretKey = "unknow";
var deviceRegistID = "APA91bE952T7PPK4NM3tnzPeoIbUc2WaaxK0NvZU7fvGw3OcrmG4zM-NSJ41N6nRNPn0_R7X7gLUMMcyQFNPW5YoVXrr0MRCwaNfNfTNelsACcXEsnQlrghUmfQHx46qJRqDMYHsvjKaTOIYjVUK_V_ha3UYQ7fMNg";

try
{
    var client = new Amazon.SimpleNotificationService.AmazonSimpleNotificationServiceClient(AwsAccessKey, AwsSecretKey, Amazon.RegionEndpoint.GetBySystemName(Endpoint));

    var result = client.CreatePlatformEndpoint(new Amazon.SimpleNotificationService.Model.CreatePlatformEndpointRequest()
                    {
                        PlatformApplicationArn = "arn:aws:sns:ap-northeast-1:473637008560:app/GCM/fs_dev_and",
                        Token = deviceRegistID,
                    });

    result.EndpointArn.Dump();
}
catch(Exception ex)
{
    ex.Dump();
}
```
  
  
  
## 토픽에 메시지 보내기(전체 메시지)
  
```
var Endpoint = "ap-northeast-1";
var AwsAccessKey = "unknow";
var AwsSecretKey = "unknow";
var subject = "test343";
var logMessage = "fdfdsfsdfsd"; // 주 메시지 내용
var TopicArn = "arn:aws:sns:ap-northeast-1:473637008560:fs_dev_topic";

try
{
    var client = new Amazon.SimpleNotificationService.AmazonSimpleNotificationServiceClient(AwsAccessKey, AwsSecretKey, Amazon.RegionEndpoint.GetBySystemName(Endpoint));
    //client.Dump();

    client.Publish(new Amazon.SimpleNotificationService.Model.PublishRequest()
                    {
                        Message = logMessage,
                        Subject = subject,
                        TopicArn = TopicArn
                    });
}
catch(Exception ex)
{
    ex.Dump();
}
```
  
  
  
## 한 디바이스에만 보내기
  
```
var Endpoint = "ap-northeast-1";
var AwsAccessKey = "unknow";
var AwsSecretKey = "unknow";
var targetArn = "arn:aws:sns:ap-northeast-1:473637008560:endpoint/GCM/fs_dev_and/952bd2a6-0c18-3469-a2dd-8071e16fa7a9"; // Application에 추가했을 때 반환 받는 endpoint 이다.

try
{
    var client = new Amazon.SimpleNotificationService.AmazonSimpleNotificationServiceClient(AwsAccessKey, AwsSecretKey, Amazon.RegionEndpoint.GetBySystemName(Endpoint));
    //client.Dump();

    client.Publish(new Amazon.SimpleNotificationService.Model.PublishRequest()
                    {
                        Message = "434324523",
                        Subject = "TEST - 123",
                        TargetArn = targetArn,
                    });
}
catch(Exception ex)
{
    ex.Dump();
}
```
  
  
  
### 토픽에 엔드포인트 추가하기
- 중복으로 넣어도 에러가 발생하지 않는다.
  
```
var Endpoint = "ap-northeast-1";
var AwsAccessKey = "unknow";
var AwsSecretKey = "unknow";
var topicArn = "arn:aws:sns:ap-northeast-1:473637008560:fs_dev_topic";
var endpoint = "arn:aws:sns:ap-northeast-1:473637008560:endpoint/GCM/fs_dev_and/5058d06f-460d-3126-9b70-74252519a219";  // Application에 추가했을 때 반환 받는 endpoint 이다.

try
{
    var client = new Amazon.SimpleNotificationService.AmazonSimpleNotificationServiceClient(AwsAccessKey, AwsSecretKey, Amazon.RegionEndpoint.GetBySystemName(Endpoint));

    var result = client.Subscribe(topicArn, "application", endpoint);
    result.SubscriptionArn.Dump(); // 이 값을 저장해서 topic에서 제거할 때 사용한다.
}
catch(Exception ex)
{
    ex.Dump();
}
```
  
  
### 토픽에서 엔드포인트 삭제하기
  
```
var Endpoint = "ap-northeast-1";
var AwsAccessKey = "unknow";
var AwsSecretKey = "unknow";
var subscriptionArn = "arn:aws:sns:ap-northeast-1:473637008560:fs_dev_topic:25959ad0-03af-4c16-a8ed-bb6082ec9c34"; // 토픽에 추가할 때 반환 받았던 SubscriptionArn

try
{
    var client = new Amazon.SimpleNotificationService.AmazonSimpleNotificationServiceClient(AwsAccessKey, AwsSecretKey, Amazon.RegionEndpoint.GetBySystemName(Endpoint));

    var result = client.Unsubscribe(subscriptionArn);
    result.Dump();
}
catch(Exception ex)
{
    ex.Dump();
}
```
  
  
  
### 애플리케이션에서 엔드포인트 삭제하기
- 만약 topic에 등록 되어 있다면 topic에서 자동 삭제 되지 않는다.
  
```
var Endpoint = "ap-northeast-1";
var AwsAccessKey = "unknow";
var AwsSecretKey = "unknow";

try
{
    var client = new Amazon.SimpleNotificationService.AmazonSimpleNotificationServiceClient(AwsAccessKey, AwsSecretKey, Amazon.RegionEndpoint.GetBySystemName(Endpoint));

    var request = new Amazon.SimpleNotificationService.Model.DeleteEndpointRequest
    {
        EndpointArn = "arn:aws:sns:ap-northeast-1:473637008560:endpoint/GCM/fs_dev_and/5058d06f-460d-3126-9b70-74252519a219"
    };

    var result = client.DeleteEndpoint(request);
    result.Dump();
}
catch(Exception ex)
{
    ex.Dump();
}
```
  
  
### 엔드포인트가 유효한지 조사한다
- 만약 없는 Endpoint라면 NotFoundException 예외가 발생
- 있지만 사용할 수 없다면 GetEndpointAttributesResponse의 Attributes 멤버에서(Dictionary 자료형) "Enabled" 값이 false로 되어 있다.
  
```
var Endpoint = "ap-northeast-1";
var AwsAccessKey = "unknow";
var AwsSecretKey = "unknow";

try
{
    var client = new Amazon.SimpleNotificationService.AmazonSimpleNotificationServiceClient(AwsAccessKey, AwsSecretKey, Amazon.RegionEndpoint.GetBySystemName(Endpoint));

    var request = new Amazon.SimpleNotificationService.Model.GetEndpointAttributesRequest
    {
        EndpointArn = "arn:aws:sns:ap-northeast-1:473637008560:endpoint/GCM/fs_dev_and/952bd2a6-0c18-3469-a2dd-8071e16fa7a9"
    };

    var result = client.GetEndpointAttributes(request);

  string value = "";
  if (result.Attributes.TryGetValue("Enabled", out value))
  {
      value.Dump(); // true or false
  }
    result.Dump();
}
catch(Exception ex)
{
    ex.Dump();
}
```
  