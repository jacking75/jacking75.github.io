---
layout: post
title: C# - scriptcs-nancy
published: true
categories: [.NET]
tags: c# .net ScriptCs nancy
---
## 개요
- [scriptcs-nancy](https://github.com/adamralph/scriptcs-nancy)
- NancyFX 라이브러리를 사용할 수 있게 해준다.
- 중요. 2015-09-15
    - ConcurrentDictionary<string, AgentData> AgentStatus 와 같이 클래스(혹은 구조체)를 컨테이너에 저장하면 모듈 로딩 에러가 발생하므로 객체를 담는 컨테이너 사용불가
  
  
  
## 설치 및 간단 사용
- 명령창을 관리자 권한으로 실행한다
- 폴더를 하나 생성한다. 예) mkdir C:\hellonancy.
- 위에 생성한 폴더로 이동 후 다음의 명령어를 실행한다.
    - scriptcs -install ScriptCs.Nancy
- start.csx 라는 파일을 생성 후 아래의 내용을 입력한다.
    - Require<NancyPack>().Get("/", _ => "Hello world").Host();
- start.csx 스크립트를 실행한다.
- 브라우져에서 다음 주소로 이동한다 http://localhost:8888/
  
  
  
## 간단 설명
- Go()와 Stop() 으로 호스팅 시작과 중단을 한다.
- Host()를 호출하면 스크립팅은 대기 상태에 들어간다. 이것은 내부적으로 Go()를 호출 후 사용자가 key를 누를 때까지 대기 하다가 Stop()을 호출한다.
- 기본 URL은 http://localhost:8888/  이다. 만약 다른 주소로 바꾸고 싶거나 복수의 URL's를 지정할 수 있다.
- 모든 기본 URL's는 시작할 때 콘솔 창에 출력된다.
- The most commonly used Nancy namespaces are also imported into your script/session:
  
<pre>
Nancy
Nancy.Bootstrapper
Nancy.Conventions
Nancy.Cryptography
Nancy.ErrorHandling
Nancy.Hosting.Self
Nancy.ModelBinding
Nancy.Security
Nancy.Validation

You can import more namespaces with your own using statements
</pre>
  
  
  
## 활용
### In-line routes
  
```
var n = Require<NancyPack>();
n.Get("/", _ => "Hello world");
n.Post("/", _ => ... ); // do something else cool
```
  
- DELETE, GET, OPTIONS, PATCH, POST and PUT을 지원하고, async도 지원
  
  
### Custom URL's
  
```
At(777)                    // http://localhost:777
At(777, 888)               // http://localhost:777 and http://localhost:888
At("http://localhost/abc/")
At("http://localhost/abc/", "http://localhost/def/")
```
  
  
### Interactive browsing
- Browse()는 기본 웹 브라우져에서 root URL로 이동한다(or the first URL if multiple URLs are being used).
- BrowseAll() opens a browser window for each root URL when multiple URLs are being used.
- To browse to a specific resource, pass the relative address of the resource as an argument, e.g. Browse("hello") or BrowseAll("hello").
  
  
### Managing the host yourself
- If you don't want to take advantage of the built in hosting in ScriptCs.Nancy, you can also manage the lifetime of the host yourself:
  
```
using (var host = new NancyHost(new DefaultNancyPackBootstrapper(), new Uri("http://localhost:8888/")))
{
    host.Start();    
    Console.ReadKey();
}
```
  
- Note that DefaultNancyBootstrapper is not suited to scriptcs. DefaultNancyPackBootstrapper inherits from DefaultNancyBootstrapper and is specifically designed to work with scriptcs.
  
  
  
## 예제
### 초 간단 호스팅
  
```
Require<NancyPack>().Get("/", _ => "Hello world").Host();
// Require<NancyPack>().Get(g => g["/"] = _ => "Hello world").Host();
// Require<NancyPack>().Module(m => m.Get["/"] = _ => "Hello world").Host();
```
  
  
### 서버 실행 후 자동으로 브라우져에서 열기
  
```
var n = Require<NancyPack>().Get("/", _ => "Hello world").Go().Browse();
```
  
  
### 간이 루팅 시 뷰 파일 지정
  
```
Post["/"] = _ => ["Index.cshtml","I'm a Model"];
```
  
  
### 간이 루팅 시 html 포맷 문자열 반환
  
```
Get["/login"] = _ =>
        {
            return new Nancy.Responses.HtmlResponse()
            {
                Contents = (s) =>
                {
                    using (var sw = new StreamWriter(s, System.Text.Encoding.UTF8))
                    {
                        sw.Write(@"
                            <html>
                            <head>
                            <title>ログイン</title>
                            </head>
                            <body>
                            <form action=""/login?RedirectUrl=/secure"" method=""post"">
                            <label for=""username"">ユーザー名</label>
                            <input type=""text"" name=""username"" />
                            <input type=""submit"" />
                            </form>
                            </html>
                            ");
                    }
                }
            };
        };
```
  
  
### 동적 라우팅
  
```
Get["/Hello/{name}"] = route => "Hello,"+route.name+"!";
```
  
  
### 조건절 요청
  
```
Get["/", context =>
   {
   return context.Request.Headers["user-agent"].Any(agent =>
   {
       return agent.Contains("iOS") || agent.Contains("Android");
   });
}] = _ => "Mobile Page";
```
  
- Context.Request.Form["myname"]
  
  
### Uri 지정
  
```
Require<NancyPack>().Get("/", _ => "Hello world");

using (var host = new NancyHost(new DefaultNancyPackBootstrapper(), new Uri("http://localhost:8888/")))
{
    host.Start();    
    Console.ReadKey();
}
```
  
```
var address = "http://localhost:1234/";

var host = new NancyHost(new Uri(address));
host.Start();

Console.WriteLine("Nancy is running at " + address);
Console.WriteLine("Press any key to end");
Console.ReadKey();

host.Stop();
```
  
  
### 수동 라우팅
  
```
Require<NancyPack>().Host();

public interface IGreeter
{
    string Greeting { get; }
}

public class Greeter : IGreeter
{
    private int count;

    public string Greeting
    {
        get { return "Hello World! We've said hello " + (++count).ToString() + " time(s)." ; }
    }
}

public class IndexModule : NancyModule
{
    public IndexModule()
    {
        Get["/"] = _ =>    View["index"]; // located in views folder
    }
}

public class HelloModule : NancyModule
{
    public HelloModule(IGreeter greeter)
    {
        Get["/hello"] = _ => greeter.Greeting;
    }
}
```
  
  
### NancyModule 클래스 사용
  
```
public class AdminModule : Nancy.NancyModule
{
    public AdminModule()
        : base("/admin")
    {
        Get["/"] = _ => "/admin です";
        Get["/login"] = _ => "/admin/login です";
        Get["/logout"] = _ => "/admin/logout です";
    }
}
```
  
  
### 모델과 뷰 구조 분리
  
<pre>
modules
  home.csx
views
  home.html
</pre>
  
```
//home.csx
public HomeModule : NancyModule
{
    public HomeModule()
    {
        Get["/"] = _ => View["index"];
    }
}
//home.html contains something which looks :cool:.
```
  
```
//nancy.csx
#load "modules\home.csx"
Require<NancyPack>.Host();
```
  
  
### 요청 속성 얻기
  
```
public class SampleModule : Nancy.NancyModule
{
    public SampleModule()
    {
        Get["/"] = _ => "message: " + Request.Query.message;

        Post["/"] = _ => "message: " + Request.Form.message;
    }
}
```
  
```
public class ProductModule : Nancy.NancyModule
{
    public ProductModule()
    {
        Get["/product/{id}"] = parameters => "id: " + parameters.id;
    }
}
```
  
  
### Bootstrapper 사용
  
```
Require<NancyPack>().Use(new CustomBootstrapper()).Host();

public interface IGreeter
{
    string Greeting { get; }
}

public class Greeter : IGreeter
{
    private readonly string greeting;

    public Greeter(string greeting)
    {
        this.greeting = greeting;
    }

    public string Greeting
    {
        get { return this.greeting; }
    }
}

public class IndexModule : NancyModule
{
    public IndexModule(IGreeter greeter)
    {
        Get["/"] = _ => greeter.Greeting;
    }
}

public class CustomBootstrapper : DefaultNancyPackBootstrapper
{
    protected override void ApplicationStartup(Nancy.TinyIoc.TinyIoCContainer container, IPipelines pipelines)
    {
        base.ApplicationStartup(container, pipelines);
        container.Register(typeof(IGreeter), (f, o) => new Greeter("Hello World!"));
    }
}
```
  
  
### Razor ViewEngine 설치
- 사용 불가
  
  
### 구조화된 예제
- https://github.com/scriptcs/scriptcs-samples/tree/master/nancy
  
  
### NancyFX
#### 파라미터 값 얻기
  
```
Get["/files/{id}/{name}"] = _ =>

{

     return Response.AsJson<User>(new User { Id = _.id, Name = _.name });

};
```
  
<pre>
-->
{

Id: 5555,
Name: "hello"
}
</pre>
  
  
### 부트스트랩퍼
- 낸시 어플리케이션이 시작할 때 공통적으로 수행할 환경을 제공. 마치 Global.asax 처럼, 자바나 .NET 의 Spring 에 Application-Context.xml 처럼.
- 없는 페이지 요청했을 때의 처리를 커스텀마이즈
  
```
using Nancy;

namespace NancyLecture
{
    public class Boot : DefaultNancyBootstrapper
    {
        protected override void ApplicationStartup(TinyIoC.TinyIoCContainer container, Nancy.Bootstrapper.IPipelines pipelines)
        {
            base.ApplicationStartup(container, pipelines);
            pipelines.OnError += (context, ex) =>
            {
                return "Sorry. We got Error :-(";
            };
        }
    }
}
```
  
  
### Json으로 답변 보내기
  
```
Get["/json"] = _ => {
    var data = new {
        Id = 1,
        Name = "rabitarochan",
        BlogUrl = "http://rabitarochan.hatenablog.com/"
    };

    return Response.AsJson(data);
};
```
  
<pre>
{
    Id: 1,
    Name: "rabitarochan",
    BlogUrl: "http://rabitarochan.hatenablog.com/"
}
</pre>
  
  
### Json.NET을 json 직렬화 라이브러리로 사용
- https://github.com/NancyFx/Nancy.Serialization.JsonNet
  
```
public class CustomJsonSerializer : JsonSerializer
{
    public CustomJsonSerializer()
    {
        this.ContractResolver = new CamelCasePropertyNamesContractResolver();
    }
}
```
  
```
public class Bootstrapper : DefaultNancyBootstrapper
{
    protected override void ConfigureApplicationContainer(Nancy.TinyIoc.TinyIoCContainer container)
    {
        base.ConfigureApplicationContainer(container);

        container.Register(typeof(JsonSerializer), typeof(CustomJsonSerializer));
    }
}
```
  
  
### 클라이언트가 보낸 json 데이터 풀기
- ModelBinding을 이용한다
  
```
namespace NancyJsonTest.Modules
{
    using Nancy;
    using Nancy.ModelBinding; // 클래스로 변환하기 위한 확장 메소드 용

    public class HomeModule : NancyModule
    {
        // form 값용 클래스
        class TestForm
        {
            // input type=text name=userName
            public string UserName { get; set; }

            // input type=password name=password
            public string Password { get; set; }
        }

        public HomeModule()
        {
            Get["/json"] = _ => {
                var data = new {
                    Id = 1,
                    Name = "rabitarochan",
                    BlogUrl = "http://rabitarochan.hatenablog.com/"
                };

                return Response.AsJson(data);
            };

            Post["/json"] = _ => {
                var form = this.Bind<TestForm>();

                return Response.AsJson(form);
            };        
        }
    }
}
```
  
  
  
## 자작 예제
### 기본
  
```
Require<NancyPack>().Get("/", _ => "Hello world");

using (var host = new NancyHost(new DefaultNancyPackBootstrapper(), new Uri("http://localhost:8888/")))
{
    host.Start();    
    Console.ReadKey();
}
```
  
  
### module + jquery
- Content 디렉토리에 jquery 파일이 있어야 한다
  
```
Require<NancyPack>().Host();

public class IndexModule : NancyModule
{
    public IndexModule()
    {
        Get["/"] = _ =>    View["index"]; // located in views folder
    }
}
```
  
```
// Views\index.html
<!DOCTYPE html>
<html lang="kor">
  <head>
    <meta charset="utf-8">
    <title>NancyFx 튜토리얼</title>

    <script type="text/javascript" src="../Content/jquery-2.1.4.min.js"></script>
    </head>

  <body>
    <h1>Hello NancyFx !!</h1>

    <script type="text/javascript">
    $(document).ready(function(){
     $("#msgid").html("This is Hello World by JQuery");
    });
    </script>
    <div id="msgid">
    </div>
  </body>
</html>
```
  
  
### 멀티 url's
  
```
Require<NancyPack>().Host();

public class IndexModule : NancyModule
{
    public IndexModule()
    {
        Get["/"] = _ =>    View["index"]; // located in views folder
            Get["/test"] = _ =>    View["test"];
    }
}
```
  
```
\\ Views\test.html
<!DOCTYPE html>
<html lang="kor">
  <head>
    <meta charset="utf-8">
    <title>NancyFx 튜토리얼 2</title>
    </head>

  <body>
    <h1>Hello NancyFx !!</h1>
    </body>
</html>
```
  
  
### 입력 정보 받기
  
```
Require<NancyPack>().Host();

public class IndexModule : NancyModule
{
    public IndexModule()
    {
        Get["/"] = _ =>    View["index"]; // located in views folder
        Get["/test04"] = _ =>    View["test04", this.Request.Url];
    }
}
```
  
```
\\ Views\test04.html
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.0 Transitional//EN">
<html>
    <head>
        <title>Nancy - Static view served by self-host</title>
    </head>
    <body>
        <h1>Static view served by self-host</h1>
        <p>
            This view was served by the Nancy self-host.
        </p>
        <ul>
            <li>Scheme: @Model.Scheme</li>
            <li>HostName: @Model.HostName</li>
            <li>Port: @Model.Port</li>
            <li>BasePath: @Model.BasePath</li>
            <li>Path: @Model.Path</li>
        </ul>
        <a href="http://localhost:8888/nancy/">http://localhost:8888/nancy/</a><br/>
        <a href="http://localhost:8888/nancy/testing">http://localhost:8888/nancy/testing</a><br/>
        <a href="http://127.0.0.1:8898/nancy/">http://127.0.0.1:8898/nancy/</a><br/>
        <a href="http://127.0.0.1:8898/nancy/testing">http://127.0.0.1:8898/nancy/testing</a><br/>
        <a href="http://localhost:8889/nancytoo/">http://localhost:8889/nancytoo/</a><br/>
        <a href="http://localhost:8889/nancytoo/testing">http://localhost:8889/nancytoo/testing</a><br/>

        <script type="text/javascript">
        if (true)
        {
            <h1><span class="label label-success">Running...</span></h1>
        }
        </script>
    </body>
</html>
```
  
  
### React.JS
  
```
Require<NancyPack>().Host();

public class IndexModule : NancyModule
{
    public IndexModule()
    {
        Get["/"] = _ =>    View["reactJS01"];
    }
}
```
  
```
\\ Content\app.js
var generateId = (function() {
  var id = 0;
  return function() {
    return '_' + id++;
  }
})();

var todos = [{
  id: generateId(),
  name: 'Buy some milk'
}, {
  id: generateId(),
  name: 'Birthday present to Alice'
}];

var TodoStorage = {
  on: function(_, _callback) {//TODO use EventEmitter
    this._onChangeCallback = _callback;
  },
  getAll: function(callback) {
    callback(todos);
  },
  complete: function(id) {
    for(var i = 0; i < todos.length; i++) {
      var todo = todos[i];
      if(todo.id === id) {
        var newTodo = React.addons.update(todo, {done: {$set: true}});
        todos = React.addons.update(todos, {$splice: [[i, 1, newTodo]]});
        this._onChangeCallback();
        break;
      }
    }
  },
  create: function(name, callback) {
    var newTodo = {
      id: generateId(),
      name: name
    };
    todos = React.addons.update(todos, {$push: [newTodo]});
    this._onChangeCallback();
    callback();
  }
};

var Todo = React.createClass({
  handleClick: function() {
    TodoStorage.complete(this.props.todo.id);
  },
  render: function() {
    var todo = this.props.todo;
    var doneButton = todo.done ? null : <button onClick={this.handleClick}>Done</button>;
    return (<li>{todo.name}{doneButton}</li>);
  }
});

var TodoForm = React.createClass({
  getInitialState: function() {
    return {
      name: ''
    };
  },
  handleNameChange: function(e) {
    this.setState({
      name: e.target.value
    });
  },
  handleSubmit: function(e) {
    e.preventDefault();
    var name = this.state.name.trim();
    TodoStorage.create(name, function() {
      this.setState({
        name: ''
      });
    }.bind(this));
  },
  render: function() {
    var disabled = this.state.name.trim().length <= 0;
    return (
      <form onSubmit={this.handleSubmit}>
        <input value={this.state.name} onChange={this.handleNameChange}></input>
        <input type="submit" disabled={disabled}></input>
      </form>
    );
  }
});

var TodoList = React.createClass({
  render: function() {
    var rows = this.props.todos.filter(function(todo) {
      return !todo.done;
    }).map(function(todo) {
      return (<Todo key={todo.id} todo={todo}></Todo>);
    });
    return (
      <div className="active-todos">
        <h2>Active</h2>
        <ul>{rows}</ul>
        <TodoForm/>
      </div>
    );
  }
});

var App = React.createClass({
  getInitialState: function() {
    return {
      todos: []
    };
  },
  componentDidMount: function() {
    var setState = function() {
      TodoStorage.getAll(function(todos) {
        this.setState({
          todos: todos
        });
      }.bind(this));
    }.bind(this);
    TodoStorage.on('change', setState);
    setState();
  },
  render: function() {
    return (
      <div>
        <h1>My Todo</h1>
        <TodoList todos={this.state.todos}/>
      </div>
    );
  }
});

React.render(
  <App></App>,
  document.getElementById('app-container')
);
```
  
```
\\ Views\reactJS01.html
<html>
  <head>
    <script src="http://fb.me/react-with-addons-0.12.2.js"></script>
    <script src="http://fb.me/JSXTransformer-0.12.2.js"></script>
  </head>
  <body>
    <h1>react.js 테스트</h1>

    <div id="app-container"></div>
    <script type="text/jsx" src="../Content/app.js"></script> <!-- html 이외의 파일은 꼭 Content 폴더에 있어야 한다 -->
  </body>
</html>
```
  
  
### React.JS
  
```
class TestForm
{
    // 꼭 속성으로 만들어야 한다!!!!
    public string User { get; set; }
    public string Pass { get; set; }
}

Require<NancyPack>().Host();

public class IndexModule : NancyModule
{
    public IndexModule()
    {
        Get["/"] = _ =>    "Hello world";
        Post["/json"] = _ => {
        // 클라이언트에서 보낸 데이터가 form에 바인딩 된다.      
        var form = this.Bind<TestForm>();
                return Response.AsJson(form);
      };        
    }
}
```
  