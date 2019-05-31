---
layout: post
title: C# - SharpZipLib을 사용한 복수 개의 파일 압축 및 해제
published: true
categories: [.NET]
tags: c# .net SharpZipLib
---
SharpZipLib : http://icsharpcode.net/OpenSource/SharpZipLib/Default.aspx    
  
```
// 압축
void Compression()
{
    try
    {
        string zipPath = "test.zip";
        System.IO.FileStream writer = new System.IO.FileStream( zipPath,
                                        System.IO.FileMode.Create,
                                         System.IO.FileAccess.Write, System.IO.FileShare.Write);
       
        ICSharpCode.SharpZipLib.Zip.ZipOutputStream zos =
                            new ICSharpCode.SharpZipLib.Zip.ZipOutputStream(writer);
       
        foreach (string file in DiffFiles)
        {
            int Substringindex = textBox2.Text.Length;
            string f = file.Substring(Substringindex + 1);
           
            ICSharpCode.SharpZipLib.Zip.ZipEntry ze =
                                     new ICSharpCode.SharpZipLib.Zip.ZipEntry(f);

            System.IO.FileStream fs = new System.IO.FileStream( file,
                             System.IO.FileMode.Open, System.IO.FileAccess.Read,
                                  System.IO.FileShare.Read);
           
            byte[] buffer = new byte[fs.Length];
            fs.Read(buffer, 0, buffer.Length);
            fs.Close();
           
            ze.Size = buffer.Length;
           
            ze.DateTime = DateTime.Now;

            // 새로운 엔트리(파일)을 넣는다.
            zos.PutNextEntry(ze);
           
            // 쓰기
            zos.Write(buffer, 0, buffer.Length);
        }

        zos.Close();
        writer.Close();
    }
    catch (Exception ex)
    {
        MessageBox.Show(ex.ToString());
    }
}


// 해제
void DeCompression(string filename)
{
    string zipPath = filename;
    string extractDir = Environment.CurrentDirectory;

    System.IO.FileStream fs = new System.IO.FileStream( zipPath,
                                         System.IO.FileMode.Open,
                                 System.IO.FileAccess.Read, System.IO.FileShare.Read);
   
    ICSharpCode.SharpZipLib.Zip.ZipInputStream zis =
                            new ICSharpCode.SharpZipLib.Zip.ZipInputStream(fs);

    ICSharpCode.SharpZipLib.Zip.ZipEntry ze;
   
    while ((ze = zis.GetNextEntry()) != null)
    {
        if (!ze.IsDirectory)
        {
            string fileName = System.IO.Path.GetFileName(ze.Name);

            string destDir = System.IO.Path.Combine(extractDir,
                             System.IO.Path.GetDirectoryName(ze.Name));
           
            if (false == Directory.Exists(destDir))
            {
                System.IO.Directory.CreateDirectory(destDir);
            }

            string destPath = System.IO.Path.Combine(destDir, fileName);

            System.IO.FileStream writer = new System.IO.FileStream(
                            destPath, System.IO.FileMode.Create,
                                    System.IO.FileAccess.Write,
                                        System.IO.FileShare.Write);

            byte[] buffer = new byte[2048];
            int len;
            while ((len = zis.Read(buffer, 0, buffer.Length)) > 0)
            {
                writer.Write(buffer, 0, len);
            }

            writer.Close();
        }
    }

    zis.Close();
    fs.Close();
}
```
  