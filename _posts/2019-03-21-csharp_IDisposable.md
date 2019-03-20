---
layout: post
title: C# - IDisposable
published: true
categories: [.NET]
tags: c# .net IDisposable
---
## 개요
- Disposable.Create() 로 IDisposable을 모을 수 있다.
- 컨테이너(IDisposableContainer)를 만들고 IDisposable한 인터페이스를 .AddTo(container)로 등록한다.
  
```
public sealed class Resource : IDisposable {
    private readonly string name;
    public Resource( string name ) {
        this.name = name;
    }
    public void Dispose() => Console.WriteLine( "Disposed." + this.name );
}

static void Main( string[] args ) {
    // IDisposable한 Action 컨테이너를 만든다.
    IDisposableContainer container = Disposable.Create();

    // 임의의 리소스를 만들고, 컨테이너에 저장
    var resource = new Resource( "hoge" );
    var removeToken = resource.AddTo( container );

    removeToken.Dispose();  // resource.Dispose()를 실행하고, container에서 resource를 해제한다.

    // removeToken가 필요 없는 경우…
    Resource resource2 = new Resource().AddToChain( container );

    // container에 저장된 IDisposable한 Action울 모아서 해방한다.
    container.Dispose();
}
```
  
  
  
## 사용 방법
- IDisposable한 오브젝트는 확장 메소드에 의해서 AddTo, AddToChain 메소드를 사용할 수 있으므로 AddTo, AddToChain에 등록처의 IDisposableContainer를 넘긴다.
    - AddTo : 컨테이너에서 개별적으로 해방할 수 있는 IDisposable을 취득한다.
    - AddToChain : 컨테이너에 오브젝트를 등록하고 그대로 반환한다.
  
  
  
## DisposableToken
- 외부에서 리소스 등록만 허가하고 싶은 경우가 있다.
- DisposableToken을 공개하는 것으로 IDisposable.Dispose()을 외부에 비공개로 할 수 있다.
  
```
IDisposableContainer container = Disposable.Create();
DisposableToken token = container.Token;
```
  
- token을 AddTo, AddToChain 메소드에 넘길 수 있다.
  
  
  
## CancellationToken으로서 이용
- DisposableToken은 CancellationToken으로 암묵적으로 캐스트 할 수 있다. 그래서 Rx의 Subscribe하거나 Task의 캔슬토큰으로 이용할 수 있다.
  
```
//DisposableToken을 CancellationToken으로서 이용한다
IDisposableContainer container = Disposable.Create();
DisposableToken token = container.Token;
Observable.Interval( TimeSpan.FromSeconds( 1 ) )
    .Subscribe( Console.WriteLine , token );
```
  
container.Dispose()가 호출 될 때 취소된다.
  
  
  
## 파이널리자 이용
- .NET에는 불요하게된 오브젝트를 회수하는 GC 기능이 있다.
- 파이널리자가 정의된 오브젝트는 회수 되기 직전에 어떤 처리를 실행할 수 있다.
  
```
// 파이널리즈를 가능화 한다
IDisposableContainer container = Disposable.Create().ToFinalizable();
```
  
- 이와 같이 ToFinalizable()를 호출하면 파이널리자로서 container.Dispose()가 호출되도록 변환할 수 있다.
  
  
  
## 약 참조에 의한 관리
- 파이널리자를 정의한 오브젝트는 보통의 오브젝트에 비해서 회수 되기까지의 시간이 걸린다.
  
```
IDisposableContainer container = Disposable.Create().ToWeak();
```
  
- ToWeak()를 호출하면 파이널리자에 속박되지 않고 container의 참조 무효화를 검출하고, Dispose() 할 수 있도록 한다.
  
  
  
## 동기 컨텍스트 이용
- UI 스레드(STA)에서 조작하지 않으면 안되는 것이 있다.
- ToWeak로 하고 ToFinalizable로 하고 UI 스레드와는 서로 다른 스레드 상에서 동작하기 때문에 UI 스레드에 돌아와서 Dispose해야할 필요가 있다.
  
```
IDisposableContainer container =
        Disposable.Create().ToSynchronizable( SynchronizationContext.Current );
```
  
- ToSynchronizable로 동기 컨텍스트를 지정할 수 있다.
- SynchronizationContext에는 Send(동기)、Post(비동기)가 있지만 Post가 이용된다.
  
  
  
## 조합
- container의 참조가 무효화 되었다면 동기 컨텍스트에서 동기하여 Dispose 되도록 한다.
  
```
IDisposableContainer container = Disposable.Create()
   .ToSynchronizable( SynchronizationContext.Current )
   .ToWeak();
```
  
  
  
## 코드
- 스레드 세이프
- 복 수회 Dispose를 호출해도 문제 없다.
- Dispose 후에 리소스가 등록되면 부활한다.(다시 파이널리즈 가능이 된다)
- C# 6.0, VS2015 필요
  
```
using System;
using System.Collections.Concurrent;
using System.Threading;
using System.Threading.Tasks;
namespace PITA {
    /// <summary>
    /// 복수의 리소스 및 델리게이트를 집약 관리하는 컨테이너를 뜻한다
    /// </summary>
    public interface IDisposableContainer : IDisposable {
        /// <summary>
        /// 오브젝트를 컨테이너에 등록한다.
        /// </summary>
        /// <typeparam name="T">등록될 오브제그트의 타입</typeparam>
        /// <param name="resource">컨테이너에 등록되는 오브젝트</param>
        /// <returns>컨테이너에 등록된 오브젝트</returns>
        T Register<T>( T resource ) where T : IDisposable;

        /// <summary>
        /// <see cref="Action"/>을 리소스로서 컨테이너에 등록한다
        /// </summary>
        /// <param name="action">컨테이너에 등록하는 델리게이트</param>
        void Register( Action action );

        /// <summary>
        /// (해제 가능) 오브젝트를 컨테이너에 등록한다.
        /// </summary>
        /// <typeparam name="T">등록 되는 오브젝트의 타입</typeparam>
        /// <param name="resource">컨테이너에 등록 되는 오브젝트</param>
        /// <returns>コンテナから<paramref name="resource"/>을 해방하고,<paramref name="resource"/>の<see cref="IDisposable.Dispose"/>의 실행에 이용할 수 있다<see cref="IDisposable"/></returns>
        RemoveableDisposableToken RemoveableRegister<T>( T resource ) where T : IDisposable;

        /// <summary>
        /// (해제 가능) <see cref="Action"/>을 컨테이너에 등록한다
        /// </summary>
        /// <param name="action">컨테이너에 등록 되는 델리게이트</param>
        /// <returns>컨테이너에서<paramref name="action"/>을 해방하고, <paramref name="action"/>의 실행에 이용 할 수 있다<see cref="IDisposable"/></returns>
        RemoveableDisposableToken RemoveableRegister( Action action );

        /// <summary>
        /// 컨테이너에 관련된 토큰을 취득한다
        /// 이 토큰은 <see cref="CancellationToken"/>로 행동 할 수 있다
        /// </summary>
        DisposableToken Token {
            get;
        }
    }

    /// <summary>
    /// 범용 가능한<see cref="IDisposable"/>을 구축하기 위한 기저 클래스를 뜻한다
    /// </summary>
    public abstract class AnonymousDisposableBase : IDisposableContainer {

        /// <summary>
        /// 何もしない<see cref="IDisposable"/>を取得します。
        /// </summary>
        public static IDisposable Empty => Disposable.Empty;

        private int disposed = 0;
        private Action disposes = null;
        private CancellationTokenSource _cancelSource = null;

        /// <summary>
        /// リソース追加します。
        /// </summary>
        public T Register<T>( T resource ) where T : IDisposable {
            this.Register( resource.Dispose );
            return resource;
        }

        /// <summary>
        /// デリゲートをリソースとして登録します。
        /// </summary>
        /// <param name="action">リソースとして登録されるデリゲート</param>
        public void Register( Action action ) {
            if( Interlocked.CompareExchange( ref this.disposed , 0 , 1 ) == 1 )
                GC.ReRegisterForFinalize( this );
            this.disposes += action;
        }

        /// <summary>
        /// (解除可能) リソースを登録します。
        /// </summary>
        /// <typeparam name="T">登録されるオブジェクトの型</typeparam>
        /// <param name="resource">リソースとして登録されるオブジェクト</param>
        /// <returns>リソースの解除およびリソース<see cref="IDisposable.Dispose"/>の実行に使用できるコンテナ</returns>
        public RemoveableDisposableToken RemoveableRegister<T>( T resource ) where T : IDisposable => this.RemoveableRegister( resource.Dispose );

        /// <summary>
        /// (解除可能) <see cref="Action"/>を登録します。
        /// </summary>
        /// <param name="action">リソースとして登録される<see cref="Action"/></param>
        /// <returns>解除および登録された<see cref="Action"/>の実行に使用できる<see cref="IDisposable"/></returns>
        public RemoveableDisposableToken RemoveableRegister( Action action ) {
            this.Register( action );
            return new RemoveableDisposableToken( action , this.removeAction );
        }

        private void removeAction( Action action ) => this.disposes -= action;

        /// <summary>
        /// このオブジェクトに関連する<see cref="CancellationToken"/>を取得します。
        /// </summary>
        public CancellationToken CancelToken {
            get {
                if( Interlocked.CompareExchange( ref this.disposed , 0 , 1 ) == 1 )
                    GC.ReRegisterForFinalize( this );
                return LazyInitializer.EnsureInitialized( ref this._cancelSource ).Token;
            }
        }

        /// <summary>
        /// 登録操作のみに限定したトークンを取得します。
        /// </summary>
        public DisposableToken Token => new DisposableToken( this );

        /// <summary>
        /// 登録されている<see cref="IDisposable.Dispose"/>または、<see cref="Action"/>を呼び出し、それらのリソースを解放します。
        /// </summary>
        protected void OnDispose() {
            Interlocked.CompareExchange( ref this._cancelSource , null , this._cancelSource )?.Cancel();
            Interlocked.CompareExchange( ref this.disposes , null , this.disposes )?.Invoke();

            if( Interlocked.CompareExchange( ref this.disposed , 1 , 0 ) == 0 )
                GC.SuppressFinalize( this );
        }

        /// <summary>
        /// 登録されている<see cref="IDisposable.Dispose"/>または、<see cref="Action"/>を呼び出し、それらのリソースを解放します。
        /// </summary>
        public virtual void Dispose() => this.OnDispose();

        /// <summary>
        /// このオブジェクトのキャンセルトークンを取得します。
        /// </summary>
        /// <param name="source"></param>
        public static implicit operator CancellationToken( AnonymousDisposableBase source ) => source.CancelToken;
    }

    /// <summary>
    /// 汎用的な使用を目的とする<see cref="IDisposable"/>を提供します。
    /// </summary>
    public sealed class AnonymousDisposable : AnonymousDisposableBase {
        /// <summary>
        /// <see cref="AnonymousDisposable"/>クラスの新しいインスタンスを初期化します。
        /// </summary>
        internal AnonymousDisposable() {
            return;
        }

        /// <summary>
        /// <see cref="AnonymousDisposable"/>クラスの新しいインスタンスを初期化します。
        /// </summary>
        /// <param name="action">このオブジェクトの<see cref="IDisposable.Dispose"/>が実行されたとき呼び出す必要のある<see cref="Action"/></param>
        internal AnonymousDisposable( Action action ) {
            this.Register( action );
        }
    }

    /// <summary>
    /// <see cref="SynchronizationContext"/>を使用して、<see cref="IDisposable.Dispose"/>を呼び出します。
    /// </summary>
    public sealed class ContextDisposable : AnonymousDisposableBase {
        private readonly SynchronizationContext context;
        /// <summary>
        /// 現在の<see cref="SynchronizationContext"/>を指定して、<see cref="ContextDisposable"/>クラスの新しいインスタンスを初期化します。
        /// </summary>
        /// <param name="resource"><see cref="SynchronizationContext"/>に同期して<see cref="IDisposable.Dispose"/>を呼び出す必要のあるオブジェクト</param>
        internal ContextDisposable( IDisposable resource ) : this( SynchronizationContext.Current , resource ) {
            return;
        }

        /// <summary>
        /// <see cref="SynchronizationContext"/>を指定して、<see cref="ContextDisposable"/>クラスの新しいインスタンスを初期化します。
        /// </summary>
        /// <param name="context">同期コンテキスト</param>
        /// <param name="resource"><see cref="SynchronizationContext"/>に同期して<see cref="IDisposable.Dispose"/>を呼び出す必要のあるオブジェクト</param>
        internal ContextDisposable( SynchronizationContext context , IDisposable resource ) {
            this.context = context;
            this.Register( resource );
        }

        /// <summary>
        /// コンストラクタで指定された<see cref="SynchronizationContext"/>に同期して、<see cref="IDisposable.Dispose"/>の実行を試みます。
        /// </summary>
        public override void Dispose() {
            if( this.context != null ) {
                if( this.context == SynchronizationContext.Current ) {
                    this.OnDispose();
                } else {
                    this.context.OperationStarted();
                    this.context.Post( _ => {
                        try {
                            this.OnDispose();
                        } finally {
                            this.context.OperationCompleted();
                        }
                    } , null );
                }
            } else {
                base.Dispose();
            }
        }
    }

    /// <summary>
    /// <see cref="IDisposable.Dispose"/>をファイナライザで実行できるようにカプセル化します
    /// </summary>
    public sealed class AnonymousFinalizable : AnonymousDisposableBase {
        /// <summary>
        /// <see cref="AnonymousFinalizable"/>クラスの新しいインスタンスを初期化します。
        /// </summary>
        /// <param name="resource">このオブジェクトがファイナライズされると、関連して<see cref="IDisposable.Dispose"/>を実行するオブジェクト</param>
        internal AnonymousFinalizable( IDisposable resource ) {
            if( resource == null )
                throw new ArgumentNullException( nameof(resource) );
            this.Register( resource );
        }

        /// <summary>
        /// ファイナライザ
        /// </summary>
        ~AnonymousFinalizable() {
            this.Dispose();
        }
    }

    /// <summary>
    /// リソースの登録操作に限定します。
    /// また、<see cref="CancellationToken"/>としての振る舞いを提供します。
    /// </summary>
    public struct DisposableToken {
        private readonly IDisposableContainer source;

        /// <summary>
        /// トークンに関連付けられる<see cref="IDisposableContainer"/>を指定して<see cref="DisposableToken"/>を初期化します。
        /// </summary>
        /// <param name="resource">ホストされる<see cref="AnonymousDisposableBase"/></param>
        public DisposableToken( IDisposableContainer resource ) {
            this.source = resource;
        }

        /// <summary>
        /// リソースを登録します。
        /// </summary>
        /// <typeparam name="T">登録されるリソースの型</typeparam>
        /// <param name="resource">登録されるリソース</param>
        /// <returns>登録されたリソース</returns>
        public T Register<T>( T resource ) where T : IDisposable => this.source.Register( resource );

        /// <summary>
        /// <see cref="Action"/>をリソースとして登録します。
        /// </summary>
        /// <param name="action">リソースとして登録される<see cref="Action"/></param>
        public void Register( Action action ) => this.source.Register( action );

        /// <summary>
        /// (解除可能) <see cref="Action"/>を登録します。
        /// </summary>
        /// <param name="action">リソースとして登録される<see cref="Action"/></param>
        /// <returns>解除および登録された<see cref="Action"/>の実行に使用できる<see cref="IDisposable"/></returns>
        public RemoveableDisposableToken RemoveableRegister( Action action ) => this.source.RemoveableRegister( action );

        /// <summary>
        /// (解除可能) リソースを登録します。
        /// </summary>
        /// <typeparam name="T">登録されるオブジェクトの型</typeparam>
        /// <param name="resource">リソースとして登録されるオブジェクト</param>
        /// <returns>リソースの解除およびリソース<see cref="IDisposable.Dispose"/>の実行に使用できるコンテナ</returns>
        public RemoveableDisposableToken RemoveableRegister<T>( T resource ) where T : IDisposable => this.source.RemoveableRegister( resource );

        /// <summary>
        /// キャンセルトークンを取得します。
        /// </summary>
        public static implicit operator CancellationToken( DisposableToken token ) => ( (AnonymousDisposableBase)token.source ).CancelToken;
    }

    /// <summary>
    /// 弱い参照として<see cref="IDisposable"/>を管理します。
    /// トークンの参照が維持されなくなると、自動的に<see cref="IDisposable.Dispose"/>)を実行し、リソースを解放できます。
    /// </summary>
    internal static class WeakDisposableManager {
        private static long containerSeed = 0;
        private static int cleanuping = 0;
        private static int autoCleanupRunning = 0;
        private readonly static ConcurrentDictionary<long , WeakDisposable> weakObjects = new ConcurrentDictionary<long , WeakDisposable>();

        /// <summary>
        /// 定期的に参照の無効なトークンを検出し、それらのリソースを解放します。
        /// </summary>
        private async static void autoCleanupAsync() {
            if( Interlocked.CompareExchange( ref autoCleanupRunning , 1 , 0 ) == 0 ) {
                var oldCount = gcCollectCount();
                try {
                    while( weakObjects.Count > 0 && !Environment.HasShutdownStarted ) {
                        await Task.Delay( TimeSpan.FromMinutes( 1 ) ).ConfigureAwait( false );
                        var gc = gcCollectCount();
                        if( gc != oldCount ) {
                            oldCount = gc;
                            Cleanup();
                        }
                    }
                } finally {
                    Interlocked.Exchange( ref autoCleanupRunning , 0 );
                }
            }
        }

        /// <summary>
        /// ジェネレーション 1 以上で発生したガベージコレクトの発生回数を取得します。
        /// </summary>
        /// <returns>ジェネレーション 1 以上で発生したガベージコレクトの発生回数</returns>
        private static int gcCollectCount() {
            int count = 0;
            for( int i = 1; i <= GC.MaxGeneration; i++ )
                count += GC.CollectionCount( i );
            return count;
        }

        /// <summary>
        /// 参照が無効なトークンを検出し、それらのリソースを解放します。
        /// </summary>
        public static void Cleanup() {
            if( weakObjects.Count > 0 && Interlocked.CompareExchange( ref cleanuping , 1 , 0 ) == 0 ) {
                try {
                    foreach( var x in weakObjects ) {
                        if( x.Value.HasInstance )
                            continue;
                        x.Value.Dispose();
                    }
                } finally {
                    Interlocked.Exchange( ref cleanuping , 0 );
                }
            }
        }

        /// <summary>
        /// トークンの参照が解除されたときに実行される<see cref="Action"/>を指定して、トークンを生成します。
        /// </summary>
        /// <param name="action">トークンの参照が解除されたときに実行される<see cref="Action"/></param>
        /// <returns>このトークンオブジェクトの参照が保持されなくなると、<paramref name="action"/>を実行します。</returns>
        public static IDisposableContainer Create( Action action ) {
            if( action == null )
                throw new ArgumentNullException( nameof(action) );
            return new WeakDisposable( action ).WeakToken;
        }

        /// <summary>
        /// トークンの参照が解除されたときに解放される<see cref="IDisposable"/>を指定して、トークンを生成します。
        /// </summary>
        /// <param name="resource">トークンの参照が解除されたときに解放される<see cref="IDisposable"/></param>
        /// <returns>このトークンオブジェクトの参照が保持されなくなると、<paramref name="resource"/>を解放します。</returns>
        public static IDisposableContainer Create( IDisposable resource ) {
            if( resource == null )
                throw new ArgumentNullException( nameof(resource) );
            return Create( resource.Dispose );
        }

        /// <summary>
        /// 参照の維持に使用するトークンを表します。
        /// </summary>
        public sealed class WeakToken : IDisposableContainer {
            private readonly WeakDisposable host;

            internal WeakToken( WeakDisposable host ) {
                this.host = host;
            }

            public DisposableToken Token => this.host.Token;
            public void Register( Action action ) => this.host.Register( action );
            public T Register<T>( T resource ) where T : IDisposable => this.host.Register( resource );
            public RemoveableDisposableToken RemoveableRegister( Action action ) => this.host.RemoveableRegister( action );
            public RemoveableDisposableToken RemoveableRegister<T>( T resource ) where T : IDisposable => this.host.RemoveableRegister( resource );
            public void Dispose() => this.host.Dispose();
        }

        internal sealed class WeakDisposable : AnonymousDisposableBase {
            private readonly WeakReference<WeakToken> weakToken;
            private readonly long seed;

            public WeakDisposable() {
                this.seed = Interlocked.Decrement( ref containerSeed );
                weakObjects.TryAdd( seed , this );
                this.weakToken = new WeakReference<WeakToken>( new WeakToken( this ) );
                autoCleanupAsync();
            }

            /// <summary>
            /// <see cref="WeakDisposable"/>クラスの新しいインスタンスを初期化します。
            /// </summary>
            /// <param name="action">トークンの参照が無効になると、実行される<see cref="Action"/></param>
            public WeakDisposable( Action action ) : this() {
                if( action == null )
                    throw new ArgumentNullException( nameof(action) );
                this.Register( action );
            }

            /// <summary>
            /// 参照の保持に使用するトークンオブジェクトを取得します。
            /// </summary>
            public WeakToken WeakToken {
                get {
                    WeakToken tmp;
                    if( this.weakToken.TryGetTarget( out tmp ) )
                        return tmp;
                    throw new InvalidOperationException( nameof(WeakToken) );
                }
            }

            /// <summary>
            /// トークンの参照が保持されているかどうか取得します。
            /// </summary>
            public bool HasInstance {
                get {
                    WeakToken tmp;
                    return this.weakToken.TryGetTarget( out tmp ) && tmp != null;
                }
            }

            /// <summary>
            /// コンストラクタによって関連付けられた<see cref="Action"/>を実行し、それらのリソースを解放します。
            /// </summary>
            public override void Dispose() {
                this.OnDispose();
                WeakDisposable tmp = null;
                weakObjects.TryRemove( this.seed , out tmp );
            }
        }
    }

    /// <summary>
    /// <see cref="IDisposableContainer"/>の作成および、<see cref="IDisposable"/>の拡張メソッドを提供します。
    /// </summary>
    public static class Disposable {
        /// <summary>
        /// ファイナライザが<paramref name="source"/>の<see cref="IDisposable.Dispose"/>を実行できるように変換します。
        /// </summary>
        /// <param name="source">ファイナライザで<see cref="IDisposable.Dispose"/>を実行するオブジェクト</param>
        public static IDisposableContainer ToFinalizable<T>( this T source ) where T : class, IDisposable =>
            source is AnonymousFinalizable ? source as AnonymousFinalizable : new AnonymousFinalizable( source );

        /// <summary>
        /// 現在の<see cref="SynchronizationContext"/>に同期して<paramref name="source"/>の<see cref="IDisposable.Dispose"/>を実行できるように変換します。
        /// </summary>
        /// <param name="source">現在の<see cref="SynchronizationContext"/>に同期して<see cref="IDisposable.Dispose"/>を実行するオブジェクト</param>
        public static IDisposableContainer ToSynchronizable( this IDisposable source )
            => source is ContextDisposable ? (ContextDisposable)source : new ContextDisposable( source );

        /// <summary>
        /// <paramref name="context"/>によって指定された<see cref="SynchronizationContext"/>に同期して、<paramref name="source"/>の
        /// <see cref="IDisposable.Dispose"/>を実行できるように変換します。
        /// </summary>
        /// <param name="source"><paramref name="context"/>によって指定された<see cref="SynchronizationContext"/>に同期して<see cref="IDisposable.Dispose"/>を実行するオブジェクト</param>
        /// <param name="context">同期コンテキスト</param>
        public static IDisposableContainer ToSynchronizable( this IDisposable source , SynchronizationContext context )
            => source is ContextDisposable ? (ContextDisposable)source : new ContextDisposable( context , source );

        /// <summary>
        /// 参照が維持されなくなると、<paramref name="source"/>の<see cref="IDisposable.Dispose"/>を実行します。
        /// </summary>
        /// <param name="source">参照が維持されていない場合に解放する必要のある<see cref="IDisposable"/></param>
        /// <returns>参照の維持に使用するトークン</returns>
        public static IDisposableContainer ToWeak( this IDisposable source )
            => source is WeakDisposableManager.WeakToken ? (IDisposableContainer)source : WeakDisposableManager.Create( source );

        /// <summary>
        /// 指定の<see cref="Action"/>を連結した<see cref="IDisposable"/>を作成します。
        /// </summary>
        /// <param name="source">1番目のリソース</param>
        /// <param name="action">リソースとして振る舞う<see cref="Action"/></param>
        /// <returns><paramref name="source"/>と<paramref name="action"/>を連結した<see cref="IDisposable"/></returns>
        public static IDisposableContainer Concat( this IDisposable source , Action action ) {
            if( source == null )
                throw new ArgumentNullException( nameof(source) );
            if( action == null )
                throw new ArgumentNullException( nameof(action) );

            var container = source as IDisposableContainer;
            if( container != null ) {
                container.Register( action );
                return container;
            } else {
                return new AnonymousDisposable( source.Dispose + action );
            }
        }

        /// <summary>
        /// 2つの<see cref="IDisposable"/>を連結した<see cref="IDisposable"/>を作成します。
        /// </summary>
        /// <param name="source">1番目のオブジェクト</param>
        /// <param name="resource">2番目のオブジェクト</param>
        /// <returns>新しい<see cref="IDisposable"/></returns>
        public static IDisposableContainer Concat( this IDisposable source , IDisposable resource ) => Concat( source , resource.Dispose );

        /// <summary>
        /// (解除可能) <paramref name="resource"/>を<paramref name="container"/>に登録します。
        /// </summary>
        /// <typeparam name="T">オブジェクトの型</typeparam>
        /// <param name="resource"><paramref name="container"/>に登録されるリソース</param>
        /// <param name="container"><paramref name="resource"/>が登録される対象</param>
        /// <returns>リソース(<paramref name="resource"/>)の登録解除および解放に使用できるトークン</returns>
        public static RemoveableDisposableToken AddTo<T>( this T resource , IDisposableContainer container ) where T : IDisposable {
            if( resource == null )
                throw new ArgumentNullException( nameof(resource) );
            if( container == null )
                throw new ArgumentNullException( nameof(container) );
            return container.RemoveableRegister( resource );
        }

        /// <summary>
        /// (解除可能) <paramref name="token"/>がキャンセルされると、<paramref name="resource"/>の<see cref="IDisposable.Dispose"/>を実行します。
        /// </summary>
        /// <param name="resource"><paramref name="token"/>がキャンセルされると、<see cref="IDisposable.Dispose"/>を実行する必要のあるオブジェクト</param>
        /// <param name="token">トークンがキャンセルされると、<paramref name="resource"/>を解放します。</param>
        /// <returns>リソース(<paramref name="resource"/>)の登録解除および解放に使用できるトークン</returns>
        public static RemoveableDisposableToken AddTo<T>( this T resource , CancellationToken token ) where T : IDisposable {
            if( resource == null )
                throw new ArgumentNullException( nameof(resource) );
            return new RemoveableDisposableToken( token.Register( resource.Dispose ).Dispose + (Action)resource.Dispose );
        }

        /// <summary>
        /// (解除可能) <see cref="DisposableToken"/>を指定して、リソースを登録します。
        /// </summary>
        /// <param name="resource"><paramref name="token"/>に登録されるオブジェクト</param>
        /// <param name="token"><paramref name="resource"/>の登録対象</param>
        /// <returns>リソース(<paramref name="resource"/>)の登録解除および解放に使用できるトークン</returns>
        public static RemoveableDisposableToken AddTo<T>( this T resource , DisposableToken token ) where T : IDisposable {
            if( resource == null )
                throw new ArgumentNullException( nameof(resource) );
            return token.RemoveableRegister( resource );
        }

        /// <summary>
        /// <paramref name="resource"/>を<paramref name="container"/>に登録します。
        /// </summary>
        /// <typeparam name="T">オブジェクトの型</typeparam>
        /// <param name="resource"><paramref name="container"/>に登録されるリソース</param>
        /// <param name="container"><paramref name="resource"/>が登録される対象</param>
        /// <returns>コンテナに登録されたオブジェクト。<paramref name="resource"/></returns>
        public static T AddToChain<T>( this T resource , IDisposableContainer container ) where T : IDisposable {
            if( resource == null )
                throw new ArgumentNullException( nameof(resource) );
            if( container == null )
                throw new ArgumentNullException( nameof(container) );
            return container.Register( resource );
        }

        /// <summary>
        /// <paramref name="resource"/>を<see cref="CancellationToken"/>に登録します。
        /// </summary>
        /// <param name="resource"><paramref name="token"/>がキャンセルされると、<see cref="IDisposable.Dispose"/>を実行する必要のあるオブジェクト</param>
        /// <param name="token"><paramref name="resource"/>の<see cref="IDisposable.Dispose"/>の実行がトリガーされるトークン</param>
        /// <returns>トークンに登録されたオブジェクト。<paramref name="resource"/></returns>
        public static T AddToChain<T>( this T resource , CancellationToken token ) where T : IDisposable {
            if( resource == null )
                throw new ArgumentNullException( nameof(resource) );
            token.Register( resource.Dispose );
            return resource;
        }

        /// <summary>
        /// トークンにリソースを登録します。
        /// </summary>
        /// <param name="resource"><paramref name="token"/>に登録されるオブジェクト</param>
        /// <param name="token"><paramref name="resource"/>の登録対象</param>
        /// <returns>トークンに登録されたオブジェクト。<paramref name="resource"/></returns>
        public static T AddToChain<T>( this T resource , DisposableToken token ) where T : IDisposable {
            if( resource == null )
                throw new ArgumentNullException( nameof(resource) );
            return token.Register( resource );
        }

        /// <summary>
        /// リソースを集約管理できるコンテナを生成します。
        /// </summary>
        /// <returns>リソースを集約管理できるコンテナ</returns>
        public static IDisposableContainer Create() => new AnonymousDisposable();

        /// <summary>
        /// リソースを集約管理できるコンテナを生成します。
        /// </summary>
        /// <param name="action">コンテナに登録するデリゲート</param>
        /// <returns>リソースを集約管理できるコンテナ</returns>
        public static IDisposableContainer Create( Action action ) => new AnonymousDisposable( action );

        /// <summary>
        /// リソースを集約管理できるコンテナを生成します。
        /// </summary>
        /// <param name="resource">コンテナに登録するリソース</param>
        /// <returns>リソースを集約管理できるコンテナ</returns>
        public static IDisposableContainer Create<T>( T resource ) where T : IDisposable => new AnonymousDisposable( resource.Dispose );

        /// <summary>
        /// 何もしない<see cref="IDisposable"/>を取得します。
        /// </summary>
        public static IDisposable Empty {
            get;
        }
        = new EmptyDisposable();

        /// <summary>
        /// 参照が無効化されたトークンをクリーンアップします。
        /// </summary>
        public static void WeakDisposableCleanup() => WeakDisposableManager.Cleanup();

        /// <summary>
        /// 何もしない<see cref="IDisposable"/>を表します。
        /// </summary>
        private sealed class EmptyDisposable : IDisposable {
            public void Dispose() {
                return;
            }
        }

    }

    /// <summary>
    /// <see cref="IDisposableContainer"/>に登録されたリソースを回収し、そのリソースの解放に使用されるトークンを表します。
    /// </summary>
    public struct RemoveableDisposableToken : IDisposable {
        private readonly Action<Action> remove;
        private Action dispose;

        /// <summary>
        /// <see cref="IDisposable.Dispose"/>に関連付ける<see cref="Action"/>を指定して、<see cref="RemoveableDisposableToken"/>を初期化します。
        /// </summary>
        /// <param name="action"><see cref="IDisposable.Dispose"/>に関連付ける<see cref="Action"/></param>
        /// <param name="remove"><paramref name="action"/>を取り消すために使用するデリゲート</param>
        internal RemoveableDisposableToken( Action action , Action<Action> remove ) {
            if( action == null )
                throw new ArgumentNullException( nameof(action) );
            this.dispose = action;
            this.remove = remove;
        }

        /// <summary>
        /// <see cref="IDisposable.Dispose"/>に関連付ける<see cref="Action"/>を指定して、<see cref="RemoveableDisposableToken"/>を初期化します。
        /// </summary>
        /// <param name="action"><see cref="IDisposable.Dispose"/>に関連付ける<see cref="Action"/></param>
        internal RemoveableDisposableToken( Action action ) {
            if( action == null )
                throw new ArgumentNullException( nameof(action) );
            this.dispose = action;
            this.remove = null;
        }

        /// <summary>
        /// コンストラクタで指定された<see cref="Action"/>を実行します。
        /// </summary>
        public void Dispose() {
            if( this.remove != null )
                this.remove( this.dispose );
            Interlocked.CompareExchange( ref this.dispose , null , this.dispose )?.Invoke();
        }

        public override int GetHashCode() => ( this.dispose?.GetHashCode() ).GetValueOrDefault( 0 );
        public override bool Equals( object obj ) => obj is RemoveableDisposableToken && this.dispose == ( (RemoveableDisposableToken)obj ).dispose;
        public static bool operator ==( RemoveableDisposableToken x , RemoveableDisposableToken y ) => x.GetHashCode() == y.GetHashCode();
        public static bool operator !=( RemoveableDisposableToken x , RemoveableDisposableToken y ) => x.GetHashCode() != y.GetHashCode();
    }

}
```
  
  
  
## 참고
- http://qiita.com/Temarin_PITA/items/43396f85890c5acb5b30
