---
layout: post
title: golang - erning/gorun 패키지 소개
published: true
categories: [Golang]
tags: go gorun 번역
---
[출처](https://qiita.com/nnao45/items/b8eb4907921e3bd0cd8d)  
  
Github : https://github.com/erning/gorun  
용도는 매우 간단하다. "Go 소스 코드를 Python이나 Ruby 같은 실행 파일로 돌린다."라는 느낌이다.  
아이디어의 승리라는 느낌이다.   
  
hello.go
```
#!/usr/bin/env gorun //go 에 gorun 패키지를 쓰면 완성.

package main

func main() {
    println("Hello world!")
}
```  
<pre>
$ chmod + x hello.go
 $ ./hello.go # "go run hello.go"가 아니어도 실행
Hello world!
</pre>  
  
  
## 내부의 움직임  
소스 코드 : https://github.com/erning/gorun/blob/master/gorun.go  
main 함수부터 살펴 보자. gorun는 세빙을 첫 번째 줄에 박아 놓고 있으므로 실행 파일로 실행해도 내포 되어 있는 동작으로는 gorun hello.go와 다르지 않다.   
소스를 봐도 그냥 첫 번째 인수를 Run 함수에 대입하고 있을 뿐이다.
```
func main() {
    args := os.Args[1:]
...
    err := Run(args)
...
}
```  
  
이어 Run 함수에서는 먼저 인수의 소스 파일에서 실행 용 바이너리를 두기 위해 디렉토리 또는 임시 파일을 RunFile 함수를 사용하여 작성하고 있다.  
```
func Run(args []string) error {
    sourcefile := args[0]
    rundir, runfile, err := RunFile(sourcefile)
...
}

func RunFile(sourcefile string) (rundir, runfile string, err error) {
    rundir, err = RunDir()
    if err != nil {
        return "", "", err
    }
    sourcefile, err = filepath.Abs(sourcefile)
    if err != nil {
        return "", "", err
    }
    sourcefile, err = filepath.EvalSymlinks(sourcefile)
    if err != nil {
        return "", "", err
    }
    runfile = strings.Replace(sourcefile, "%", "%%", -1)
    runfile = strings.Replace(runfile, string(filepath.Separator), "%", -1)
    runfile = filepath.Join(rundir, runfile)
    runfile += ".gorun"
    return rundir, runfile, nil
}
```
  
이 후 파일이 있는지 보고, 시간으로 차등해서 보고 변화가 없는지 보고 이리저리하여 오류 분기 후 문제가 없으면 Compile 함수로 돌진한다.  
```
func Run(args []string) error {
...
    err := Compile(sourcefile, runfile)
...
}
```
  
여기가 움직임의 본진이다.   
보면 알 수 있듯이 진짜로 "pid로 훅하여 go build로 컴파일 하여 실행" 이다.  
```
func Compile(sourcefile, runfile string) (err error) {
    pid := strconv.Itoa(os.Getpid())

    content, err := ioutil.ReadFile(sourcefile)
    if len(content) > 2 && content[0] == '#' && content[1] == '!' {
        content[0] = '/'
        content[1] = '/'
        sourcefile = runfile + "." + pid + ".go"
        ioutil.WriteFile(sourcefile, content, 0600)
        defer os.Remove(sourcefile)
    }

    gotool := filepath.Join(runtime.GOROOT(), "bin", "go")
    if _, err := os.Stat(gotool); err != nil {
        if gotool, err = exec.LookPath("go"); err != nil {
            return errors.New("can't find go tool")
        }
    }

    out := runfile + "." + pid
    err = Exec([]string{gotool, "build", "-o", out, sourcefile})
    if err != nil {
        return err
    }
    return os.Rename(out, runfile)

func Exec(args []string) error {
    cmd := exec.Command(args[0], args[1:]...)
    cmd.Stdout = os.Stdout
    cmd.Stderr = os.Stderr
    err := cmd.Run()
    base := filepath.Base(args[0])
    if err != nil {
        return errors.New("failed to run " + base + ": " + err.Error())
    }
    return nil
}
```
  
패키지 자체는 "Go 언어에서 go run의 움직임을 쉘에서 온디맨드로 재현하고 있다"라는 동작으로 되어 있다.  
    
