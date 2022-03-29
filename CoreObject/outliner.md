# 委托(Delegate)
## 类型
- 单播: TDelegate模板类，通常用DECLARE_DELEGATE_XXXParams宏声明，后面的宏表示带XXX个参数
- 多播: TMulticastDelegate模板类，通常用DECLARE_MULTICAST_DELEGATE以及DECLARE_MULTICAST_DELEGATE_XXXParams宏进行声明，后面的宏表示带有XXX个参数
- 动态: TBaseDynamicDelegate(单播)、TBaseDynamicMulticastDelegate(多播)模板类子类,通常用DECLARE_DYNAMIC_DELEGATE(单播)和DECLARE_DYNAMIC_MULTICAST_DELEGATE(多播)宏进行声明，DECLARE_DYNAMIC_DELEGATE_XXXParam版本的声明表示带有XXX个参数
- 事件: TMulticastDelegate模板子类, 可以看成是特殊的多播，只有定义该事件的类才可以调用

## 绑定
- 单播: Bind...(BindStatic, BindRaw, BindLambda, BindSP, BindThreadSafeSP, BindUFunction, BindUObject), 通过Bind后面带的关键字，即可知道需要绑定的类型是静态，C++原生，UObject里面的函数。
- 多播: Add...(AddStatic, AddRaw, AddLambda, AddWeakLambda, AddSP, AddThreadSafeSP, AddUFunction, AddUObject)
- 动态单播: BindDynamic
- 动态多播: AddDynamic, AddUniqueDynamic

## 执行
- 单播: Execute, ExecuteIfBound
- 多播：BroadCast
- 动态单播: Execute, ExecuteIfBound
- 动态多播: Broadcast