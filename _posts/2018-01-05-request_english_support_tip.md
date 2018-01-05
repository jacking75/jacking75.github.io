---
layout: post
title: 영어로 기술 문의를 할 때의 영작문 Tips
published: true
categories: [번역]
tags: 번역
---
### 첫 번째: 자신이 하고 싶은 것과 문제를 간결하게 설명한다  
영어는 짧고 알기 쉽게 간단하게 설명하는 것이 매우 중요하다.  
나는 가급적 직접 만든 템플릿에 맞추어 자신의 하려고 하는 것과 발생한 문제를 설명하도록 한다.  
<pre>
I tried to $자신의 하고 싶은 일 however $일어난 문제.
</pre>  
  
#### 자신의 하고 싶은 일의 예
- I tried to browse my upload history on History page....  
  "이력 페이지에서 업로드 이력을 보려고 했는데"  
  Browse를 사용할 때는 어디서 "on~" 라고 말하는 것을 잊지 말도록!  
- I tried to change one of user's permissions from "User" to "Admin"...  
  "어느 유저의 권한을 사용자에서 관리자로 변경하려고 했었지만"  
  Change를 사용할 때는 A에서 B에 와 같이 어떻게 변경했는지 알수 있도록 "from A to B"를 사용한다.  
- I tried to upload a configuration file from my machine to/conf via SSH....  
  "설정 파일을 SSH를 이용하여 자신의 머신에서/conf에 올리라고 했는데"  
  upload나 download를 사용할 때도 From A to B, 그리고 어떻게 전송했는지도 해결의 핵심이 된다.  
    
  
#### 일어난 문제의 사례  
- It's loading forever.  
  "로드 중에서 화면이 바뀌지 않는다"  
- It seems like it's failing.  
  "아무래도 실패한 것 같다"  
  "It seems like"은 매우 중요한 표현이다. 자신의 조작 잘못인지, 버그인지 모를 때 자주 사용한다.  
- I got a following error message.  
  "다음과 같은 에러 메시지가 나왔다"  
  이것도 편리하다. 스스로 무리하게 오류를 설명하지 않고 그대로 에러 메시지를 붙이자.  
- I am unable to save it.   
  "보존할 수 없다"  
  I am unable to도 편리한 표현이다. 동사를 붙이는 것만으로 끝이다.  
  
    
여유가 있으면 "Could you please look into this?"를 붙이면 더 좋은 이미지를 줄 수 있다고 생각한다.   
  
위는 어디까지나 템플릿이다.  문장으로 설명할 수 없을 때도 있다고 생각하므로 필요에 따라서 늘려도 좋지만 "무엇이 하고 싶은지" "무엇을 할 수 없는지"에 집중하여 쓰면 좋다.  
   
     
### 두 번째: 환경을 설명한다
문제가 불거진 조건을 설명할 필요가 있다.  
조건은 문장으로 하면 읽기 힘들기 때문에 항목으로 쓴다. 일례를 보자.  
   
<pre>   
Environment
OS:macOS Sierra 10.12.6
Processor:2.5GHz Intel Core i7
Memory:16 GB 1600 MHz DDR3
File Size:1GB
Browser:Chrome version:59.0.3071.115(Official Build)(64 bit)
</pre>  
  
만약 티켓을 내고 싶은 제품이나 서비스 공개된 Q&A 커뮤니티나 서포트 티켓이 있으면 다른 사람이 어떻게 질문하고 있는지를 한번 보면 좋다고 생각한다.  
여기에서 지원 받는 데 필요한 정보를 찾기도 한다.  
영어로 지원 티켓을 낼 때는 시차도 있으므로 가급적 다음 날까지 해결로 이어질 응답을 얻을 수 있도록 필요한 정보를 처음 티켓을 등록할 때 분명히 명기할 필요가 있다.  
  
  
### 세 번째: 일어난 경위를 설명한다
여기도 역시 문장으로 설명하자면 and 나 then으로 연결된 짧은 문장들이 연이어 나오고, 시계열(과거형과 과거 완료나 과거 진행형)을 올바르게 사용하지 않으면 알기 어렵기 때문에 번호부 항목으로 하고 있다.  
지원 지역에 따라서는 지원 담당도 영어가 두번째 외국어인 경우도 있으므로 번호부의 항목을 적으면 실수가 적다.  
  
<pre>  
Procedure
#1:Go to History page
#2:Click on"Browse History"
#3:The loading icon is shown more than 5 min.
</pre>  
  
이 방법이라면 모두 현재형으로 괜찮다 & 주어가 필요 없으므로 간단하게 전달할 수 있다.  
만약 각 항목의 마지막 글에서 문제를 전하지 않는 경우, 아래의 방법으로 자신이 기대하는 동작을 보충하면 보다 알기 쉽게 피처 리퀘스트로 고려 될 수 있다.  
<pre>
Expected:My upload history to be shown immediately after clicking on"Browse History".
</pre>  
  
자신이 시도한 것도 보충으로 쓸 수 있다.  
<pre>
What I've checked:looked into logs but it didn't seem to have any errors after clicking on"Browse History".
</pre>  
"로그를 확인했지만 "이력을 보기" 클릭 후에 에러가 일어난 것처럼 보이지 않았다"  
  
    
물론 스크린 샷이나 로그를 첨부하면 보다 알기 쉽다고 생각한다.  
  
  
### 네 번째: 마지막으로
마지막으로 어떤 도움이 필요한지 한마디 더 한다.  
<pre>
Could you tell me what is wrong here?
Is there any better way to do this?
Could you help me to resolve this issue?
Could you tell me how to do this?
Could you please help me out?
</pre>  
  
등이 있다.  
  
마지막으로 "Thank you" 나 "Cheers" 나 "Looking forward to hearing from you" 라고 하는 것은 아무리 서비스를 사는 측이라도 매너라고 생각한다. 이런 한마디로 세상은 더 좋아질 것이라고 생각한다.   
  
  
<br>
<br>  

출처: http://qiita.com/ninuyama/items/a7f5b380909500adc2a8