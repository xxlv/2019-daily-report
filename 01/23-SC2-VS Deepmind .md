## 2019年1月23日 HZ 晴朗  风暴-0123

#### 意外的惊喜

看到SC2 在周五要举行Deepmind AI 对阵 人类选手啦。。。。

希望我的偶像InNovation能上场呢~~

时间在周五凌晨2点，比较晚，周六还要加班，

但是一定要看呀~~

开心__(*^▽^*)

#### 生活

有点不舒服，可能是感冒了。

昨晚跟妹子看了《我想吃掉你的胰脏》，不错的电影，但是有些地方的处理实在是不能接受，比如最好为啥会被人杀掉呀，莫名其妙，而且前面一直在铺垫有凶杀案。

里面的台词最喜欢的以及一句话，

> 我想把你指甲里的污垢煎成茶喝掉
> 我想吃掉你的胰脏

 今天碰见一个神经病司机，差点撞到行人还猛打方向，滴滴举报没人理，哼。

有点小感冒，晚上跟女票聊天~~挺开心。

房东终于把我的押金退给我了，真可恶。

于是买了一条牛仔裤，还吃了最爱吃的猪扒饭，用了5快钱的优惠券。吃了CY妹妹的2颗巧克力。


#### 工作

今天处理了一个奇怪的bug，是会话保持关闭引起的，只在IOS端出现，但是具体的原理还没想通。

重现步骤是：在一个IOS端的某个点击按钮会触发一个神秘的弹层，这个页面是内嵌H5,用的weui组件，定位到是$.toast(msg)的msg为undefined的时候，会导致弹出一个默认的提示。

导致msg为undefined的原因是 ajax的success callback中的data是undefined，这个原因可能是由于session 找不到导致执行了静默登录逻辑。

