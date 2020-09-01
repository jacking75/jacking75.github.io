---
layout: post
title: C# - foreach로 순회 중 List의 요소 삭제하기
published: true
categories: [.NET]
tags: .net c# foreach List .netcore
---  
List에 있는 요소를 foreach로 순회하는 중에 요소를 삭제하는 방법     
삭제할 항목을 다른 컨테이너에 담아 놓고, 뒤에 일괄적으로 삭제한다.  
   
```
List<GameObject> RemoveList;

foreach(GameObject obj in colliderList)
{
    if(obj.currenthp <= 0)
   {
       RemoveList.Add(list);
   }
}

foreach (GameObject obj in RemoveList)
{
    colliderList.Remove(list);
}

RemoveList.Clear();
```  
  