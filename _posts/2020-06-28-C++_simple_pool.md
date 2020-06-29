---
layout: post
title: C++ - 간단한 Object 메모리 풀링
published: true
categories: [C++]
tags: c++ memory pool
---
의사 코드  
```
class Player
{
    Player(const int index) {}
};

class UserManager
{
public:
    void NewUser();
    void DeleteUser(const int playerIndex);
private:
    list<shared_ptr<Player>> m_PlayerList;  // 실제 사용 중인 플레이어들 
    vector<shared_ptr<Player>> m_PlyerPool; // Player 객체 pool
    deque<int> m_EmptyPoolIndex;
};


void UserManager::Init(const int MaxUserCount)
{
    for(int i = 0; i < MaxUserCount; ++i)
    {
        m_EmptyPoolIndex.push_back(i);        
        m_PlyerPool.push_back(make_shared<Player>(i);
    }
}

void UserManager::NewUser()
{
    if(m_EmptyPoolIndex.empty())
    {
        return;
    }
    
    auto index = m_EmptyPoolIndex.pop_front();
    m_PlayerList.insert(m_PlyerPool[index]);
}

void UserManager::DeleteUser(const int playerIndex)
{
    m_EmptyPoolIndex.push_back(playerIndex);
    m_PlyerPool[index].Clear();
    m_PlayerList.remove();
}
```
  