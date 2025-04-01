# Lyra

## Steps

### ULyraDeveloperSettings是怎么配置生效的？

UCLASS(config=EditorPerProjectUserSettings，自动读取ini，继承于UDeveloperSettingsBackedByCVars，部分可在console修改

### L_ShooterPerf怎么控制机器人数量？

场景Actor，LyraPlayerStart，TryClaim断点，其中一个是玩家ULyraPlayerSpawningManagerComponent::ChoosePlayerStart，其余是ULyraBotCreationComponent::SpawnOneBot。

搜索调用点，B_ShooterBotSpawner_Perf（ULyraBotCreationComponent子类
）里的Event ServerCreateBots（覆盖了ULyraBotCreationComponent::ServerCreateBots，BlueprintNativeEvent
，C++实现会读取ULyraDeveloperSettings）。蓝图里NumBotsToCreate覆盖为20（父类是5）。
B_ShooterBotSpawner_Perf是在B_ShooterGame_Perf里Actions引用的。

### B_ShooterGame_perf是在哪里加载的？

DA_ShooterGame_ShooterPerf里指定了Map ID（L_ShooterPerf）和Experience ID（B_ShooterGame_Perf），需要进一步看ULyraUserFacingExperienceDefinition是如何使用

定位ABP是ABP_Mannequin_Base
运行调试会提示，Post process Animation Blueprint正在运行（ABP_Manny_PostProcess）
后处理动画蓝图是以主蓝图输出为输入，在Skeletal Mesh中设置，做一些通用辅助性的后处理

## 异步更新
EventGraph为空，使用BlueprintThreadsafeUpdateAnimation更新（不在Game Thread）
BlueprintThreadsafeUpdateAnimation要访问Game Object要通过Property Access（线程安全）
大概原理是，动画在Game Thread Tick时，会根据蓝图的反射，把需要的属性复制一份。Property Access实际访问的是这个复制后的值。
这里调用的方法也要求是Thread Safe，主要是存储各种变量。

## Linked Anim Graph
相当于父类模板方法，子类可以定制部分实现，但这里实际并不是继承关系，而是Link（要主动链接）。最朴素的是直接拖一个Linked Anim Graph节点，与调用一个方法类似，只不过此时是调用另一个ABP。

完整的流程：
- 创建Animation Layer Interface（ALI_ItemAnimLayers），添加多个Animation Layer（类似于接口，可以定义输入），非Default Group的所有Layer会共享同一个Anim Instance（直接在Details里输入Group的名字就行，没有新建Group入口）；
- ABP（ABP_Mannequin_Base）Class Settings中Implemented Interfaces加入ALI_ItemAnimLayers；
- ABP中可以直接把这些Layer拖出来使用（与调用函数类似），Layer可以在主ABP中实现，也可以留待子ABP填充；
- Layer的覆盖实现，均是ABP_ItemAnimLayersBase子类，例如ABP_PistolAnimLayers。逻辑基本上都在父类实现，子类修改Class Defaults中的各种变量（主要是Anim Set）。此外，在Asset Override中还可修改Anim Graph Override（能修改的都是UE自动导出的，如Sequence Player）。
- 主蓝图与Layer实现的组合，是通过Link Anim Class Layers节点（有对应的Unlink）。这一步应该是B_WeaponInstance_Base里做的，后续再分析调用。

## Locomotion

### Idle & turn in place

- idleBreak: (On Become Relevant) SetUpIdleBreakAnim，根据CurrentIdleBreakIndex从Idle_Breaks中选取要播放的动画，CurrentIdleBreakIndex = （CurrentIdleBreakIndex + 1） % Idle_Breaks.length()；
- idle: 
    + (On Become Relevant) SetUpIdleState
        * ChooseIdleBreakDelayTime
        * ResetIdleBreakTransitionLogic
    + (On Update) UpdateIdleState
    + idleStance
        * idle: (On Update) UpdateIdleAnim
        * StanceTransition: (On Become Relevant) SetupIdleTransition


## Misc

## Animation Fast Path
蓝图节点运行时右上角有⚡️标识。对于某些简单输入（成员变量，布尔取反，Struct成员），能自动优化避免进入蓝图虚拟机提升性能。
- 在Project Settings中Anim Blueprints，开启Optimize Anim Blueprint Member Variable Access（默认打开）；
- 在ABP的Class Settings中打开Warn About Blueprint Usage，没有通过Fast Path访问的会报warning。

## 节点左上角有👀标识
右键Toggle Pose Watch

## Output Animation Pose节点中On Become Relevant作用
On Initial Update只会执行一次，就是首次被激活的时候，而且必定在On Become Relevant之前。On Become Relevant则是每次被激活，节点需要计算的时候都会执行。On Update则是相关时连续执行。
