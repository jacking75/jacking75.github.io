---
layout: post
title: Rust - 편리한 매크로 소개
published: true
categories: [Rust]
tags: rust macro
---
[출처](https://qiita.com/elipmoc101/items/f76a47385b2669ec6db3  )  
   
Rust의 표준 매크로는 하나의 파일에 모두 정리하고 있다.  
문서에서 볼 수 있다.  
  https://doc.rust-lang.org/src/std/macros.rs.html  
일부 매크로는 컴파일러 매직이기도 하고, 보고있는 것만으로도 꽤 재미있다.  
  
  
# Rust의 편리한 매크로 목록
  
## compile_error!
`panic!`은 런타임 오류를 내지만,이것은 컴파일 시에 에러를 낼 수 있는 매크로이다.  
컴파일시 코드에 `compile_error!(어떤 문자열);`이 포함 되어 있으면 즉시 컴파일이 실패한다.  
  
예  
```
macro_rules! foo {
    () => {
        compile_error!("式を渡してね！");
    };
    ($e:expr) => {};
}

foo!();
```
  
이것을 실행하면 `foo!();` 부분이 `compile_error!( "식을 건네주세요!");`로 대체 되고, 컴파일 에러가 된다.  
  
오류 내용은 이런 느낌  
<pre>
error : 식을 건네주세요!
  -> src \ hoge.rs : 13 : 9
   |
13 | compile_error! ( "식을 건네주세요!");
   | ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
...
18 | foo! ();
   | ------- in this macro invocation
</pre>
   
   
   
## env!
`env!`는 환경 변수 등의 값을 취득하기 위한 매크로이다.  
런타임이 아니라 컴파일 시에 값이 대체 된다는 것에 주의해야 한다.  
형식은 문자열 리터럴이다.  
  
사용법  
```
env!( "PATH" ); 
env!( "SystemRoot" );
```
  
Cargo가 정의하는 환경 변수도 얻을 수 있다.  
여기에서 어떤 값이 있는지 쓰고 있다.  
  https://doc.rust-lang.org/cargo/reference/environment-variables.html  
예를 들면 cargo.exe의 위치를 알고 싶을 때는 `env!("CARGO")` 라고 하면  
~/cargo.exe 같은 전체 경로를 얻을 수 있다.  
Cargo에 정의되어 있는 환경 변수는 자작 라이브러리 buildscript를 쓸 때 편리하다.  
  
  
  
## option_env!
조금 전의 `env!`에는 조금 문제가 있는데, `env!( "존재하지 않는 환경 변수 이름")` 처럼 없는 환경 변수를 지정하면 컴파일 에러가 된다.  
만약 환경 변수가 없다면 이쪽의 처리로 분기한다 라는 방법이 없다  
그래서 `option_env!` 이다.  
이것은 `Option<&'static str>` 형식으로 값을 반환한다.  
즉, 환경 변수를 찾지 않아도 컴파일 에러가 되지 않고 `Option::None`을 반환한다.  
  
  
  
## concat_idents!
이것은 식별자를 결합한다.  
식별자는 것이 포인트이다!  문자열을 결합하는 것은 아니다!  
  
공식에서 가져온 예이다.  
```
fn foobar() -> u32 { 23 }

let f = concat_idents!(foo, bar);
println!("{}", f());
```
  
`concat_idents!(foo, bar)`는 컴파일시 `foobar`로 대체한다.  
`concat_idents!(foo, bar, baz)`처럼 여러게 연결도 가능하다.  
  
  
## concat!
이것은 컴파일 시에 전달된 값을 결합하여 문자열로 대체하는 매크로이다.  
  
예  
```
// foo10truebar로 대체! 
concat! ( "foo" ,  10 ,  true ,  "bar" );
```
  
문자열 이외에도 결합 대상으로 할 수 있다.   
리터럴이라면 기본적으로 결합 할 수 있다.  
  
  
## line!
이 매크로는 열으면 매크로가 위치한 줄 번호로 대체된다.  
형식은 부호없는 정수 리터럴이다.  
```
fn  main () { 
  let  hoge:u32 = line!();  // hoge = 2

  let  huga:u32 = line!();  // huga = 4 
}
```
  
  
## column!
이것은 앞의 `line!`의 열 번호 버전이다  
  
  
## file!
이것은 이 매크로가 배치 되어 있는 파일의 경로를 문자열 리터럴로 대체한다.  
전체 경로가 아닌 프로젝트에서 상대 경로이다.  
  
  
## stringify!
이 매크로는 어떤 무언가를 그대로 문자열로 만들어 버리는 것이다.  
  
예  
```
stringify!(1+1);     // "1+1"
stringify!(foobar);  // "foobar"
stringify!(file!()); // "file ! (  )"

// "fn main (  ) { println ! ( "{}" , 1 + 2 ) }"
stringify!(fn main(){println!("{}",1+2 )})
```
  
  
## include_str!
컴파일 시 파일에 적혀 있는 문자열로 대체한다.  
사전에 적당히 파일을 준비한다.  
  
hoge.txt  
```
abcd
efgh
매크로 최고!
```
  
그리고 이렇게 된다  
```
const s: &str = include_str!("hoge.txt");

fn main() {
    println!("{}", s);
}
```
  
이제 hoge.txt가 출력된다.  
  
  
## include_bytes!
include_str!의 bytes 버전이다.  
  
  
## module_path!
현재 위치하고 있는 모듈까지의 모듈 경로를 컴파일 시에 문자열로 전개한다.  
  
```
mod foo{
    pub mod bar{
        pub fn baz(){
            println!(module_path!());    
        }
    }
}

fn main() {
    foo::bar::baz();
}

/*  project_name::foo::bar 가 표시된다*/
```  
  
  
## include!
include_str!는 파일의 내용을 문자열 리터럴로 전개하지만, 이것은 이대로 전개다.
C 언어 include와 비슷하다.  
   
hello.rs  
```
println!( "hello!" )
```
  
```  
fn  main() { 
  include!( "hello.rs" ); 
}
```
  
컴파일 하면 `include!( "hello.rs")` 이 `println!( "hello!")` 로 대체되고, 실행하면 hello!가 출력된다.  
  
  
## try!
일단 튜토리얼의 URL을 보자.  
  http://rust-lang-ja.github.io/the-rust-programming-language-ja/1.6/book/error-handling.html  
`try!`는 Result의 내용을 추출을 unwrap 보다 안전하게, 패턴 매치를 쓰지 않고 간결하게 할 수 있다.  
  
예를 보인 것이 빠를 것이다.  
```
fn foo() -> Result<i32, ()> {
    Result::Ok(777)
}

fn bar() -> Result<i32, ()> {
    let num = try!{foo()};
    println!("{}", num);
    Ok(num)
}

fn main() {
    bar();
}

/* 777가 표시된다 */
```
  
이렇게 하여 `try!`를 사용하면 내용물을 쉽게 꺼낼 수 있다.  
unwrap와 다른 점은 값이 Err 라고해도 panic!이 되지 않고 return 되는 것이다.  
  
아마 bar 함수의 try! 부분은 이렇게 바뀐다  
```
fn bar() -> Result<i32, ()> {
    let num = match foo() {
        Ok(x) => x,
        Err(x) => return Err(x),
    };
    println!("{}", num);
    Ok(num)
}
```
  
  
## ? 연산자
바로 앞에 try! 매크로를 소개했지만 이것을 단순화한 연산자가 `?` 연산자이다.  
이 연산자는 try!의 신텍스슈거 구문으로 식을 괄호로 감싸는 번거로움을 없앨 수 있다.  
  
`try!`에서 보여준 bar 함수를 `?` 연산자로 고쳐 쓴 예  
```
fn bar() -> Result<i32, ()> {
    let num = foo()?;
    println!("{}", num);
    Ok(num)
}
```
  
