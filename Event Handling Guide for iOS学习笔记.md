#Event Handling Guide for iOS
1. iOS中的事件：事件是一个被发送给 app，用以通知 app 用户动作的对象。许多事件都是 UIEvent 类的实例。每一个事件对象都有一个类型
2. iOS中内置的手势识别： 

  手势| UIKitClass
  ---|------
 点按（任何次数的点击）|UITapGestureRecognizer
 捏合/放（对一个视图进行缩放）|UIPinchGestureRecognizer
 平移或拖动|UIPanGestureRecognizer
 轻拂（在任意方向上）|UISwipeGestureRecognizer
 旋转（手指往不同的方向滑动）|UIRotationGestureRecognizer
 长按|UILongPressGestureRecognizer
3. 在 iOS 7之后，如果 iOS 认为一个由屏幕底部开始的触摸应该显示控制中心的话，这个手势就不会被传递到当前运行的 app 。如果 iOS 认为这个触摸不应该显示控制中心的话，这个手势到达 app 的时间会有一些延迟。
4. 手势要么是分离的，要么是持续性的。

	* 一个分离的手势，是说这个手势只发生一次，不会持续。分离的手势只会给其目标（Target）发送一次消息。
	* 一个持续性的手势会在一段时间内持续发生。持续性的手势会持续给其目标（Target）发送消息，直到多点触控序列结束。

5. 使用内建的手势识别时需要做到以下三点：

	* 创建并设置一个手势识别（gesture recognizer）实例
	* 将其附着（attach）到一个视图上
	* 实现处理手势的动作方法

6. 手势识别的状态机:

	![手势识别的状态机](https://developer.apple.com/library/content/documentation/EventHandling/Conceptual/EventHandlingiPhoneOS/Art/gr_state_transitions_2x.png)

	UIGestureRecognizerStateEnded 状态即为 Recogize状态（当用户的最后一个手指离开这个view的时候会转换到这个状态）


7. 每当手势识别器的状态改变时，手势识别器都会向其目标（Target）发送一个动作消息（除非其状态改变到 cancel 或 recogined 状态）。

8. 默认情况下，哪个手势识别器首先接收触摸事件时不确定的。但是可以通过以下的方式覆写这些默认的行为：
	* 指定一个手势识别器在另一个手势识别器之前去分析一个触摸
	
		当一个手势识别失败后在让另一个手势识别器去分析这个触摸。使用如下这两个方法：
		
			gestureRecognizer:shouldRequireFailureOfGestureRecognizer:
			gestureRecognizer:shouldBeRequiredToFailByGestureRecognizer:
	* 允许两个手势识别器同时运行
		
		实现 `UIGestureRecognizerDelegate` 的 `gestureRecognizer:shouldRecognizeSimultaneouslyWithGestureRecognizer:` 方法
		指定两手势识别器之间单向的关系，使用：
		
			canPreventGestureRecognizer:
			canBePreventedByGestureRecognizer:
		方法，这样的 prevent 就会是单向的
	* 组织一个手势识别器分析触摸
		
		可以通过向手势识别器添加 delegate 来改变手势识别器的行为，可以使用如下的两个可选的 delegate 方法来组织一个手势识别器分析触摸：
		
			gestureRecognizer:shouldReceiveTouch:
			gestureRecognizerShouldBegin:
		gestureRecognizer:shouldReceiveTouch: 会在每次有新触摸的时候被调用
		如果你需要越长越好的时间来确定一个手势识别器是否应该分析触摸，使用`gestureRecognizerShouldBegin:`方法，这个方法将会在手势识别器的由 possible 状态转换到其他状态时被调用。也可以使用 UIView 的 `gestureRecognizerShouldBegin:` 方法（当 view 或者 view controller 不能成为 delegate 时）
		
9. 一个多点触控事件是由类型为 UIEventTypeTouches 的 UIEvent 来表示的。当手指运动时，iOS 将 UITouch 对象传递给这个 event。
10. 一个 UITouch 对象代表了一个手指，当手指运动时，UIKit 会追踪这个手指并改变 UITouch 对象的属性（触摸阶段、在视图中的位置、先前的位置、时间戳等）。
	触摸阶段是由 UITouch 对象的 phase 属性来保存的。
	
	![多点触摸序列和触摸阶段](https://developer.apple.com/library/content/documentation/EventHandling/Conceptual/EventHandlingiPhoneOS/Art/event_touch_time_2x.png)

11. 默认的触摸事件传递路径：当一个触摸发生时，这个 touch 对象由 UIApplication 对象传递到 UIWindow 对象。然后window对象在将这些 touch 对象集合发送给这个触摸发生的视图之前，首先将这些touch 对象集合发送给触摸发生的视图（或者其父视图）上附着的任何手势识别器。

	![默认的触摸事件传递路径](https://developer.apple.com/library/content/documentation/EventHandling/Conceptual/EventHandlingiPhoneOS/Art/path_of_touches_2x.png)

	触摸的消息序列：
	![触摸的消息序列](https://developer.apple.com/library/content/documentation/EventHandling/Conceptual/EventHandlingiPhoneOS/Art/recognize_touch_2x.png)
	
12. [手势的3个容易混淆的属性 cancelsTouchesInView/delaysTouchesBegan/delaysTouchesEnded:](http://blog.csdn.net/fys_0801/article/details/50605837)		
13. 创建自定义的手势识别
	1. 创建 UIGestureRecognizer 的子类，在子类的 .h 文件中加入 `#import <UIKit/UIGestureRecognizerSubclass.h>`这句

	2. 在 .h 文件中加入如下的方法声明：
	
			- (void)reset;
			- (void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event;
			- (void)touchesMoved:(NSSet *)touches withEvent:(UIEvent *)event;
			- (void)touchesEnded:(NSSet *)touches withEvent:(UIEvent *)event;
			- (void)touchesCancelled:(NSSet *)touches withEvent:(UIEvent *)event;
		在每个方法的实现中，都要调用父类的实现。
	3. 实现上述的几个方法，在实现的时候，最重要的是正确、合适的设置自定义的手势识别器的 state 属性。
	
	详细细节请查阅：[WWDC 2012: Building Advanced Gesture Recognizers.](https://developer.apple.com/videos/play/wwdc2012/233/)
	
14. 当用户产生了一个事件（event），UIKit 创建一个对象，这个对象包含处理这个事件所需要的信息，然后将这个信息当前活动的 app 的事件队列。
15. 事件将沿一条特定的路径传递，直到传递到一个可以处理它的对象。首先，UIApplication 单例对象将会从队列的首部取一个事件将其分发出去进行处理。通常，这个事件将会被分发到 app 的主窗口（key window），它将事件传递个一个初始对象进行处理。这个初始对象取决于事件（event）的类型。

	* 触摸事件：对于触摸事件，窗口对象会将其首先传递到触摸发生的视图，这个视图叫做 hit-test view。寻找 hit-test view 的处理被称作 hit-testing。
	* 运动和远程控制事件：对于这些事件，主窗口会将晃动事件或远程控制事件分发给第一响应者进行处理。

16. hit-testing内部机制： iOS 使用 Hit-testing 来找出触摸是在哪个视图发生的。hit-testing 需要检查一个触摸是否是在相关的视图对象的边界之内，如果是在这个视图的边界之内，还要递归的对这个视图的所有子视图进行检查。在视图层级中包含这个触摸点的层级最低的视图就叫做 hit-testing view。
17. `hitTest:withEvent:` 通过给定的 CGPint 和 UIEvent 来返回hit-testing view。`hitTest:withEvent:` 开始会调用 `pointInside:withEvent:` 方法，如果传入给 `hitTest:withEvent:` 方法的 CGPoint 在视图的边界之内，`pointInside:withEvent:` 返回 YES。然后这个方法会在这个视图的每个返回 YES 的子视图上递归的调用 `hitTest:withEvent:` 方法。
18. 响应者链从第一响应者开始，到 application 对象结束。响应对象是UIResponder 的子类的实例。UIView 是 responder 对象，CALayer 不是 responder 对象。
19. 一个对象想要成为第一响应者需要做如下的事情：
	* 覆写 `canBecomeFirstResponder` 方法，返回 YES
	* 接受  `becomeFirstResponder` 方法，如果需要的话，一个对象可以自己向自己发送这个方法。
	通常会在覆写的 `viewDidAppear:` 方法中调用 `becomeFirstResponder` 方法。
20. 事件不是唯一依赖于响应者链的对象。响应者链还用下以下的地方：
	* Motion events（运动事件）：为与 UIKit 一起处理晃动事件，第一响应者必须实现 UIResponder 类的 `motionBegan:withEvent:` 或 `motionEnded:withEvent:` 方法
	* Touch events(触摸事件)
	* Remote control events（远程控制事件）：为处理远程控制事件，第一响应者必须实现 UIResponder 类的 `remoteControlReceivedWithEvent: ` 方法
	* Action messages：当用户操作一个控件，并且动作的目标（Target）是`nil`，这个消息将被传递到由第一响应者开始的响应者链（第一响应者可以是控件本身）
	* Editing-menu messages：当用户点击编辑菜单的命令，iOS 使用响应者链来寻找实现了需要方法的对象（比如：`copy:`、`cut:`、`paste:`）
	* Text editing：当用户点击 text field 或 text view，这个视图将会自动成为第一响应者。默认情况下，将会弹出虚拟键盘，你可以使用一个自定义的输入空间来代替这个虚拟键盘。你也可以向任何响应者对象添加自定义的输入视图。
	
	UIKit 会在用户点击 text field 或 text view 的时候令其自动成为第一响应者，app 必须调用 `becomeFirstResponder` 方法来设置其他第一响应者对象。

1. iOS 中的响应者链：
	![iOS 中的响应者链](https://developer.apple.com/library/content/documentation/EventHandling/Conceptual/EventHandlingiPhoneOS/Art/iOS_responder_chain_2x.png)
	
	不要直接向 nextResponder 发送消息，调用父类的该事件处理方法，让 UIKit 处理事件在响应者链中的传递。