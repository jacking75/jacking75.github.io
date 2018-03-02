---
layout: post
title: C++ - Windows 에서 hiredis 사용하기
published: true
categories: [C++]
tags: c++ redis hiredis
---  
MS에서 Windows용으로 redis를 포팅한 것이 있어서 여기서 hiredis 윈도우 버전을 구할 수 있다.  
  
아래 글은 MS가 윈도우로 포팅한 redis 프로젝트를 기준으로 한 것인데 얼마전에 hiredis 부분만 따로 떨어져 나온 프로젝트가 공개 되었다(이 글은 2017년 10월 이전이다).  
https://github.com/Microsoft/hiredis  
    
### 입수
- https://github.com/MSOpenTech/redis
  
    
### 빌드
- msvs 디렉토리에서 VS용 솔루션 파일을 연다.
- hiredis와 Win32_Interop 두 개의 프로젝트를 빌드한다.
- 빌드가 무사히 되면 'hiredis.lib', 'Win32_Interop.lib' 파일이 만들어진다.
  
  
### 테스트 코드
- 콘솔 프로젝트를 만든다.
- 솔루션 설정
    - 포함 디렉토리: hiredis 소스 디렉토리를 설정한다.
    - 라이브러리 디렉토리: lib 파일이 출력된 디렉토리를 설정한다.
  
- 코드
  
```
#include <stdio.h>
#include <stdlib.h>
#include <hiredis.h>

#pragma comment(lib, "hiredis.lib")
#pragma comment(lib, "Win32_Interop.lib")

int main(void) {
	redisContext *conn = NULL;
	redisReply *resp = NULL;
	int loop = 0;

	// 접속
	conn = redisConnect("127.0.0.1",  // 접속처 redis 서버
						6379           // 포트 번호
						);

	if ((NULL != conn) && conn->err) {
		// error
		printf("error : %s\n", conn->errstr);
		redisFree(conn);
		exit(-1);
	}
	else if (NULL == conn) {
		exit(-1);
	}

	// Value 얻기
	for (loop = 1; loop < 4; loop++) {
		resp = (redisReply *)redisCommand(conn,          // 컨넥션
			"GET %d", loop // 명령어
		);
		if (NULL == resp) {
			// error
			redisFree(conn);
			exit(-1);
		}
		if (REDIS_REPLY_ERROR == resp->type) {
			// error
			redisFree(conn);
			freeReplyObject(resp);
			exit(-1);
		}
		printf("%d : %s\n", loop, resp->str);
		freeReplyObject(resp);
	}

	// 뒷 정리
	redisFree(conn);
	return 0;
}
```
  
  
### 참고
- cpp로 랩핑하고 싶다면 [이 프로젝트](https://github.com/RedisCppTeam/RedisCpp-hiredis)를 참고하면 좋다.
- 리눅스에서는 [이 글](http://qiita.com/Ki4mTaria/items/d73cf3d244c903d493eb)을 참고한다.
