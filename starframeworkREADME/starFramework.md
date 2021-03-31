# StarFramework使用说明

## 框架草图

![unity角色输入控制框架V1.3](D:\桌面文件\unity学习日志\CSDN博客\starframework\img\unity角色输入控制框架V1.3.png)

## 框架简介

本框架适用于unity中的人物开发，目前包括的模块有人物输入控制模块，人物状态控制模块，人物动画控制模块，人物技能控制模块，人物动画播放模块，人物音效播放模块，人物移动实现模块，本框架前身为CBAEFramework，现在正式更名为starFramework。

## 事件流

就如草图中，每一条从左侧的三大事件源走向右侧的线条，我们称为一条事件流，事件流由Controller控制（动画事件比较特殊，由unity控制），Behaviour声明，Action发布。

事件末流，草图中事件流经由事件中心发布后来到右侧，这里是我们说的事件末流，其中，当事件末流流经技能模块和动画播放器可以选择开启第二条事件流，技能模块可以成为Info，Input的控制源，动画播放器可以成为动画的控制源。

## Controller&Detection

### Controller和Detection的关系

Controller类在starFramework中负责集中管理输入和移动检测，检测类需要继承Detection，这样才可以被对应的Controller管理。starFramework中共有InputController，InfoController两个，其对应的检测基类分别为BaseDetection，BaseInfoDetection。

现在我们知道了Controller和Detection的关系，那么Detection是用来做什么的？

### Controller的职责

上面已经说过Controller负责集中管理Detection，管理的方式是重写awake周期函数并调用AddDetection方法进行Detection的注册。经过注册的Detection就可以被Controller管理。

### Detection的职责

#### InputController

InputController的Detection负责两件事：一个是输入检测，此功能需要重写ValueListener方法，InputController会每帧调用自己管理的Detection的ValueListener来检测玩家的输入。同时角色的状态机FSM也会根据这里的检测结果来切换对应的状态，当然你也可以将输入检测这个功能完全搬到FSM中。

另一个是移动检测，实现此检测不需要重写ValueListener，你只需要书写一个检测函数，并记住注册此Detection时使用的名字，InputController会向外暴露一个查询API，外部类需要使用此API来获取需要使用的移动检测。

#### InfoController

InfoController的Detection负责一件事，就是状态检测，运作原理和InputController的移动检测一样。

## Behaviour&BehaviourEvent

### Behaviour和BehaviourEvent的关系

Behaviour在starFramework中负责管理事件的集中声明，而BehaviourEvent就是一个一个的事件声明类的基类，所有事件声明都需要继承BehaviourEvent，只有这样你写的事件声明才可以被Behaviour管理。

### Behaviour的职责

负责集中管理所有的事件声明类，当我们写好一个事件声明类后我们就需要来到对应的Behaviour里重写awake生命周期函数并调用AddEvent方法注册事件声明类。

这里需要注意，由于Unity动画事件的特殊性，AnimBehaviour需要在本类中直接书写事件注册和调用。

### BehaviourEvent的职责

除开AnimBehaviour不使用BehaviourEvent外，InputBehaviour和InfoBehaviour都需要事件声明单独写成一个类并继承BehaviourEvent。

BehaviourEvent为事件声明类提供了模板，子类只需要重写这些方法，就会实现对应的功能。继承BehaviourEvent的子类在声明事件时需要使用本框架提供的MyEventHandler<>作为标准事件使用，执意使用EventHandler会导致报错。

本框架建议所有的事件声明都使用继承自EventArgs的参数。

## Action&ActionEvent

### Action和ActionEvent之间的关系

Action和ActionEvent的关系类似于Behaviour和BehaviourEvent的关系，Action作为管理类集中管理所有继承ActionEvent的子类，我们已经知道继承自BehaviourEvent的子类负责事件声明。而继承ActionEvent的子类负责事件的发布，在事件发布上我们使用一个单例事件中心来实现。

### Action的职责

负责集中管理所有的事件发布类，当我们写好一个事件发布类后我们就需要来到对应的Action里重写awake生命周期函数并调用AddEvent方法注册事件发布类。

### ActionEvent的职责

这里我们将ActionEvent具体的分为输入事件发布类InputActionEvent，动画事件发布类AnimActionEvent和状态事件发布类InfoActionEvent，他们将各司其职，为他们的子类提供规范。子类只需要重写这些方法，就会实现对应的功能。

当你书写ActionEvent子类的时候你会发现ActionEvent中也有名为AddListener和RemoveListener的方法，没错，你需要在这里对事件进行注册，而需要注册的behaviourEvent我们会要求你在注册ActionEvent时传入其名称。

## ActionFuncRealizer&ActionMethod

### ActionFuncRealizer和ActionMethod的关系

ActionFuncRealizer我们叫做人物的移动实现模块基类，负责为具体的移动实现类提供一个规范。ActionFuncRealizer负责集中管理所有继承ActionMethod的子类，创建移动组件。继承ActionMethod的子类主要负责调用人物移动，使用移动组件。

### ActionFuncRealizer的职责

前面说过，ActionFuncRealizer负责集中管理ActionMethod，所有注册的ActionMethod都会被纳入对应ActionFuncRealizer的管理中。ActionFuncRealizer为了准确的创建移动组件。同时也要存储所有的ActionMethod。需要传入两个类型，一个接口，一个枚举。

### ActionMethod的职责

ActionMethod主要负责调用移动代码和创建移动组件，可见ActionMethod并不负责具体的移动实现，这时你还记得在使用ActionFncRealizer，和ActionMethod时要求传入的类型吗？一个接口，一个枚举，这两个一同决定了我们的ActionMethod需要使用一个什么样的移动组件。其中接口决定了移动的种类，是行走，是飞行，还是跳跃。枚举决定了怎么行走，怎么飞行，以及怎么跳跃。

## AnimationPlayer&AnimationDetection

### AnimationPlayer和AnimationDetection的关系

AnimationPlayer和AnimationDetection的关系等价于Controller和Detection之间的关系，实际上由于unity动画事件是由unity调用，而动画的播放由AnimationPlayer来控制，所以AnimationPlayer间接的等价于一个Controller。

### AnimationPlayer的职责

和Controller一样负责集中注册管理AnimationDetection。

### AnimationDetection的职责

AnimationDetection负责在特定时机播放正确的动画，同时由于一些情况下的动画不能由输入检测控制（比如下落状态），所以我们会需要使用状态机的生命周期函数来解决这个问题。

## SoundPlayer&SoundDetection

### SoundPlayer和SoundDetection的关系

SoundPlayer和SoundDetection的关系设计参考了AnimationPlayer，但是却不同于AnimationPlayer。虽然AnimationPlayer和SoundPlayer实际上都属于一条事件流的末端，但Animation可以使用动画事件来开启第二条事件流，SoundPlayer却不能。而SoundPlayer通常会由动画事件来调用，在合适的时机来播放合适的音效。

## SkillSystem

技能框架采用技能动态组合的方式，在我们需要释放一个技能时由技能释放器根据传入的技能信息动态组合技能效果，本框架的技能系统向外提供一个技能外观类，可以被框架在任何时间调用，并不需要拘束于事件流使用。

## FSM

状态机，负责集中管理角色状态，是角色状态控制模块的一部分，状态机中的状态向外提供了向其生命周期函数添加内容的委托，可以供我们在其他模块中选择使用。

