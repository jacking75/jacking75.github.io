---
layout: post
title: C++ - CAtlMap에서 KEY 값을 두 개 사용하고 싶을 때
published: true
categories: [C++]
tags: c++ CAtlMap
---
예제 코드
  
```
#include <atlcoll.h>
#include <boost/functional/hash.hpp>  // hash를 만들기 위해 사용

// KEY가 될 유저 정의형
struct FRIENDKEY
{
    union
    {
        struct KEY
        {
            INT32    MyID;
            INT32  FriendID;
        };
        KEY Key;
        INT64 nValue;
    };

    FRIENDKEY() : nValue(0) {}
    FRIENDKEY(INT64 _value) : nValue(_value) {}
    FRIENDKEY(INT32 _MyID, INT32 _FriendID)
    {
        Key.MyID = _MyID;
        Key.FriendID = _FriendID;
    }
};

class FriendKeyTraits : public CElementTraits <FRIENDKEY>
{
public:
    static ULONG Hash(const FRIENDKEY& element) throw()
    {
        boost::hash<INT64> hasher;
        return (ULONG)hasher(element.nValue);
    }

    static bool CompareElements(const FRIENDKEY& a, const FRIENDKEY& b)
    {
        return (a.Key.nMyID == b.Key.MyID && a.Key.FriendID == b.Key.FriendID) ? true : false;
    };
};

struct FRIENDPRESENT
{
    INT32    a;
    INT32 b;
    char c;
    INT16 d;
};

 

typedef CAtlMap<FRIENDKEY, FRIENDPRESENT, FriendKeyTraits > MAP_FRIEND;
MAP_FRIEND m_Frinedlist;

 

bool IsExitFriendPresent( INT32 MyID, INT32 FriendID )
{
    FRIENDKEY FriendKey( MyID, FriendID );
    bool bIsExit = m_Frinedlist.Lookup( FriendKey );
    return bIsExit;
}
```