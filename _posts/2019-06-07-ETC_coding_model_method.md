---
layout: post
title: 모델이나 메소드에 이름을 붙일 때는 영어의 품사에 주의 하자
published: true
categories: [ETC]
tags: coding
---
## 모델의 이름은 명사로 한다
예: "지불 정보"를 나타내는 모델을 만들고 싶은 경우 
- × Pay
- ○ Payment
  
pay는 동사로 pay의 명사형이 payment 이다.  
Pay 모델이 아니라 Payment 모델을 만들자.  
  
  
예: "종교"를 나타내는 모델을 만들고 싶은 경우
- × Religious
- ○ Religion
  
religious는 형용사로 명사형은 religion 이다.  
Religious 모델이 아니라 Religion 모델을 만들자.
  
  
  
### 두 단어를 연결하여 모델을 만드는 경우는 형용사+명사 or 명사+명사로 하자
예: "첨부 파일"을 나타내는 모델을 만들고 싶은 경우
- × AttachFile
- ○ AttachedFile
  
file은 명사지만 attach는 동사이다.
동사+명사의 모델을 만들면 메서드처럼 보일 수 있다.  
  
첫 단어는 동사가 아니라 형용사를 쓰자. 다만 attach의 형용사가 없으므로 대신 과거 분사형의 attached를 사용한다.  
형용사+명사 뿐만 아니라 UserGroup 등 명사+명사의 모델명을 짓는 것도 OK 이다.  
  
  
  
### 처리를 실행하는 메소드는 동사만 or 동사+명사로 하자
예: "User의 상태를 액티브하게 한다" 메소드를 정의하는 경우
- × user.active
- ○ user.activate
  
active는 형용사이다. 즉 "적극적인" 이라는 상태를 표시한다다.  
"액티브하게 해"라는 동작(변경)을 나타내는 경우는 active 동사형인 activate를 사용하는 것이 적절하다.  
  
  
예: "티켓 기간 만료를 통보하다"라는 메소드를 정의하는 경우
- × ticket.notify_expire
- ○ ticket.notify_expiration
  
expire도 notify도 동사이다. 보통 동사를 거듭 쓰는 용법은 없다.  
동사 뒤에는 목적어, 즉 명사를 가져온다. 때문에 여기서는 notify(동사)+expiration(명사)로 notify_expiration 라는 메소드를 정의한다.  
  
  
  
### 이름을 붙일 때는 가산 명사인지 불가산 명사인지를 의식한다
  
- × informations
- ○ information
  
영어의 명사에는 가산 명사(countable noun)와 불가산 명사(uncountable noun)이 있다.  
가산 명사가 있으면 a dog, two dogs 처럼 두개 이상 있으면 복수형의 s를 붙이지만 불가산 명사는 항상 단수형이다.  
원래 불가산 명사에는 센다는 개념이 없기 때문에 an information 이나 two informations 라는 표현 자체가 불가능하다.  
  
그리고 또 한가지, 소스 코드 상에서 잘 사용될 만한 불가산 명사의 예
- × datas
- ○ data
  
data도 불가산 명사이다.  
엄밀히는 "datum 이라는 단어의 복수형이 data"인데 일상적으로는 data는 불가산 명사로 다루는 것 같다.  
  
    
  
### 해외 사이트를 이용하고 영어를 조사 때의 Tips
  
- 단어 자체의 정의를 확인하고 싶은 경우
    - 단어 + define
    - 예: pay define
- 명사형이나 동사형을 확인하고 싶은 경우
    - 단어 + verb(동사) or noun(명사) or adjective(형용사)
    - 예: pay noun
- 복수형을 확인하고 싶은 경우
    - 단어+plural
    - 예: payment plural
- 가산 명사인지 불가산 명사인지를 조사하고 싶은 경우
    - 단어 + countable uncountable
    - 예: information countable uncountable
- 반대의 의미의 단어를 조사하고 싶은 경우
    - 단어 + opposite
    - 예: active opposite
- 처음에 떠오르는 단어가 확실하지 않을 경우
    - http://thesaurus.com 같은 시소러스(유어 사전) 사이트를 이용
    - 예: http://thesaurus.com/browse/information
  
    
<br>  
<br>  
  
출처: https://qiita.com/jnchito/items/459d58ba652bf4763820  


