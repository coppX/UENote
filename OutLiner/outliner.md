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
## Character
## Controller
## Camera
## GameMode
## GameState
## PlayerState
## Actor
## Component

# 网络同步
- 帧同步
- 状态同步
## 属性同步
### 网络相关性

## RPC调用
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