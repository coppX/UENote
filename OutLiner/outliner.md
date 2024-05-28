# UE大纲（草稿）
# 委托(Delegate)
## 类型
- 单播: TDelegate模板类，通常用DECLARE_DELEGATE以及DECLARE_DELEGATE_XXXParams宏声明，后面的宏表示带XXX个参数
- 多播: TMulticastDelegate模板类，通常用DECLARE_MULTICAST_DELEGATE以及DECLARE_MULTICAST_DELEGATE_XXXParams宏进行声明，后面的宏表示带有XXX个参数
- 动态单播: TBaseDynamicDelegate模板类子类,通常用DECLARE_DYNAMIC_DELEGATE宏进行声明，DECLARE_DYNAMIC_DELEGATE_XXXParam版本的声明表示带有XXX个参数
- 动态多播:TBaseDynamicMulticastDelegate(多播)模板类子类, 通常用DECLARE_DYNAMIC_MULTICAST_DELEGATE宏进行声明，DECLARE_DYNAMIC_MULTICAST_DELEGATE_XXXParam版本的声明表示带有XXX个参数 
- 事件: TMulticastDelegate模板子类, 可以看成是特殊的多播，只有定义该事件的类才可以调用

## 绑定
- 单播: Bind...(BindStatic, BindRaw, BindLambda, BindSP, BindThreadSafeSP, BindUFunction, BindUObject), 通过Bind后面带的关键字，即可知道需要绑定的类型是静态函数，C++原生函数，Lambda, SharedPtr成员函数，线程安全的SharedPtr成员函数，基于UFunction的成员函数，UObject里面的函数。
- 多播: Add...(AddStatic, AddRaw, AddLambda, AddWeakLambda, AddSP, AddThreadSafeSP, AddUFunction, AddUObject)
- 动态单播: BindDynamic
- 动态多播: AddDynamic, AddUniqueDynamic
- 事件: 和多播一样
## 执行
- 单播: Execute, ExecuteIfBound
- 多播：BroadCast
- 动态单播: Execute, ExecuteIfBound
- 动态多播: Broadcast
- 事件: 和多播一样

# GamePlay
## Pawn
兵卒，设计给玩家操作的对象抽象，Pawn由Controller拥有，可以接受输入，pawn不被认定为具有人的特性
## Character
Character是带有人形风格的Pawn，Character继承于Pawn，默认情况下，带有一个拥有碰撞的CapsuleComponent和一个用于移动的CharacterMovementComponent组件
## Controller
控制Pawn的行为，继承自AActor，Controller有两种，一种是PlayerController，一种是AIController，PlayerController用来控制Pawn，需要玩家手动来控制。而AIController控制Pawn时，是计算机用来控制NPC的行为。
## GameMode
GameMode继承于AInfo，承担了游戏玩法的职责，包括游戏的开始结束，游戏内实体的Spawn(包括Pawn，PlayerController,AIController也是由GameMode负责Spawn)，关卡切换，只存在于服务器，然后Replicated到所有客户端
## GameState
GameState继承于AInfo，用来保存当前游戏的状态数据，其中可以包括联网玩家列表、得分、棋类游戏中棋子的位置，或者开放时间场景中完成任务的列表等。存在于服务端，然后也可以Replicated到多个客户端。
## PlayerState
PlayerState继承于AInfo，用来保存玩家的数据，比如玩家姓名、得分、MOBA等比赛中等级，PlayerState存在客户端，所有玩家的PlayerState存在于所有的机器上，并且可以Replicated以保存同步
## Actor
## Component

# 网络同步
- 帧同步(Lockstep/锁步同步): 游戏逻辑在客户端，不同的客户端之间通过发送操作指令，使得所有的客户端保持一致。比如玩家A向前移动了一步，然后这个移动操作就会同步给其他客户端，使得大家都看到A向前移动一步，并且lock step只有大家都完成了同步才会继续往下执行。
- 状态同步:游戏逻辑在服务端，服务器只同步影响游戏功能的某些重要状态变量，并且这些重要变量是在服务器运算出来的或者至少校验过的，客户端拿到这些状态变量后自行做本地的表现。
  
一般来说帧同步在实时性、节省流量方面比较好，状态同步则在安全性角度来说更胜一筹。游戏逻辑在服务器上能有效减少外挂，对于中途加入/断线重连也能天然支持。UE采用的是状态同步，并且分为属性同步和RPC
## 属性同步
由于游戏逻辑在服务端，如果服务端玩家过多，对于每个玩家需要同步所有的玩家数据量很大，这样就会对网络带宽造成很大的负担。UE采用以下几种方式来减少要同步的数据量
### 相关性计算
与本玩家有关的数据才同步，比如同步在视野里能看到的玩家，或者一定距离内的玩家数据。某个Actor和某个客户端连接是否相关的属性配置，我们可以简单归为三类
- 同步距离:参考ReplicationGraph
- 同步范围:(AlwaysRelevant，和所有的连接都相关，OnlyRelevantToOwner，仅和Owner相关)
- 同步频率:(NetUpdateFrequency,上次同步间隔小于改频率，在该帧不进行同步)
### 优先级计算
当网络不好时，会根据优先级，先同步优先级高的数据。如果带宽饱和了则当前帧就不会同步优先级低的数据，然后没有同步的数据优先级会提升，会在接下来的几帧后会被同步。
### 成员变量计算
需要同步的Actor里面有很多成员变量，但是并不是所有的变量都需要同步，只有被标记为Replicated的成员变量，并且当它的值发生变化的时候才会同步给相关的客户端。并且还有额外标记控制同步，比如bNetInit标记只在刚建立起同步通道时才会同步。

Actor、Pawn和Character的部分功能不会复制:
- Skeletal Mesh和Static Mesh组件
- 材质
- 动画蓝图
- 粒子系统
- 音效发射器
- 物理对象
## RPC调用

RPC的声明有三种:
- 将某函数声明为在服务器上调用，但在客户端执行: UFUNCTION(Client)
- 将某函数声明为在客户端上调用，但在服务器执行: UFUNCTION(Server)
- 从服务器调用，在服务器和当前所有连接的n个客户端上执行(共n + 1): UFUNCTION(NetMulticast)

### 从服务器调用的RPC
|Actor所有权         | 未复制         | NetMulticast          | Server       | Client
|:-----------------|:--------------|:----------------------|:-------------|:------------------|
|Client-owned actor | 在服务器上运行  | 在服务器和所有客户端上运行 | 在服务器上运行 | 在actor所属客户端上运行|
|Server-owned actor | 在服务器上运行  | 在服务器和所有客户端上运行 | 在服务器上运行 | 在服务器上运行 |
|Unowned actor      | 在服务器上运行  | 在服务器和所有客户端上运行 | 在服务器上运行 | 在服务器上运行 |

### 从客户端调用的RPC
|Actor所有权                  | 未复制                 | NetMulticast         | Server       | Client
|:---------------------------|:----------------------|:---------------------|:-------------|:--------------------|
|owned by invoking client    | 在执行调用的客户端上运行  | 在执行调用的客户端上运行 | 在服务器上运行 | 在执行调用的客户端上运行 |
|owned by a different client | 在执行调用的客户端上运行  | 在执行调用的客户端上运行 | 丢弃         | 在执行调用的客户端上运行 |
|server-owned actor          | 在执行调用的客户端上运行  | 在执行调用的客户端上运行 | 丢弃         | 在执行调用的客户端上运行 |
|Unowned actor               | 在执行调用的客户端上运行  | 在执行调用的客户端上运行 | 丢弃         | 在执行调用的客户端上运行 |
### RPC的调用限制
1. 它们必须从Actor上调用
2. Actor必须被复制
3. 如果RPC是从服务器调用并且在客户端上执行，则只有实际拥有这个Actor的客户端才会执行函数。
4. 如果RPC是从客户端调用并在服务器上执行，客户端就必须拥有调用RPC的Actor。
5. 多播RPC是个例外
- 如果它们是从服务器调用，服务器将在本地和所有已连接的客户端上执行它们。
- 如果它们是从客户端调用，则只能在本地而非服务器上执行。  


RPC函数参数除了UObject类型的指针以及const FString&的字符串外，其他类型的指针或者引用都不可以作为RPC的参数，RPC函数没有返回值。
### RPC分类
- 可靠RPC：UE会用UDP模仿TCP协议，来保证RPC严格按序到达远端，适用于对GamePlay很关键但不经常调用的函数，包括碰撞事件、武器发射的开始或结束，或生成Actor
- 非可靠RPC: UE只管发送RPC调用请求，并不保证一定到达, 但发送的速度和频率要高于可靠RPC, 用于对GamePlay而言不重要或者经常调用的函数，比如Actor移动每帧都可能变换，因此使用非可靠RPC复制该Actor移动。    
注: 滥用可靠RPC可能会导致其队列溢出，此操作将强制断开连接。若逐帧调用RPC，应将其设为不可靠。若拥有与玩家输入绑定的可靠RPC，应该限制玩家调用该函数的频率。  

TCP连接的优点是可靠稳定，但速度慢这个缺点导致他不适合网游。所以UE在UDP的基础上融合了TCP的特点，加入了乱序重连，以及对Reliable的丢包重传机制，既保证了可靠性，也保证了传输速度。  
RPC默认不可靠，如果要在远端保证调用，则需要添加关键字Reliable  
UFUNCTION(Server, Reliable)
### 可靠性
- 丢包重传：被标记为reliable的包会在发送端保存一个备份，只有收到接收端的ACK包确认后才会清理掉，若收到ACK包跳序则就会触发重传。
- 乱序整理：UE会在发送端为包添加序列号，在接收端会对收到的包进行排序，如果有不完整的包会等待重组。
### 网络角色和授权
Actor的网络角色将决定游戏运行期间控制Actor的机器
- ROLE_None:Actor在网络游戏中没有Role，不会复制
- ROLE_Authority:Actor为授权状态，会将信息复制给其他机器上的远程代理
- ROLE_SimulatedProxy:Actor为远程代理，由另外一台机器上的Actor完全控制。网络游戏中如拾取物，发射物或交互对象等多数Actor将在远程客户端上显示为模拟代理
- ROLE_AutonomousProxy:Actor为远程代理，能够本地执行部分功能，但会接收授权Actor中的矫正，自主代理通常为玩家直接控制的Actor所保留，如Pawn  
  
Actor存在Role和RemoteRole，Role是本地的，RemoteRole是对应端，比如Role是客户端ROLE_AutonomousProxy，RemoteRole就是ROLE_Authority。
### 连接/NetConnection
对于任意一个Actor(客户端),他可以有连接，也可以无连接。一旦Actor有连接，他的Role就是ROLE_AutonomousProxy，如果没有连接，他的Role就是ROLE_SimulatedProxy。对于一个actor，他有三种方式来获得这个连接(或者说让自己属于这个连接):
- 设置自己的Owner为拥有连接的PlayerController,或者自己Owner的Owner为拥有连接的PlayerController，也就是官方文档说的查找他最外层的Owner是否为PlayerController而且这个PlayerController必须拥有连接。
- 这个Actor必须是Pawn并且Possess了拥有连接的PlayerController。
- 这个Actor设置自己的Owner为拥有连接的Pawn。这个区别于第一点就是，Pawn与Controller的绑定方式不是通过Owner这个属性。而是Pawn本身就具有Controller这个属性(成员)，所以Pawn的Owner可能为空。

对于组件来说也一样，也是通过它归宿的那个actor，然后在去找和actor相关的PlayerController，用上面的方法。我们发现连接和我们操作的对象有关，像NPC就没有连接，他们的Role就是ROLE_SimulatedProxy。
### 连接的作用
- RPC需要确定哪个客户端将执行运行于客户端的RPC。当服务端调用UFUNCTION(Client)时，我怎么知道应该去哪个客户端执行
- Actor复制和连接相关性。Actor是可以同步，他的属性也是，有些东西我们就需要同步给连接的客户端
- 在涉及所有者时的Actor属性复制条件。比如以下例子
  ```cpp
    void AActor::GetLifetimeReplicatedProps(TArray<FLifetimeProperty>& OutLifetimeProps) const
    {
        DOREPLIFETIME_CONDITION(AActor, ReplicatedMovement, COND_AutonomousOnly);
    }
  ```
  这里的ReplicatedMovement属性就限制在了只拥有COND_AutonomousOnly的Actor上才能同步,这种条件一般只在Actor初始化的时候同步一次，接下来的游戏过程中不会同步，一般在同步玩家的姓名性别这种不会改变属性的场景下会用，如果游戏过程中改变了，就需要手动调用函数来同步，这样就能减少同步消耗。
## 属性同步 or RPC
对于到底选择属性同步还是选择RPC来做网络同步，会根据以下几点来做选择
- 属性同步只能是从服务端同步给客户端，不存在从客户端同步到服务端，如果需要从客户端同步数据到服务端，则采用RPC的方式，RPC是可以双向同步的。
- 属性同步的持久性比较好，RPC的存在只有一瞬间，调用完成后就消失了，玩家断线重连后采用变量同步的方式能正确同步到之前的数据，但是RPC的同步已经无法回放了，因为RPC调用完就没了。
- RPC的实时性比属性同步要好，RPC瞬间发送给远端执行，属性同步还需要等待packet满了之后才会发送网络请求，但是实时性相差不大。
# 网络连接
## UDP如何做到可靠
## 断线重连
## ReplicationGraph
# 多线程
## FRunnable
FRunnable是UE最基础的线程用法
```cpp
class CORE_API FRunnable
{
public:
    virtual bool Init();
    virtual uint32 Run() = 0;
    virtual void Stop();
    virtual void Exit();
    virtual class FSingleThreadRunnable* GetSingleThreadInterface();
	virtual ~FRunnable()
};
```
我们自定义一个继承于FRunnable类型的类，并且需要实现Run接口，这个Run接口就是我们需要线程去执行的函数，这个类型就相当于对线程执行函数进行了封装，真正的线程执行函数是这个Run。而执行这个线程函数的线程确实是FRunnableThread，具体用法如下:
```cpp
void ATestRunnableActor::BeginPlay()
{
    Super::BeginPlay();
    FTestRunnable* Runnable1 = new FTestRunnable(TEXT("线程1"), this);
    FTestRunnable* Runnable2 = new FTestRunnable(TEXT("线程2"), this);
    FRunnableThread* RunnableThread1 = FRunnableThread::Create(Runnable1, *Runnable1->MyThreadName);
    FRunnableThread* RunnableThread2 = FRunnableThread::Create(Runnable2, *Runnable2->MyThreadName);
}
```
FRunnable（线程执行体）和FRunnableThread（线程类）是最简单的实现多线程方式，它只有创建、暂停、销毁、等待完成等基础功能。在实战中也较少用到。
## AsyncTask
AsyncTask是利用的UE底层的线程库来执行的，可以分为FAsyncTask和FAutoDeleteAsyncTask
### FAsyncTask
FAsyncTask是一个模板类，真正的AsyncTask需要你自己写，FAsyncTask和AsyncTask的关系类似于上面的FRunnableThread和FRunnable，也是需要定义一个AsyncTask子类，并且实现DoWork，然后用子类去实例化FAsyncTask模板，FAsyncTask模板就会调用DoWork执行任务
### FAutoDeleteAsyncTask
FAutoDeleteAsyncTask和FAsyncTask几乎一样，不同的是FAutoDeleteAsyncTask执行完任务后会自动删除


例子:
```cpp
class FTestAsyncTask : public FNonAbandonableTask
{
    friend class FAutoDeleteAsyncTask<FTestAsyncTask>;

    void DoWork()
    {
        ...
    }
    ...
};

void ATestAsyncActor::ATestAsyncTaskClass()
{
    (new FAsyncTask<FTestAsyncTask>())->StartBackgoundTask();
    (new FAsyncTask<FTestAsyncTask>())->StartSynchronousTask();
    (new FAutoDeleteAsyncTask<FTestAsyncTask>())->StartBackgoundTask();
    (new FAutoDeleteAsyncTask<FTestAsyncTask>())->StartSynchronousTask();
}
```

这里的StartBackgroundTask和StartSynchronousTask的区别:
- StartBackgroundTask会利用线程池里空闲的线程来执行
- StartSynchronousTask则是主线程来执行，会阻塞主线程  


上面的Task继承于FNonAbandonableTask,当FAsyncTask销毁的时候，会调用Abandon函数，如果FAsyncTask里面的Task继承于FNonAbandonableTask的话，这个时候就不会丢弃而是等待执行完成才会完成FAsynTask的销毁。如果不需要丢弃任务则不能继承FNonAbandonableTask，需要自己实现CanAbondon和Abandon函数。
## TaskGraph
TaskGraph就比前面几种要复杂了，他可以创建多个线程任务，并且指定各个任务之间的依赖关系，按照该关系来处理任务。虽然各个任务直接有复杂的关系，但是和前面几种也类似，需要我们自己创建任务类型，每种类型里面实现DoTask函数来表示要执行的任务类型，比如FTickFunctionTask、FReturnGraphTask等任务，我们先创建一个任务
```cpp
class FMyTestTask
{
public:
    FMyTestTask() {}
    static const TCHAR* GetTaskName() { return TEXT("MyTestTask"); }
    FORCEINLINE static TStatId GetStatId()
    {
        return RETURN_QUICK_DECLARE_CYCLE_STAT(FMyTestTask, STATGROUP_TaskGraphTasks);
    }

    static ENamedThreads::Type GetDesiredThread()
    {
        return ENamedThreads::AnyThread;
    }
    /*
    namespace ESubsequentsMode
    {
        enum Type
        {
            TrackSubsequents,
            FireAndForget
        };
    }
    */
    static ESubsequentsMode::Type GetSubsequentsMode()
	{
		return ESubsequentsMode::TrackSubsequents;
	}
 
    // 任务的执行逻辑，其中参数：
    //  CurrentThread - 任务执行的线程类型信息
    //  MyCompletionGraphEvent - 该任务的后续任务，可以通过DontCompleteUntil让其挂起直到后续后续任务完成再继续
    void DoTask(ENamedThreads::Type CurrentThread, const FGraphEventRef& MyCompletionGraphEvent)
    {
        MyCompletionGraphEvent->DontCompleteUntil(TGraphTask<FSomeChildTask>::CreateTask(NULL,CurrentThread).ConstructAndDispatchWhenReady());
    }
};
```
说明:
- GetStatId: 固定写法，RETURN_QUICK_DECLARE_CYCLE_STAT第一个参数为类名
- GetDesiredThread: 可以指定使用哪种线程来执行这个任务。除了AnyThread还有GameThread, RHIThread, AudioThread等多种线程种类。
- GetSubsequentsMode::TrackSubsequents: 追踪完成状态，一般用这个，说明存在后续任务
- GetSubsequentsMode::FireAndForget: 不需要追踪任务完成状态，只有没有任何依赖的Task才用。
- DoWork: 任务执行体
- FGraphEventRef:是FGraphEvent的指针，用来传递任务完成状态的，比如我上一个任务完成了就把完成时间传给我这个任务，我这个任务就能开始执行了，我完成了任务，我就会把完成事件传递给下一个任务(如果有)。

线程在TaskGraph里面被称为FWorkerThread，通过全局单例类FTaskGraphImplementation来创建和分配任务的，默认情况下会开启5个基本线程StatThread、RHIThread、AudioThread、GameThread、ActualRenderingThread，还有其他无名的线程。FWorkerThread里面其实封装的FRunnableThread类型成员就是真正的线程，FWorkerThread还封装了FTaskThreadBase类型成员，它继承于FRannable线程执行体。FTaskThreadBase有两个子类FTaskThreadAnyThread和FNamedTaskThread，对应着有名字的(比如上面五个基本线程)和没有名字的线程执行体。  

### 单个线程中任务执行顺序
如果我们将多个任务放到一个线程里面，他们的执行顺序是怎样的。
- FTaskThreadAnyThread: 放到无名线程里面，会在创建任务的时候按照优先级放到IncomingAnyThread数组里面，然后每次线程执行完成后从这个数组里面弹出一个未执行的任务来执行，我们可以随时修改和调整这个任务队列。
- FNamedTaskThread: 放到有名字的线程里面，会被放到骑本身维护的队列里面，通过FThreadTaskQueue来处理执行顺序，一旦放到这个队列里面，我们就无法随意调整任务了。
  
### 任务依赖
一个任务依赖于多个事件，当多个事件全部触发之后才会执行这个任务。多个事件怎么来的呢，这些事件是前面的任务完成后才会触发的，每个任务完成都会触发一个事件，所以就能实现后面的任务依赖于前面的任务完成这个功能。  

设置依赖事件
```cpp
static FConstructor CreateTask(const FGraphEventArray* Prerequisites = NULL, ENamedThreads::Type CurrentThreadIfKnown = ENamedThreads::AnyThread)
{
    int32 NumPrereq = Prerequisites ? Prerequisites->Num() : 0;
    if (sizeof(TGraphTask) <= FBaseGraphTask::SMALL_TASK_SIZE)
    {
        void *Mem = FBaseGraphTask::GetSmallTaskAllocator().Allocate();
        return FConstructor(new (Mem) TGraphTask(TTask::GetSubsequentsMode() == ESubsequentsMode::FireAndForget ? NULL : FGraphEvent::CreateGraphEvent(), NumPrereq), Prerequisites, CurrentThreadIfKnown);
    }
    return FConstructor(new TGraphTask(TTask::GetSubsequentsMode() == ESubsequentsMode::FireAndForget ? NULL : FGraphEvent::CreateGraphEvent(), NumPrereq), Prerequisites, CurrentThreadIfKnown);
}

FGraphEventRef Join = TGraphTask<FVictoryTestTask>::CreateTask(NULL, ENamedThread::GameThread).ConstructAndDispatchWhenReady();
FTaskGraphInterface::Get().WaitUntilTaskCompletes(Join, ENamedThreads::GameThread_Local);
```
CreateTask第一个参数就是该任务的依赖事件数组(这里为NULL)，如果传入一个事件数组的话，那么当前任务就会通过SetupPrereqs函数设置这些依赖事件，并且在所有的依赖事件都触发后再将该任务放到任务队列里面分配给线程执行。  
当执行CreateTask时，会通过FGraphEvent::CreateGraphEvent()构建一个新的后续事件，然后通过ConstructAndDispatchWhenReady我们就能得到这个后续事件Join，我们在下面通过WaitUntilTaskCompletes等待该任务结束并且触发事件Join后继续执行。当前面这个事件完成后，就会调用DispatchSubsequents()去触发他后续的任务。
## 使用建议
对于消耗大的，复杂的任务不建议使用TaskGraph，一是因为TaskGraph如果被分配到游戏线程会阻塞整个游戏线程，二是因为不在有名字的线程上执行，也可能会影响到其他游戏逻辑，比如物理计算相关的任务就是在没有名字的线程上执行。复杂任务建议用FRunnable或者AsyncTask，因为AsyncTask都是使用新建的线程池里面的线程，一般与游戏tick无关，对于简单的任务，或者具有依赖关系的线程就放到TaskGraph里面使用。  
不要在非游戏线程里面进行以下操作:
- 不要Spawn/Modify/delete UObjects AActors
- 不要使用定时器TimerManager
- 不要使用绘制接口
# UObject
## 序列化
## 垃圾回收
## 反射

# 模块
# Subsystem

# UHT说明符
## UPROPERTY
- EditAnyWhere: 在编辑器中可见，且可编辑
- EditDefaultsOnly: 只在类默认设置中可见
- EditInstanceOnly：可通过属性窗口进行编辑，但只能在实例上进行，不能在原型上进行编辑
- VisibleAnywhere: 在编辑器中可见，但不能编辑
- BlueprintReadOnly: 蓝图只读
- BlueReadWrite: 蓝图可读写
- Replicated：应通过网络复制该属性
- ReplicatedUsing=FunctionName: 说明符指定一个回调函数，该函数在通过网络更新属性时执行。

## UFUNCTION
- BlueprintCallable：此函数可在蓝图或关卡蓝图中调用
- BlueprintPure：此函数不对拥有它的对象产生任何影响，可在蓝图或关卡蓝图图表中执行
- BlueprintimplementableEvent：需要在蓝图里面重载，不能在C++里面实现
- BlueprintNativeEvent：此函数旨在被蓝图覆盖掉，但是也具有默认原生实现，用于声明名称与主函数相同的附加函数，但是末尾添加了_Implementaion,是写入代码的位置，如果未找到任何蓝图覆盖，该自动生成的代码将调用Implementaion方法
- CallInEditor：通过细节(detail)面板中的按钮在编辑器中的选定实例上调用此函数
- Server: 该函数仅在服务器上执行。声明一个与主函数同名的附加函数，但末尾添加了_Implementation，。自动生成的代码将在必要时调用该方法。
- Client:  该函数仅在客户端上执行。声明一个与主函数同名的附加函数，但末尾添加了_Implementation，。自动生成的代码将在必要时调用该方法。
- Reliable:该功能通过网络复制，并且无论带宽或网络错误如何，都保证到达。仅当与Client或Server结合使用时才有效。
- Unreliable:该功能通过网络复制，但由于带宽限制或网络错误而失败。仅当与Client或Server结合使用时才有效。
- WithValidation:声明一个与主函数同名的附加函数，末尾添加_Validatebool。此函数采用相同的参数，并返回bool以指示是否应继续调用主函数。
- NetMulticast:该函数在服务器上本地执行，并复制到所有客户端，而不考虑Actor的NetOwner。 
