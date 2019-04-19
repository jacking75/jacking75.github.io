---
layout: post
title: Linux - ������ profile, bashrc �� login shell vs non-login shell �� ����
published: true
categories: [OS]
tags: Linux OS shell bashrc bashrc
---
[��ó](http://webdir.tistory.com/126)  
  
## Shell ��� ����
��밡���� shell ��� ����  
```
cat /etc/shells

  /bin/sh
  /bin/bash
  /sbin/nologin
  /bin/dash  
```
  
���� ������� shell�� Ȯ��  
```
echo $SHELL

  /bin/bash
```
  
  
  
## Login shell vs Non-login shell
�α��� shell�� ����ڰ� �α��� ������ ����Ǵ� shell�� �ǹ��Ѵ�(default bash).  
�α����Ͽ� bash�� ó�� �����Ҷ�(login shell �϶�), ���� ��ũ��Ʈ ���ϵ��� �����Ͽ� ȯ���� �����Ѵ�.  
  
1. /etc/profile  
2. ~/.bash_profile    or    ~/.bash_login    or    ~/.profile  
3. ~/.bashrc  
4. /etc/bashrc  
  
/etc/profile �� .profile�� shell�� bash�� �ƴ϶� �α����ϸ� �ε�Ǿ� ����ǰ� .bashrc �� .bash_login, .bash_profile�� bash shell�� �α��� �Ǿ��� ��츸 ������ �ȴ�.  
  
�α��� shell�� �α׾ƿ��Ҷ� ���� ������ ã�Ƽ� �����Ѵ�.  
<pre>  
~/.bash_logout
</pre>
  
�� �α��� shell�� �α��� ���� ����Ǵ� shell�� ���Ѵ�. GUI ȯ�濡�� �� �͹̳��� ���� ��찡 �̿� �ش��ϸ� sudo bash �� su ���� �͵� �̿� �ش��Ѵ�.  
- /etc/profile �� /etc/bashrc �� ��ü ����ڿ��� ����ȴ�.  
- ~/.bashrc �� ~/.profile, ~/.bash_profile ���� �ش� ����ڿ��Ը� ����ȴ�.   
    
����ڰ� �α����� �ϸ� /etc/profile �� /home/�����/.profile �� ��ũ��Ʈ�� �۵��Ͽ� ����ȴ�. �̴� �������� autoexec.bat ���ϰ� ���� �ƶ��̴�.  



