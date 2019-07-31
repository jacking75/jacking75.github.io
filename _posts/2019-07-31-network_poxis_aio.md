---
layout: post
title: posix aio의 네트워크 소켓을 지원에 대해서
published: true
categories: [C++]
tags: c++ posix aio linux network
---
posix의 비동기 IO의 API인 aio는 파일 조작만 지원하는 걸로 알고 있었는데 소켓 조작에도 사용할 수 있다고 한다.  
http://koyo.kr/46  
  
그런데 리눅스쪽 서버 개발 관련해서 자료를 찾을 때 소켓 통신으로 aio를 사용하는 것을 본적이 없다.  
(boost.asio나 node.js의 libuv도 사용하지 않고 있다)  
  
만약 소켓 통신에 사용할 수 있다면 왜 리눅스쪽 서버 개발에서 사용 사례가 거의 없는 이유가 궁금하여서 찾아 보았다.  
  
  
아래는 구글링해서 찾은 글이다.    
> POSIX 의 AIO 즉 asynchronous I/O 규약에 있읍니다. 그럼 AIO 를 쓸 일이지 event ports를 왜 쓰는가 라는 질문이 또 나올 수 있겠죠. AIO 는 완료 통보를 받을 때,  완료 수신 방법-1 : 그때마다 별도의 쓰레드를 실행시켜서 받습니다. 아니면,  완료 수신 방법-2 : signal 로 받습니다  
http://jeremyko.blogspot.kr/2012/10/linux-asynchronous-io.html   
   
> SIGEV_THREAD 라는 값을 사용해서, 완료 통보가 오면, aio_completion_handler() 라는 함수를 사용하는 쓰레드 하나를 생성시켜서 받겠다고 알려 주고 있읍니다. 결과적으로 완료 통보 갯수만큼 쓰레드가 생기는, 아주 무식한 상황(?)이 발생하게 됩니다.  
'완료 수신 방법-2'의 경우, 즉 완료 통보를 signal 로 받는 경우, 이때는 또 뭐가 문제냐면, signal 로 받을려면, 특정 signal 번호를 지정해서 그 번호의 signal 로 받겠다고 알려 주어야 하는데, 임의로 사용할 수 있는 signal 번호가 얼마 되지 않습니다. 몇 십개 밖에 안됩니다. 그래서 재수없게 다른 프로세스에서 사용하는 signal 번호를 설정해 버리면, 완료 통보가 엉뚱한 프로세스로 전달되어 버립니다. 그리고 signal 관련 함수에서는 GQCSEx() 처럼 한꺼번에 여러 개의 작업을 가져오는 함수가 없습니다. 따라서 '완료 수신 방법-1'과 '완료 수신 방법-2' 모두 고성능 소켓 처리에는
적합하지 않습니다. 이것을 해결하는 것이 event ports 입니다. event ports 는 여러 개의 완료 통보를 port_getn() 함수로 한꺼번에 받을 수가 있읍니다.  
  
> posix aio는 유저레벨 aio입니다~ 즉 라이브러리 레벨에서 에뮬레이션하기 때문에 posix 호환 시스템에서는 다 쓸 수 있습니다. 근데 linux의 경우 2.6 커널 이후로는 kernel mode aio가 들어간거로 아는데요 libaio가 이를 활용한다고 하는데... 저도 써본적은 없습니다 ㅎㅎ
  
  
------  
다른 글을 찾아보니 위 글과 비슷했다. kernel aio는 socket 지원을 안하는 것 같고, 성능도 epoll 쪽이 더 낫다고 한다  
http://stackoverflow.com/questions/6497217/c-how-to-use-both-aio-read-and-aio-write/6503619#6503619  
  