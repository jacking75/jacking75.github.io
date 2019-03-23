---
layout: post
title: C# - SharpSVN
published: true
categories: [.NET]
tags: c# .net SharpSVN svn
---
## 설명
* 닷넷에서 SVN을 조작하기 위한 라이브러리
* SVN 서버의 버전과 맞추어야 한다.
* 공식 홈페이지는 http://sharpsvn.open.collab.net/
* 1.7, 1.8 용은 NuGet으로 받을 수 있다.
* SharpSVN 라이브러리는 32비트,64비트로 나누어지는데 선택에 따라서 명시적으로 프로젝트의 32 or 64비트를 결정해야한다. AnyCPU로 하면 빌드 시 경고 발생.
  
  
  
## 사용 법
* 닷넷 프레임워크 4.0 이상에서는 아래의 내용을 설정 파일(app.config)에 입력해야 한다.
  
```
<startup useLegacyV2RuntimeActivationPolicy="true">
    <supportedRuntime version="v4.0"/>
</startup>
```
  
* 체크 아웃, 커밋 예
  
```
using (SvnClient client = new SvnClient())
{
    client.CheckOut(new Uri("svn://172.20.60.201/svn_program_v2/Doc"),
                        svnPath);
    Collection<SvnStatusEventArgs> changedFiles = new Collection<SvnStatusEventArgs>();
    client.GetStatus("D:\\Test\\sharpsvn", out changedFiles);

    //delete files from subversion that are not in filesystem
    //add files to subversion that are new in filesystem
    //modified files are automatically included as part of the commit

    foreach (SvnStatusEventArgs changedFile in changedFiles)
    {
        if (changedFile.LocalContentStatus == SvnStatus.Missing)
        {
            // SVN thinks file is missing but it still exists hence
            // a change in the case of the filename.
            if (System.IO.File.Exists(changedFile.Path))
            {
                SvnDeleteArgs changed_args = new SvnDeleteArgs();
                changed_args.KeepLocal = true;
                client.Delete(changedFile.Path, changed_args);
            }
            else
                client.Delete(changedFile.Path);
        }
        if (changedFile.LocalContentStatus == SvnStatus.NotVersioned)
        {
            client.Add(changedFile.Path);
        }
    }

    SvnCommitArgs ca = new SvnCommitArgs();
    ca.LogMessage = "lol";

    client.Commit("D:\\Test\\sharpsvn", ca);
}
```
  
* 업데이트와 (현재 디렉토리)리비전 번호 알기
  
```
static public bool Update(string dirFullPath)
{
        // dirFullPath는 이미 체크아웃을 한 디렉토리이며, 거북이SVN으로 업데이트도 되는 디렉토리임
    using (var client = new SharpSvn.SvnClient())
    {
                // 업데이트
        bool isResult = client.Update(dirFullPath);

                // 리비전 번호를 얻는다.
        var workingCopyClient = new SharpSvn.SvnWorkingCopyClient();
        SharpSvn.SvnWorkingCopyVersion version;
        workingCopyClient.GetVersion(dirFullPath, out version);
        var revision = version.End;
    }
}
```
  
* 예제 코드 https://gist.github.com/1450092/57f54fdbc4e84bf845a46a225de6145d6d51acdb
  