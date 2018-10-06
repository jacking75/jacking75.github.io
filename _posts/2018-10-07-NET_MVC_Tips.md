---
layout: post
title: ASP.NET MVC - 컨트룰러의 다른 뷰 사용하기, View 컨트롤
published: true
categories: [.NET]
tags: .net c# asp.net mvc
---
### 컨트룰러의 다른 뷰 사용하기

```
// 기본 뷰
public ActionResult Index()
{
	var model = new MailModels();
	....
	return View(model);
}

// 핸들러
public ActionResult SendMail(MailModels model)
{
    ...
	// 기본 뷰를 지정해서 호출한다.
	return View("Index", model);
}
```
  
  

### View 컨트롤
* 드랍다운 리스트에서 선택 항목 얻기. 선택된 항목의 문자열을 얻는다.  

```
@Html.DropDownListFor(m => m.SelMailData, Model.m_ItemList)
```
  
* 모델 유효성 검사  
틀린 경우 메시지를 View에 출력해준다  

```
public class testModels
{
	[Required]
	[Range(0, 59, ErrorMessage = "0과 59 사이에 숫자를 입력해주세요.")]
	public int ReservationTimeMin { get; set; }

	[Required(ErrorMessage = "메시지 입력하세요!")]
	[StringLength(26, ErrorMessage = "메시지 길이가 26자를 넘었습니다.")]
	public string Message { get; set; }
}
```  
  
  
```
<div>
	@Html.Label("메세지")
	@Html.TextBoxFor(m => m.Message, new { size = "60" })
	@Html.ValidationMessageFor(m => m.Message)
</div>
</pre>

* 텍스트박스 크기 설정
<pre>
@Html.TextBoxFor(m => m.Message, new { size = "60" })
</pre>

* 텍스트 박스 readonly 설정
<pre>
@Html.TextBoxFor(m => m.UserID new { @readonly="readonly" })
``` 
  

