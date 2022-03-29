# UE大纲（草稿）
# 委托(Delegate)
## 类型
- 单播: TDelegate模板类，通常用DECLARE_DELEGATE以及DECLARE_DELEGATE_XXXParams宏声明，后面的宏表示带XXX个参数
- 多播: TMulticastDelegate模板类，通常用DECLARE_MULTICAST_DELEGATE以及DECLARE_MULTICAST_DELEGATE_XXXParams宏进行声明，后面的宏表示带有XXX个参数
- 动态单播: TBaseDynamicDelegate模板类子类,通常用DECLARE_DYNAMIC_DELEGATE宏进行声明，DECLARE_DYNAMIC_DELEGATE_XXXParam版本的声明表示带有XXX个参数
- 动态多播:TBaseDynamicMulticastDelegate(多播)模板类子类, 通常用和DECLARE_DYNAMIC_MULTICAST_DELEGATE宏进行声明，DECLARE_DYNAMIC_MULTICAST_DELEGATE_XXXParam版本的声明表示带有XXX个参数 
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
## RPC调用

RPC的声明有三种:
- 将某函数声明为在服务器上调用，但在客户端执行: UFUNCTION(Client)
- 将某函数声明为在客户端上调用，但在服务器执行: UFUNCTION(Server)
- 从服务器调用，在服务器和当前所有连接的n个客户端上执行(共n + 1): UFUNCTION(NetMulticast)

RPC默认不可靠，如果要在远端保证调用，则添加关键字Reliable  
UFUNCTION(Server, Reliable)

### 可靠RPC
### 非可靠RPC
## ReplicationGraph
# 多线程
## FRunable
## AsyncTask
## Async
## TaskGraph

# UObject
## 序列化
## 垃圾回收
## 反射

# 模块

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