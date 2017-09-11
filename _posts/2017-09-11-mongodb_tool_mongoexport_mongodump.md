---
layout: post
title: MongoDB - 데이터 추출 툴 mongoexport 와 mongodump 간단 비교
published: true
categories: [DB]
tags: mongodb mongoexport mongodump
---
MongoDB의 데이터를 출력하는 툴로 mongoexport 와 mongodump가 있다.  
둘 다 mongodb-tools 패키지에 포함되는 도구이다.  
  
mongoexport는 CSV 또는 JSON 텍스트 데이터, mongodump는 BSON 형식의 이진 데이터로 출력한다.  
  
사용법도 데이터 포맷 지정 외에는 거의 똑같이 이용할 수 있다.  
        
| -                           | mongoexport                  | mongodump     |
|-----------------------------|------------------------------|---------------|
| 데이터 포맷                 | CSV/JSON                     | BSON          |
| 인덱스 등의 메타 데이터     | 없다                         | 있다          |
| 데이터 항목(key)의 지정:-f  | 가능                         | 가능          |
| 대상 데이터베이스의 지정:-d | 가능                         | 가능          |
| 대상 컬렉션의 지정:-c       | 가능                         | 가능          |
| 임포트 툴                   | mongoimport                  | mongorestore  |
| 주요 용도                   | 다른 도구에 대한 데이터 연계 | 백업/리스토어 |
  
  
출처: http://dev.classmethod.jp/server-side/db/mongoexport-and-mongodump/