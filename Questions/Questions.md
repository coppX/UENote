https://www.jianshu.com/p/6461afc2cfc7

# 1. Actor的EndPlay事件在哪些时候会调用？
 - 对Destory显示调用
 - actor的生命周期已过
 - 关卡过渡(无缝加载和加载地图)包括Actor的关卡被卸载
 - 应用程序关闭
 - play in editor终结
# 2. BlueprintImplementableEvent和BlueprintNativeEvent之间有什么区别？
- BlueprintImplementableEvent是在C++中声明的函数，然后在蓝图中来实现该函数
```cpp
UCLASS()
class YOURPROJECT_API AYourClass : public AActor
{
    GENERATED_BODY()

public:
    UFUNCTION(BlueprintImplementableEvent, Category = "YourCategory")
    void YourEvent();
};
```
- BlueprintNativeEvent是在C++中声明并定义提过一个默认实现后，可以在蓝图中覆盖掉该实现，类似于虚函数override
```cpp
UCLASS()
class YOURPROJECT_API AYourClass : public AActor
{
    GENERATED_BODY()

public:
    UFUNCTION(BlueprintNativeEvent, Category = "YourCategory")
    void YourEvent();
    virtual void YourEvent_Implementation();
};
```
# 3. BlueprintCallable和BlurprintPure在什么时候使用？
BlueprintImplementableEvent和BlueprintNativeEvent是在C++中调用蓝图的实现，而BlueprintCallable和BlurprintPure是在蓝图中调用C++的实现
```cpp
---- .h ----
// 普通函数
UFUNCTION(BlueprintCallable)
void TestFuncionA(int InValue1, float InValue2, int& OutValue1, float& OutValue2);
// 纯函数
UFUNCTION(BlueprintPure)
void TestFuncionB(int InValue1, float InValue2, int& OutValue1, float& OutValue2);

---- .cpp ----
void ATestActor::TestFuncionA(int InValue1, float InValue2, int& OutValue1, float& OutValue2)
{
}
void ATestActor::TestFuncionB(int InValue1, float InValue2, int& OutValue1, float& OutValue2)
{	
}

```
![](./BlueprintCallable.png)
BlueprintCallable和BlueprintPure的区别是，如果函数的UFUNCTION中用了BlueprintPure标记则表示这是个纯函数，在蓝图这边不能看到target引脚。如果换成BlueprintCallable则表示是个普通的函数  

# 4. UE4的蓝图中对于Foreach等循环采用的是类似并行的方式，试实现一个串行的方法。

# 5. 如何解决子弹穿墙问题？
使用射线MultiLineTraceByChannel，这个射线可以穿透多个物体(返回一个数组表示所有被射中的物体)。而LineTraceByChannel只要击中一个物体就会停止。

# 6. UE4对UStruct的内存会自动管理吗？
USTRUCT宏修饰的结构体会在UHT中生成UScriptStruct(通过xxx::StaticStruct获得)，USTRUCT修饰的结构体，是不能包括UFUNCTION的(可以包含普通函数, C++的基本功能)，否则会报错USTRUCT's Cannot contain UFUNCTION，UE想把USTRUCT单纯当成一个数据，而不是类似C++中的struct和class职责划分不明确。
USTRUCT修饰的struct拥有和UObject一样的反射、序列化、网络复制，但不受GC管理。而UStruct是UScriptStruct的基类，同样不支持GC。而UClass继承于UStruct，但是在UHT的时候，会通过DECLARE_CLASS宏，将里面的属性AddReferencedObjects。    
如果想要普通的struct/class被GC管理必须继承自FGCObject对象，并实现派生的AddReferencedObjects接口，手动将UObject* 变量加入到引用。Object对象采用垃圾回收机制，被UPROPERTY宏修饰或在AddReferencedObjects函数被手动添加引用的UObject*成员变量，才能被GC识别和追踪，GC通过这个机制，建立起引用链（Reference Chain）网络。

没有被UPROPERTY宏修饰或在AddReferencedObjects函数被没添加引用的UObject*成员变量无法被虚幻引擎识别，这些对象不会进入引用链网络，不会影响GC系统工作（如：自动清空为nullptr或阻止垃圾回收）。

垃圾回收器定时或某些阶段（如：LoadMap、内存较低等）从根节点Root对象开始搜索，从而追踪所有被引用的对象。

当UObject对象没有直接或间接被根节点Root对象引用或被设置为PendingKill状态，就被GC标记成垃圾，并最终被GC回收。

注1：USTRUCT宏修饰的结构体对象和普通的C++对象一样，是不被GC管理
注2：FGCObject对象和普通的C++对象一样，是不被GC管理

# 7. 在客户端是否可以获取到AIController？
不可以,在DS(dedicated server)模型下,AIController只存在于服务端,其主要是通过在服务端对Pawn进行操控, 然后再同步到客户端。

# 8. 客户端上面能够执行RPC的对象有哪些？
RPC（远程过程调用），是在本地调用但能在其他机器（不同于执行调用的机器）上远程执行的函数。
在客户端上能够执行RPC的对象需要满足：
- 该Actor必须被复制
- 如果RPC是从客户端调用并在服务器上执行，客户端就必须拥有调用RPC的Actor。
- 如果是多播RPC是个例外：
当从客户端调用时，只是在本地运行而非服务器上执行。

# 9. 如果在C++中需要使用windows的头文件，如何操作？
```cpp
#include "AllowWindowsPlatformTypes.h"
#include <windows.h>
#include "HideWindowsPlatformTypes.h"
```
# 10. 在头文件中经常出现的*.generated.h是什么？
表明这个class被UCLASS宏标记，或者struct被USTRUCT标记，UHT会扫描该头文件，将该类加入到反射系统，记录下来UPROPERTY和UFUNCTION的偏移量，该类才能使用到UE反射系统的GC，序列化，网络同步等功能。


# 11. 对一个Actor调用AIMoveTo失败了，其可能原因是什么？
- 检查有没有添加导航体网格
- 检查场景中的碰撞是否把AI阻挡了
- 要移动的pawn和目标点或者目标actor没有正确设置
- 检查AIController是否为空
- 如果AI是生成的而不是一开始就放到场景的(把AutoPossessAI设置为PlacedInWorldOrSpawned)
[AIMoveTo失败原因](https://blog.csdn.net/qq_41410054/article/details/130870008)

# 12. 试说出宏、函数、事件的部分区别和联系。
事件是触发，事件里面的定时不会等待完成就会接着继续执行后面的逻辑
函数和宏都会等待定时完成后才能继续执行。宏和函数都可以有返回值，而事件不能有返回值。
宏不能在其他蓝图里面调用，函数事件则可以。

# 13. 试使用C++实现一个对蓝图中任意Actor排序的框架。

# 14. Blueprintable和BlueprintType的意义。
Blueprintable:将使用该宏标志的类公开为创建蓝图的可接受基类（类似于：那些base类）。
其默认为NotBlueprintable,即不可以创建蓝图子类。

BlueprintType:将使用该宏标志的类公开为可用于蓝图中变量的类型（类似于：int）。
与之对应的有NotBlueprintType，即不可以在蓝图中创建该类型的变量。

# 15. 客户端上面对一个Actor中的RPC事件调用失败，可能原因是什么？

# 16. UE4中的RPC事件有哪些？

# 17. 如何设置Actor的同步间隔？

# 18. 若需要实现一个多播事件，如何操作？

# 19. 连接服务器的命令是什么，如何传递参数？

# 20. 为什么需要TWeakPtr？

# 21. 如果要在游戏的开始和结束执行某些操作，可以在UE4哪儿处理？

# 22. UE4中，各种字符编码如何转换？

# 23. C++源文件中的注释在蓝图中显示为乱码，为什么？

# 24. 插件中的LoadingPhase是什么？

# 25. 如何切换不同的引擎版本？

# 26. 对于一个团队项目，如何处理DDC？

# 27. UFUNCTION，UPROPERTY等宏的作用是什么？

# 28. 如何给AI增加playerstate？

# 29. ProjectileComponent是否同步？若未同步，如何操作？

# 30. 若要更改某个Actor中的组件为其派生的组件，如何操作？

# 31. UE4的游戏框架包含哪些内容？

# 32. 当前UE4在移动平台上面的问题有哪些？

# 33. 如何获取UE4的源码？

# 34. UE4服务器的默认监听端口是哪一个？采用的是UDP还是TCP协议？

# 35. Tick中的帧时间是否可靠？若不可靠，如何操作？

# 36. UE4的打包方法有哪些？

# 37. 如何制作差异包或者补丁？

# 38. 试说出Selector、Sequence、Parallel的运作流程。

# 39. UE4中的AI感知组件有哪些？

# 40. 在UE4的C++中调用父类的函数，如何操作？

# 41. UE4内置的伤害接口是什么，有哪些类型？

# 42. UE4的蓝图部分在版本控制软件中无法进行比较，你是否有好的解决方案？

# 43. UE4中的联网会话节点有哪些？

# 44. UE4中的字符串有哪些？

# 45. 获取和释放角色如何操作？

# 46. 设置地图的游戏模式，有哪些方法？

# 47. 玩家操作事件放在PlayerController和Pawn中，该如何选择？

# 48. 切换关卡的命令是什么？

# 49. UE4中是否可以支持回放？如何操作？

# 50. UE4的蓝图类型有哪些？

# 51. 添加一个USTRUCT MyStruct，是否可以？

# 52. 若要C++中的属性暴露给蓝图，如何操作？

# 53. 在C++中为对象设置默认值有哪些方法？

# 54. C++中Reliable的意义是什么，该如何实现对应的操作？

# 55. 如果项目中需要专用服务器，如何操作？

# 56. UE4中的Delegates有哪些？

# 57. 如何分析性能瓶颈在哪儿？

# 58. UE4的碰撞类型有哪些？

# 59. UE4的服务器是否适应于MMO？若不适应，有什么解决方案？

# 60. 动画蓝图是否支持同步？若不支持，有什么解决方案？

# 61. 材质参数、特效参数、声音参数如何使用？

# 62. 若要对打包之后的版本进行跟踪和调试，如何操作？

# 63. C++中如何对组件或者Actor设置同步？

# 64. UE4的AA算法有哪些？

# 65. 四元数相对于欧拉角的优点。

# 66. 简述A*算法。

# 67. UE4中需要对一个原本不支持寻路的Actor实现寻路功能，如何实现？

# 68. UI中的锚是用来干什么的？

# 69. 如何基于UE4的网络接口，实现一个网络层，如Steam？

# 70. 对于打包之后的游戏资源，有什么加密方案？

# 71. 在Actor中增加了输入事件，但是输入事件却无法触发，其原因可能有哪些？

# 72. SpawnActor的位置不对，为什么？

# 73. 在BeginPlay之后调用了某个RPC操作，客户端却没有执行到，可能原因是什么？

# 74. 在客户端没有连接到服务器之前，有什么同服务器进行通信的方案吗？

# 75. 如何在Actor中增加command命令？

# 76. 命令行中ce和ke有什么作用？

# 77. UE4中的智能指针有哪些？

# 78. 试描述你之前做过的项目中的部分功能。（没啥好说的，各个项目需求千差万别，主要目的是看有没有具体做过）

# 79. 对根组件设置Scale会有问题吗？

# 80. 如何在UE4中使用静态库或者动态库？

# 81. 试分析GameMode的运行流程，如从InitGame至Logout。（very hard， especially without source code）

# 82. UE4的自动化测试如何搞？

# 83. 多个摄像机之间如何切换？

# 84. 更新UI的方式有哪些？

# 85. 如何区分并调节不同的音效？

# 86. 如何销毁AIController？

# 87. 在C++和蓝图中如何打印调试信息？

# 88. 轴输入事件在值为0的时候会触发吗？

# 89. 3DWidget如何使用？

# 90. 游戏中的AI技术有哪些？

# 91. 导航网格和寻路组件各有什么作用？

# 92. 对于编译整个引擎耗费的大量时间，有什么解决方案？

# 93. 如何联机构建光照？

# 94. Montage是什么？

# 95. 执行动画时，将动画和声音、特效匹配的较好的方案是什么？

# 96. 如何获取动画的执行事件？

# 97. Service的执行时机是什么时候？

# 98. Observer Aborts的用途是什么？

# 99. 如何更改UE4默认的同步带宽？

# 100. UE4中的数据存取方法有哪些？
