---
layout: post
title: MQTT
published: true
categories: [Network]
tags: 번역 mqtt
---
[출처](http://tdoc.info/blog/2014/01/27/mqtt.html )  
  
##MQTT란?
MQTT(MQ Telemetry Transport)는 publish/subscribe 모델을 기반으로 하는 경량 메시지 프로토콜이다. 네트워크가 불안정한 장소에서 동작하기 위한 기능이나 힘이 약한 장치에서 움직이기 위한 경량화 등이 특징이다.  
  
MQ라고 이름이 붙어 있지만, 부하를 분산시키기 위한 이른바 Job Queue는 없다 . 그런 용도로는 AMQP 등을 사용한다.  
  
MQTT는 메시지 배달을 전문으로 하고 있으며, 특히 다수의 Publisher/Subscriber을 가져야 하는 용도로 사용되고 있다. 또한 가벼운 점과 동기 통신인 점을 살려서 실시간 통신이 필요한 용도에 사용되고 있다.  
  
구체적으로는  
- 센서로부터의 데이터 수집
- 메시지 서비스( Facebook Messanger )
- 자동차와 스마트 폰의 정보 동기화( Connected Car from CES 2014 ) 
  
등을 들 수 있다.
  
  
## 역사
MQTT는 원래는 1999년부터 IBM과 Eurotech 사가 책정했다. IBM은 IBM MessageSight 라는 제품을 가지고 있다.  
  
그 후 2011년에 Eclipse Foundation에 MQTT 코드가 기증되고, 또한 2013년에 OASIS라는 국제 표준화 단체에서 표준화가 이루어지게 되었다.  
  
사양은 로열티 무료로 공개 되어 있다.   
  
  
## 특징
MQTT의 특징을 간단하게 정리해 보겠다.  
- 고정 길이 2 바이트 라는 오버 헤드 적음과 처리의 가벼움
- 일대 다, 다 대 다 메시지 배포, 게다가 Topic wildcard 의해 유연성이 높다
- 응용 프로그램의 특성에 따라 세 종류의 QoS를 지정 가능
    - '최고로 한 번'  '적어도 한 번'  '정확하게 한 번'의 세 가지
- 절단이 발생해서 다시 연결 한 후 절단 중인 메시지를 받는 "Durable subscribe"
- Publish 측이 절단된 경우 미리 정해둔 메시지를 흘리는 'Last Will and Testament"
- 나중에 Subscribe 해 온 경우에 보낼 수 있는 마지막 메시지를 저장할 "Retain" 
  
QoS 및 Will 등은 MQTT의 특징적인 기능이다.  
  
또한 MQTT TCP/IP 상에서 동작하고, payload는 사양에서는 특별히 정해져 있지 않다. 문자열을 사용하는 것도 많고, MessagePack을 사용하고 있다는 이야기도 들은 적이 있다.  
  
  
## 경량
MQTT는 고정 길이 헤더가 최소 2바이트로 오버 헤드가 적고, 또한 프로토콜도 간단하다. 따라서 HTTP에 비해 네트워크 대역폭 및 처리 속도에 우수하다. 또한 처리가 적다는 것으로, 소비 전력도 적아서 모바일 기기에도 적합하다.  
  
[IBM의 자료 ](http://www.ibm.com/developerworks/jp/websphere/library/connectivity/ms_mqtt_ws/)에 따르면, HTTP에 비해  
- 트래픽이 10분의 1에서 100분의 1, 즉 10 ~ 100 배의 처리량
- 배터리 소모량이 10분의 1 이하로
  
라고 한다.  
  
또한 HTTP와 같이 연결형이 아니라 일정한 간격으로 heart beat를 보내어 서로 연결 지향 프로토콜로 양방향 통신을 한다. 따라서 즉시 성이 높은 통신이 가능하다.  
  
성능에 대한 [IBM의 자료](http://www-01.ibm.com/support/docview.wss?uid=swg24034416)에서는  
- Linux, 4 x 4-core 2.93GHz Intel Xeon with 32GB of RAM and 10Gbit LAN  
에서  
- 100,000 클라이언트
- 13,000 메시지/sec
  
를  
- 25%의 CPU 
  
였다고 한다. 이것은 가장 가벼운 경우(QoS0)이지만, 가장 무거운 경우(QoS2)에서도 6000 메시지/sec를 30%의 CPU load에서 했다고 한다.  
  
  
## Topic
메시지는 각각 "Topic"을 가지고 있다. Topic은  
``
/r_rudi/second_house/room10/light/watt
``
  
라고 하듯이 / 로 구분된 계층 구조로 되어 있다.  
  
Subscriber 측에서는 구독헐 Topic을 지정한다. 이때 완전 일치가 아니라 Wildcard를 지정할 수 있다.  
  
예를 들어 다음과 같이 지정할 수 있다.  
  
/r_rudi/second_house/room10/light/watt  
 완전 일치 지정  
/r_rudi/second_house/#  
 /r_rudi/second_house/ 아래 모두를 topic  
/r_rudi/second_house/+/light/watt  
 부분 일치 (이 경우 모든 방의 light의 watt를 얻는다)  
   
물론 하나의 subscriber가 여러 topic을 subscribe 할 수 있다.  
  
  
## QoS
MQTT는 메시지 전달에 대해 QoS를 설정할 수 있다.  
  
- QoS 0 : At most once - 최고로 한 번. 도착은 ​​보장하지 않는다.
- QoS 1 : At least once - 적어도 한 번. 중복 가능성이 있다.
- QoS 2 : Exactly once - 정확히 한 번
  
이것은 보내는 메시지 마다 변경할 수 있기 때문에 일반적으로 QoS 0로 도착하지 않는다면 메시지는 반드시 도착할 수 있도록 QoS 1 또는 2로 등이 있다.  
  
QoS 1과 2의 경우는 ACK과 재전송의 경우도 프로토콜 사양에 정의되어 있다.  
  
  
## Durable subscribe
MQTT는 네트워크가 불안정한 상황을 상정하고 있다. 따라서 QoS1와 QoS2의 경우 전송된 메시지를 일정 기간 서버(Broker)에서 보유하고 있어야한다.  
  
Subscriber가 명시적인 DISCONNECT와 UNSUBSCRIBE 없이 갑자기 끊기면 다시 연결이 왔을 때 그 동안의 메시지를 재 전송하는 기능이 있다.  
  
이로 인해 불안정한 서버 및 네트워크에서 작동 할 수 있다.  
  
  
## Will
클라이언트는 서버에 처음 CONNECT 했을 때 Will라는 정보를 추가 할 수 있다. 서버는 클라이언트와 통신 할 수 없는 경우에 이 Will로 지정된 Topic와 Payload를 보낸다.  
  
그러면 Subscriber 측에서 Publish 측이 죽었다는 것을 판단 할 수 있는 것이다.  
  
또한 사활 판정은 일정 간격으로 보내지는 ping에 응답하는지 여부로 판단된다. 이 간격도 연결할 때마다 정해져 있으므로,이 장치는 배터리에 대한 우려 때문에 간격을 길게 할 수 있다.  
  
  
## Retain
Retain 기능은 마지막으로 Publish된 메시지를 MQTT 서버가 보관 해 두었다가 새로운 Subscriber에 메시지를 전달하는 기능이다.  
  
MQTT는 Publish/Subscribe 타입 모델이다. 따라서, Publish된 때 Subscribe한 클라이언트에만 메시지가 전송된다. 따라서 예를 들면 1 시간 마다 업데이트 되는 정보를 얻는 Subscribe로 해도 최대 1 시간은 아무것도 정보를 얻을 수 없는 것이다.  
  
그러나 이 경우에도 Retain 기능을 사용하면 최신 정보를 얻을 수 있다.  
  
  
## 보안
MQTT는 먼저 서버(Broker)에 연결할 때 사용자 이름과 암호를 지정하도록 할 수 있다. 그러나 암호 자체는 일반 텍스트로 보내지기 때문에 SSL을 통해 보안을 구축하게 되어 있다.  
  
또한 Subscriber 인증도 없으므로 이 Topic이 Subscriber에게만 배달 하는 사양은 없다. 다만, 아래의 Mosquitto는 사용자 이름에 대해 Read 및 Write 액세스 관리를 할 수 있는 기능이 구현 되어 있다.  
  
아까 사용자 이름과 암호를 지정한다고 썼지만 이 취급에 대해서는 구현 의존이다. 일반 인증 외에도, LDAP 또는 OAuth를 통해 인증을 한다는 구현도 가능하다.  
  
  
## 구현
프로토콜이 간단하기 때문에 구현이 많다. 단, QoS 0 뿐이거나 이제대로 구현 되어 있는 것은 많지 않다.  
  
그 중 잘 참조되는 것을 열거한다.  
- [Mosquitto](http://mosquitto.org/ )
- [RabbitMQ - RabbitMQ MQTT Adapter](https://www.rabbitmq.com/mqtt.html )
- [Paho - Open Source messaging for M2M](http://www.eclipse.org/paho/ )
- [ActiveMQ Apollo](https://activemq.apache.org/apollo/ )
  
Mosquitto는 C로 쓰여진 가장 오래된 구현으로 거의 모든 기능을 구현하고, 참조 구현으로 처리되는 경우가 많다.  
  
RabbitMQ는 erlang으로 만든 정평 있는 MQ 서버이다. AMQP 등에도 대응하고 있지만, MQTT에도 대응하고 있다.  
  
Paho는 전술한 바와 같이 IBM에서 기증된 구현으로 Java 버전이나 Python, JS 등 다양한 언어에 대응하고 있다.  
  
ActiveMQ Apollo는 MQTTT의 다른 STOMP과 AMQP 등도 할 수 있다.  
  
  
