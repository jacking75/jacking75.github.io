---
layout: post
title: C# - 속성을 사용하여 항목 체크
published: true
categories: [.NET]
tags: c# .net attribute
---
DisplayName과 StringLength, Required 라는 기존 속성을 사용한다.  
Attribute.GetCustomAttribut 메소드로 대상( 이 코드에서는 프로퍼티)에서 속성을 얻을 수 있다.  
    
```  
using System;
using System.Reflection;
using System.ComponentModel;
using System.ComponentModel.DataAnnotations;

public class Program
{
    public static void Main()
    {
        Entity entity = new Entity(){Id = "123456", Name = ""};

        validate(entity);
    }

    private static void validate(object obj)
    {
        foreach(PropertyInfo prop in obj.GetType().GetProperties())
        {
            // 값
            string val = prop.GetValue(obj).ToString();
            // DisplayName 속성 취득
            DisplayNameAttribute dispNameAttr = Attribute.GetCustomAttribute(prop, typeof(DisplayNameAttribute)) as DisplayNameAttribute;
            // StringLength 속성 취득
            StringLengthAttribute lenAttr = Attribute.GetCustomAttribute(prop, typeof(StringLengthAttribute)) as StringLengthAttribute;
            // Required 속성 취득
            RequiredAttribute reqAttr = Attribute.GetCustomAttribute(prop, typeof(RequiredAttribute)) as RequiredAttribute;

            // 조사 처리
            if (lenAttr != null && val.Length > lenAttr.MaximumLength)
            {
                Console.WriteLine(string.Format("{0}({1})는 최대 길이 {2}를 넘었습니다.", dispNameAttr.DisplayName, val, lenAttr.MaximumLength));
            }
            if (reqAttr != null && string.IsNullOrEmpty(val))
            {
                Console.WriteLine(string.Format("{0}는 필수 항목입니다.", dispNameAttr.DisplayName));
            }
        }
    }

    private class Entity
    {
        [DisplayName("ID")]
        [StringLength(5)]
        [Required]
        public string Id {get;set;}

        [DisplayName("なまえ")]
        [StringLength(20)]
        [Required]
        public string Name {get;set;}
    } 
}


```  
결과      
<pre>
ID(123456)는 최대 길이 5를 넘었습니다.
이름은 필수 항목입니다.
</pre>  
  
<br>
<br>  

출처: http://qiita.com/urushibata/items/85ea115e85e7ab8f208f