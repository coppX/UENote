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