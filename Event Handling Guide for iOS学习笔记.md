#Event Handling Guide for iOS
1. iOS中的事件：事件是一个被发送给app，用以通知app用户动作的对象。许多事件都是UIEvent类的实例。每一个事件对象都有一个类型
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
		
		实现 UIGestureRecognizerDelegate 的 gestureRecognizer:shouldRecognizeSimultaneouslyWithGestureRecognizer: 方法
		指定两手势识别器之间单向的关系，使用：
		
			canPreventGestureRecognizer:
			canBePreventedByGestureRecognizer:
		方法，这样的 prevent 就会是单向的
	* 组织一个手势识别器分析触摸
		
		可以通过向手势识别器添加 delegate 来改变手势识别器的行为，可以使用如下的两个可选的 delegate 方法来组织一个手势识别器分析触摸：
		
			gestureRecognizer:shouldReceiveTouch:
			gestureRecognizerShouldBegin:
		gestureRecognizer:shouldReceiveTouch: 会在每次有新触摸的时候被调用
		如果你需要越长越好的时间来确定一个手势识别器是否应该分析触摸，使用gestureRecognizerShouldBegin:方法，这个方法将会在手势识别器的由 possible 状态转换到其他状态时被调用。也可以使用 UIView 的 gestureRecognizerShouldBegin: 方法（当 view 或者 view controller 不能成为 delegate 时）
		
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