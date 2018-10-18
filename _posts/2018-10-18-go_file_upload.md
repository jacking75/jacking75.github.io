---
layout: post
title: golang - 파일 업로드 하기
published: true
categories: [Golang]
tags: go upload
---
[출처](https://qiita.com/1000VICKY/items/c55254788b87541cad3b )  
  
Go 언어에서 HTML 페이지에서 파일 업로드 된 경우 서버 측의 임의의 장소에 업로드 된 파일을 저장하는 샘플 코드.  

sever.go  
<pre>
package main


import (format "fmt");
import ("net/http");
import "os";
import "html/template";
import "mime/multipart";
func main () {

    var mux *http.ServeMux;
    mux = http.NewServeMux();
    mux.HandleFunc("/hello", func (writer http.ResponseWriter, r *http.Request) {
        format.Fprintln(writer, "HandleFunc 메소드에 ServeHTTP를 함수와 같은 시그네처 그대로 넘긴다");
    })
    var hf http.HandlerFunc;
    hf = func (writer http.ResponseWriter, request *http.Request) {
        format.Fprintln(writer, "HandlerFunc 타입을 정의 ->Handle 메소드에 넘긴다");
    }
    mux.Handle("/hf", hf);
    mux . HandleFunc("/upload", upload);
    mux . HandleFunc("/", index);
    var myHandler = new(MyHandle);

    mux.Handle("/handle", myHandler);
    // http.Server 구조체의 포인터를 선언
    var server *http.Server;
    // http.Server 오보젝트를 확보
    server = &http.Server{}; // or new (http.Server);
    server.Addr = ":11180";
    server.Handler = mux;
    server.ListenAndServe();
}

type MyHandle struct {
    //일단 속은 빈 것으로
}
func (this *MyHandle) ServeHTTP(writer http.ResponseWriter, request *http.Request) {
    format.Fprintln(writer, "http.Handler 타입 인터페이스를 구현한 오브젝트를 http.Handle 에 넘긴다");
};


func index (writer http.ResponseWriter , request *http.Request) {
    var t *template.Template;
    /*
<!-- 템플릿 안 -->
<!DOCTYPE html>
<!-- template用のhtmlファイル -->
<html>
<head>
    <title>ファイルアップロードテスト</title>
</head>
<body>
<div class="container">
    <h1>ファイルアップロードテスト</h1>
    <form method="post" action="http://localhost:11180/upload" enctype="multipart/form-data">
        <fieldset>
            <input type="file" name="image" id="upload_files" multiple="multiple">
            <input type="submit" name="submit" value="업로드 시작">
        </fieldset>
    </form>
</div>
</body>
</html>
    */
    // 템플릿 로드
    t, _ = template.ParseFiles("template/index.html");
    t.Execute(writer, struct{}{});
}
func upload ( w http.ResponseWriter, r *http.Request) {
    // 이 핸들러 함수에 접근은 POST 메소드만 인정
    if  (r.Method != "POST") {
        format.Fprintln(w, "허가한 메소드");
        return;
    }
    var file multipart.File;
    var fileHeader *multipart.FileHeader;
    var e error;
    var uploadedFileName string;
    var img []byte = make([]byte, 1024);
    // POST된 파일 데이터를 얻는다
    file , fileHeader , e = r.FormFile ("image");
    if (e != nil) {
        format.Fprintln(w, "파일 업로드를 확인하지 못했음");
        return;
    }
    uploadedFileName = fileHeader.Filename;
    // 서버 측에 보존하기 위해 빈 파일을 만든다
    var saveImage *os.File;
    saveImage, e = os.Create("./" + uploadedFileName);
    if (e != nil) {
        format.Fprintln(w, "서버 측에서 파일 저장을 할 수 없다");
        return;
    }
    defer saveImage.Close();
    defer file.Close();
    var tempLength int64 =0;
    for {
        // 몇 byte 읽었는지를 취득
        n , e := file.Read(img);
        // 읽은 바이트 수가 0을 돌려준다면 루프를 나온다
        if (n == 0) {
            format.Println(e);
            break;
        }
        if (e != nil) {
            format.Println(e);
            format.Fprintln(w, "업로드된 파일 데이터의 복사에 실패");
            return;
        }
        saveImage.WriteAt(img, tempLength);
        tempLength = int64(n) + tempLength;
    }
    format.Fprintf(w, "문자열 HTTP로서 출력시킨다");
}
</pre>  


