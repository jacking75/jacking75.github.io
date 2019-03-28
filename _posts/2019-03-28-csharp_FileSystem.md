---
layout: post
title: C# - File
published: true
categories: [.NET]
tags: c# net file
---
## 클래스 단위로 파일에 쓰기
이 직렬화 방식은 꼭 .NET 플랫폼에서 서로 파일을 읽고 쓸 때만 사용 가능하다.만약 .NET으로 만든 프로그램에서 아래와 같이 파일을 만들고 이것을 네이티브에서 읽으면 앞에 다른 값이 들어가 있다( 정확하게는 직렬화 되는 클래스의 메타 정보가 들어가 있다 ).  
```
FileStream GloveFile = new FileStream( "0.ipt", FileMode.Create);
BinaryFormatter formatter = new BinaryFormatter();
formatter.Serialize( GloveFile, ItemFileclass );
```
  
  
  
##읽기 전용 파일을 해제 하기
  
```
System.IO.File.SetAttributes("파일 이름", System.IO.FileAttributes.Normal);
```
  
  
  
## 현재 실행 하고 있는 프로그램의 실행 경로 얻기
Environment.CurrentDirectory 를 이용하면 실행 경로를 얻을 수 있다.  
  
  
  
## 응용 프로그램의 현재 작업 디렉토리 얻기
  
```
string CurDir = System.IO.Directory.GetCurrentDirectory();
```
  
  
  
## 응용 프로그램의 현재 작업 디렉토리 변경하기
  
```
System.IO.Directory.SetCurrentDirectory(@"C:\Program Files\Microsoft Visual Studio 9.0\Common7\IDE");
```
  
  
  
## 디렉토리 삭제
  
```
System.IO.Directory.Delete(@"C:\Program Files\Microsoft Visual Studio 9.0\Common7\IDE");
//만약 디렉토리 안의 파일이나 디렉토리까지 지우고 싶다면
System.IO.Directory.Delete(@"C:\Program Files\Microsoft Visual Studio 9.0\Common7\IDE", true);
```
  
  
  
## 하위 폴더에 있는 파일까지 검색
  
```
Directory.GetCurrentDirectory();
string[] FindFiles = Directory.GetFiles(textBox6.Text, textBox1.Text, SearchOption.AllDirectories);
```
  
  
  
## 파일 삭제
  
```
File.Delete(findfile);
```
  
  
  
## 폴더 선택 - 컨트롤 사용
  
```
public void ChooseFolder()
{
    if (folderBrowserDialog1.ShowDialog() == DialogResult.OK) 
    {
        textBox1.Text = folderBrowserDialog1.SelectedPath;
    }
}
```
  
  
  
## 텍스트 파일 한줄 단위로 읽기
  
```
string readfile = "config.txt";
if (false == File.Exists(readfile))
{
        MessageBox.Show(readfile + "파일이 없습니다 !!!");
        return;
}

string readLine;
StreamReader sr = new StreamReader(readfile);
while (sr.Peek() >= 0)
{
        readLine = sr.ReadLine();

        ............

}
```
  
  
  
## 텍스트 파일 쓰기
  
```
// 일반 
new StreamWriter(@"ProtocolDef.cs")

// 아래는 유니코드로
StreamWriter sw = new StreamWriter(@"ProtocolDef.cs", false, Encoding.Unicode);

// 줄 넘김을 하려면 \r\n이 들어가 있어야 한다.  
sw.Write("ㅇㅇㅇ\r\n");

sw.Write("ㅇㅇㅇ");
sw.Close();
```
  
  
  
## 지금 폴더에 있는 폴더와 파일
  
```
string[] AllFiles = Directory.GetFileSystemEntries(textBox1.Text);

foreach (string Entry in AllFiles)
{
      listBox1.Items.Add(Entry);
}
```
  
  
  
## 지금 폴더에 있는 디렉토리와 파일
  
```
DirectoryInfo dir = new DirectoryInfo(textBox1.Text);
FileSystemInfo[] FSIs = dir.GetFileSystemInfos("*.*");
foreach (FileSystemInfo Entry in FSIs)
{
     listBox1.Items.Add(Entry.Name);
}
```
  
  
  
## 지금 폴더에 있는 디렉토리와 이 디렉토리들의 모든 하위 디렉토리 포함
  
```
string[] AllFiles = Directory.GetDirectories(textBox1.Text, "*", SearchOption.AllDirectories);
foreach (string Entry in AllFiles)
{
      listBox1.Items.Add(Entry);
}
```
  
  
  
## 지금 폴더에 있는 디렉토리와 이 디렉토리들의 모든 하위 디렉토리에 있는 모든 파일들
  
```
string[] AllFiles = Directory.GetFiles(textBox1.Text, "*.*", SearchOption.AllDirectories);
foreach (string Entry in AllFiles)
{
      CurVersionlistBox.Items.Add(Entry);
}
```
  
  
  
## 기타 등등
- GetDirectoryName  지정된 경로 문자열에 대한 디렉터리 정보를 반환한다.
- GetExtension  지정된 경로 문자열에서 확장명을 반환한다.
- System.IO.Path.GetFileName  지정된 경로 문자열에서 파일 이름과 확장명을 반환한다.
- GetFileNameWithoutExtension  확장명 없이 지정된 경로 문자열의 파일 이름을 반환한다.
- GetFullPath  지정된 경로 문자열에 대한 절대 경로를 반환한다.
- GetPathRoot  지정된 경로의 루트 디렉터리 정보를 가져온다.
  

