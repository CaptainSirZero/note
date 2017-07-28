#Swift(3.1)的笔记整理如下
一个Swift3.1版本的新浪微博客户端
##一.项目部署

####1.项目部署到oschina上：
 
```
1.在oschina上新建一个代码仓库
2.在电脑中创建一个文件作为本地代码仓库，打开终端，将oschina上的代码仓库clone下来，使用终端cd 到电脑的这个文件夹，然后敲git clone 后面跟上oschina中代码的http路径，就会将代码仓库下载到这个文件夹
3.由于oschina上没有swift的gitignore，需要手动添加，可以到GitHub上找swift.gitignore文件，找到后打开文件，把里面的内容复制，然后通过终端创建.gitignore(touch . gitignore命令)，打开后把刚才复制的内容粘贴到这个新建的文件即可，建议将Pods/打开，将Pods/前面的#号删除掉即可
4.将新建的.gitignore添加到暂缓区，命令:git add .
5.添加到本地文件仓库,命令:git commit -m "添加忽略文件",双引号中写操作的注释
6.添加文件到服务器中，命令:git push
7.使用Xcode创建项目，保存到刚才创建的本地代码仓库中，注意:项目名称最好不要有乱七八糟的符号(比如01-，！什么的)，尽可能使用纯英文的名称
8.最后在Xcode中点击commit，并写上注释比如"初始化项目"，再勾选push，就将该项目部署到oschina上了
```


####2.初始化项目

```
1.最低部署9.0
2.只支持竖屏
3.添加启动图片和应用图标
4.设置Bundle name
```



##二.搭建项目的主框架

####方式(一): 纯代码的方式搭建主框架
#####1.使用主流框架搭建

```
1.自定义tabBarController用于window的根控制器，
2.创建各个界面对应的控制器，并将其添加到tabBarController的子控制器数组中
3.由于公司提供了tabBarController的子控件的json描述文件，所以我们通过json文件来添加
4.导入json文件到项目中，注意:导入json文件后，文件后面有❓，说明没有添加到git的管理之下，可以右键文件，点击scorce control ，在选择add即可，如果在Xcode中提交失败的话，可以使用终端命令行提交试试
5.自定义一个方法用于通过字符串来创建控制器的,注意:需要在字符串前面加上【命名空间.】
6.在viewDidLoad方法中，解析json数据，步骤:
	(1)需要先获取json文件在mainBundle中的路径
	(2)再通过路径将获取json文件中的内容(这里先转换为二进制Data类，由于我对swift还不熟悉，所以只能先将json路径转换为URL，然后在通过URL来转换为Data)
	(3)通过序列化JSONSerialization将Data数据解析为数组。
```


#####2.Swift处理异常的三种方式
> 如果在调用某个方法时，该方法最后有一个throws，说明该方法会抛出异常，如果一个方法会抛出异常，那么需要对该方法进行处理
>
>  在Swift中提供了三种处理异常的方法:

```
// 比如正则表达式:
// 1.创建正则表达式的规则
let pattern = "abccc"
// 方式一:try方式，程序员手动捕捉异常
/*
// 2.创建正则表达式
do {
    let regext = try NSRegularExpression(pattern: pattern, options: .allowCommentsAndWhitespace)

} catch {
    // 系统提供了临时变量error，通过error可以获取到错误信息
    print(error)
}
*/
// 方式二:try?方式(建议使用)，系统帮助我们处理异常，如果该方法出现了异常则返回nil，如果没有异常则返回对应的对象
/*
guard let regxt = try? NSRegularExpression(pattern: pattern, options: .allowCommentsAndWhitespace) else {
    return
}
*/

// 方式三:try!(不建议使用、危险),直接告诉系统该方法没有异常，注意:如果出现了异常，那么程序会直接崩溃
let regxt = try! NSRegularExpression(pattern: pattern, options: .allowCommentsAndWhitespace)
```
####方式(二): 使用storyboard搭建主框架
```
1.这里需要使用到iOS9推出的storyboard reference技术
2.应用程序启动时加载main.storyboard，在main.storyboard中添加子控制器，这里需要先拖入一个tabBarController，再拖入4个navigationController，让他们成为tabBarContoller的子控制器，并给navigationController添加跟控制器为tableViewController
3.由于在一个storyboard中管理多个控制器比较麻烦，不好找到对应的控制器，所以我们可以使用storyboard reference技术(选中这个控制器，点击Xcode菜单-Editor-Refactor to Storyboard,就会分离出一个storyboard)，将每个导航控制器单独创建为一个storyboard，并给导航控制器的跟控制器添加对应的class来管理
tabBar上面按钮的布局的方式
```

#####1.特殊说明,微博终点 + 号
```
1.由于我们需要在tabBar中间插入一个➕按钮，所以应该在tabBar上面预留一处空白方便添加➕按钮的
方式(一) 自定义UITabBar，通过KVC给UITabBarController设置为自定义的tabBar
方式(二) 在main.storyboard中，在tabBarController中插入一个空的子控制器，注意：如果想要让预留的地方在中间位置，需要把空白的控制器放在tabBarController的第二位，然后删除这个控制器的item名称，并且设置这个控制器的item按钮的enable属性为NO
2.在storyboard中也可以设置tabBar的选中图片，这里我们通过代码设置
在自定义的tabBarController中，遍历items数组中所有的item(由于items是可选类型，所以需要强制解包，从storyboard中创建的tabBarController所以不用怕items为空的情况)，设置每个item的选中图片，注意:调整tabBar的item的属性需要在viewWillAppear中调整，不要在viewDidLoad方法中调整，因为在viewDidLoad中调整的话，调用viewWillAppear时，又会将tabBar的item调整为storyboard中的状态
3.tabBar中添加➕按钮，使用UIButton添加到tabBar中，可以在tabBarController的viewDidLoad中添加，按钮可以使用懒加载的方式
4.抽取封装代码:添加一个extension，专门设置UI界面，把设置UI界面的代码，在extension添加相关函数，抽取在里面
5.注意:Swift3的extension中不能访问当前文件非extension定义的私有属性和方法
6.swift中没有分类，用extension代替了
7.创建一个swift文件，用来给UIButton做扩展，步骤:(1)新建一个文件夹命名为Tools专门来管理，在Tools中新建一个category文件，然后按command+n新建一个swift文件，这个文件名称可以随便命名，但是尽量能看懂(比如UIButton-Extension),注意:将文件默认的Foundtion框架换位UIKit框架，不然写代码没提示哦，然后就写上 extension UIButton {}，在{}中写扩展的函数，建议写类方法
8.swift中类方法是以class开头的，类似与OC的+开头的方法，格式： class func 函数名(参数名: 参数类型)  -> 返回值类型 { 这里面实现函数 }
9.我们还可以给UIButton扩展一个便利构造函数(初始化)，以init开头，构造函数不需要写返回值
10.便利构造函数
 convenience : 便利,使用convenience修饰的构造函数叫做便利构造函数
 便利构造函数通常用在对系统的类进行构造函数的扩充时使用
 便利构造函数的特点
	 1.便利构造函数通常都是写在extension里面
	 2.便利构造函数init前面需要加载convenience
	 3.在便利构造函数中需要明确的调用self.init()，比如以下代码

 convenience init(imageName: String, backGroundImageName: String) {
    self.init()
    setImage(UIImage(named: imageName), for: .normal)
    setImage(UIImage(named: imageName + "_highlighted"), for: .highlighted)
    setBackgroundImage(UIImage(named: backGroundImageName), for: .normal)
    setBackgroundImage(UIImage(named: backGroundImageName + "_highlighted"), for: .highlighted)
    sizeToFit()
}

11.事件监听注意事项:
 监听加号按钮的点击事件，点击事件调用的方法专门在tabBarController文件下写一个extension来管理
	 事件监听的本质，就是发送消息，但是发送消息是OC的特性
	 发送消息的过程:将方法包装成@SEL --> 去类中查找方法列表 --> 根据@SEL查找imp指针(函数指针) --> 执行函数
	 如果swift中将一个函数声明成private，那么该函数不会添加到方法列表中，
	 如果在该函数前面加上@objc， 那么该函数依旧会被添加到方法列表中
	 如果调用的private声明的函数不在一个class的{}大括号中或extension的{}大括号中，会报错的
```
 
####三.【访客视图】的逻辑
```
当用户打开程序且并没有登录账号时，给用户呈现访客视图，而不是我们的主界面
最终方案:
搞一个Bool变量，判断有没有登录，比如isLogin，当isLogin = flase(没有登录) -->展示访客视图，当isLogin = true(已经登录) --> 展示正常的信息
每个界面的访客视图特点都长得差不多
1.创建一个baseViewController类作为基类，让tabBarController的子控制器都继承自这个类，
2.在这个类中定义一个变量isLogin，并赋值为flase(因为默认情况下肯定是没有登录账号的)
3.定义一个私有函数初始化访客视图，并将访客视图赋值给控制器view，访客视图需要懒加载，让访客视图为一个普通的UIView
4.重写loadView方法，在loadView方法中用三目运算符判断 isLogin为ture时就调用super.loadview(),如果为flase时，就调用自定义的哪个方法
最终实现原理:当用户登录时就会加载系统默认的控制器的view，进入登录界面，如果用户没有登录就会调用初始化访客视图的函数，并将访客视图设置为控制器的view，最终当前app中会存在4个访客视图，因为有4个控制器继承了baseViewController，每个控制器的view加载时都会调用loadView方法，而loadView方法中会执行super.loadView()函数，当用户没有登录时最终会调用baseViewController的loadView方法创建访客视图
5.自定义VisitorView(访客视图)，继承自UIView，并使用XIB搭建【访客视图】
6.让VisitorView类和XIB绑定，在VisitorView类中定义一个类方法，用于创建一个从XIB加载的VisitorView，然后在baseViewController类中的懒加载访客视图时直接从VisitorView类创建即可
7.布局访客视图界面
 - 首页:有一个转盘，转盘使用UIImageView，按【command  =】可以让图片按原始尺寸显示，给转盘添加约束，固定宽高，转盘上面有一个遮罩遮住了下面小部分，这个遮罩其实是UI提供的图片，这时也添加一个UIImageView，给遮住添加约束，固定高度为400(只要正好遮住转盘底部即可)，X和Y为0，这时再设置遮罩imageView的背景颜色为238、238、238即可
    -当给按钮设置背景图片时，不希望图片拉伸处理，可以在图片管理器中选中图片-找到slices，选中水平和垂直拉伸即可
    
8.设置访客视图中的内容:由于这一设置完以后每个界面的访客视图长得都一样的，所以我们需要拿到访客视图中的需要变化控件(拖线到VisitorView类)，该隐藏的隐藏，该显示的显示(这里需要设置【转盘、logo、文字描述】)
9.在visitorView类中对外提供一个函数，用于外界调用时设置访客视图的控件的属性，方法中直接拿到参数赋值给控件，并且函数里面设置转盘的hidden为ture(意思是只要外界调用这个方法，就让转盘隐藏)，因为只有home界面的访客视图有【转盘】，可以不调用这个方法
10.这样设置后会产生几个问题
	(1)由于在xib中给log设置了宽和高的约束，导致其他界面设置的log图片大小不一样时会造成图片显示不全
	解决方法:给UIImageview的contenMode设置为Center(居中显示，并且按照图片原来的宽高比显示)
	  
	(2)文字描述控件:换行时文字居中显示
解决方法:设置文字居中显示
11.让转盘旋转起来:给转盘对应的UIImageView控件做旋转动画
	(1)定义一个函数来做转盘动画旋转，旋转是一直在转，并不会停下来
	(2)创建CABasicAnimation基本动画，添加到转盘的layer上
	(3)产生问题:当退到后台、或转盘所在的view消失后，在进入转盘所在的view，动画会结束且不转
	(4)解决方法:核心动画都存在这种问题，这是因为动画结束后或view消失后，系统会帮我们剔除动画，我们可以设置移除核心动画即可
```


#####【导航条】及【导航按钮的】设置
```
在父类BaseViewController方法中设置导航条上的按钮
1.注册和登录按钮
2.在父类BaseViewController中添加extension，用于写事件监听的各种方法的
3.设置导航条上所有按钮的颜色为橙色，由于新浪所有的都是橙色，可以在appDelegate类中，设置UINavigationBar的全局颜色为橙色
4.每个访客视图上的注册和登录按钮，也需要添加点击事件，由于直接脱线监听方法到访客视图的VisitorView类中，不方便出来点击事件，所以我们可以将【注册】和【登录】按钮拖线属性到VisitorView类中，那么在manViewController中可以拿到这两个属性，给其添加点击事件，这样可以让代码更简洁，而且在这里我们可以直接调用导航条上注册和登录按钮的监听事件防范
5.点击导航条上的BarbuttonItem时，会自动切换为选中状态，那么这些item是需要设置UIBarButtonItem的CustonView的，设置CustonView为UIButton即可
6.导航条的标题应该设置titleView
(1)在每个子类里面设置导航BarButtonItem的属性，但是需要现在判断【当没有登录时，直接返回】，因为没有登录就会加载【访客视图】，所以没有必要设置
7.设置导航的titleView为UIButton，创建UIButton最好使用懒加载的方式，注意:titleView上按钮有两个状态下的图片(是箭头)，默认情况下箭头朝下，点击时箭头朝上(设置按钮为选中状态 )，设置完成后按钮的【图片在左边】【文字在右边】了，我们需要的是文字在左边，图片在右边 
解决方法:自定义UIButton，像有些设置UIButton固定属性的可以写在自定义的类中，重写init(frame:)函数，
注意:(1)必须要调用super.init(frame: frame)，(2)swift中规定:重写控件的init(frame)函数或init()函数，必须重写init?(coder aDecoder)函数(这个函数中可以什么都不实现)，因为一旦重写init方法表示我在这个方法内做了事情，加入这个控件是从xib创建的会调用init?(coder aDecoder)函数，那么之前重写的init中做的事情就没有意义了，按照系统默认实现就可以
切换按钮中imageView和label的位置，重写layoutSubviews函数，swift中可以修改对象结构体中成员属性的，所以可以直接拿到控件的x值进行修改
8.点击titleView时弹出一个小窗口(是一个控制器)，监听titleView上Button的点击事件，在监听事件的函数中弹出这个控制器
9.自定义弹出的这个控制器:让其继承自UIViewController，由于界面是固定的，可以在使用XIB描述这个控制器的view，背景是一个图片，添加imageView设置约束为上下左右0，然后给imageView设置图片，此时产生问题:图片拉伸严重
解决方法:设置图片只拉伸中间的1个点像素，左右不需要拉伸
10.使用modal弹出这个自定义控制器的问题:当modal弹出控制器后，以前显示的控制器的view会被移除(就是黑色的)，我们需要保证以前的控制器view不被移除
解决方法:设置要弹出的控制器的modal样式为自定义即可(可以通过小面包工具查看是否有之前的控制器view) popoverVc.modalPresentationStyle = .custom
11.改变弹出控制器的尺寸，并且弹出不应该从底部弹出，应该从titleView底部位置向下弹出，需要自定义转场动画(设置转场的代理)
 
解决方法:
	(1)在点击titleView按钮监听事件的方法中，设置转场的代理transitioningDelegate为XYHomeViewController，并遵守协议(可以专门在当前控制器中写一个XYHomeViewController 的 extension，遵守协议，写协议的方法)
	(2)实现改变弹出控制器的尺寸  目的:改变弹出view的尺寸

func presentationController(forPresented presented: UIViewController, presenting: UIViewController?, source: UIViewController) -> UIPresentationController? {
// presented: 已经弹出的控制器    XYWeiBo.XYPopoverViewController
// presenting: nil
// 需要自定义一个XYPresentationController类，继承UIPresentationController类
return XYPresentationController(presentedViewController: presented, presenting: presenting)
}

(3)需要自定义XYPresentationController类，继承UIPresentationController类,modal就是把要modal出来的控制器的view，被添加到containerView(容器视图)里面,所以需要在自定义的类中重写containerViewWillLayoutSubviews方法，在这个方法中拿到从containerView中拿到弹出的控制器的view，修改弹出控制器的view的frame即可
(4)让弹出的containerView有灰灰的蒙版，而且点击背景就会disMiss，由于containerView所有的属性及约束都是系统设置的所以最好不要去修改containerView的属性，这样可能会出现问题
解决方法:给containerView上添加一个view，设置这个view的属性，蒙版view最好是懒加载,并给蒙版添加点击手势
注意:给一个控件的frame赋值其他控件的bounds时，比如view.frame = imageView!.bounds，imageView此时必须要强制解包，不然会报错，因为如果imageView?是可选类型，那么会产生的结果是可能为nil，swift是不能讲nil赋值给CGRect结构体的。写上imageView!解包就可以，因为此时imageView是确定类型，一定会有值的，所以不会报错
(5)添加蒙版到containerView上以后，弹出的控制器view不能点击了，这是因为蒙版把弹出的控制器view盖上了，所以在添加蒙版到containerView时不能使用addSubview，需要使用insertSubview，插入到containerView的第0位
(6)当点击蒙版时，让弹出的控制器退出，拿到弹出的控制器调用dismiss，pressentedViewController就是被弹出的控制器，使用它dismiss即可
(7)自定义弹出的动画:由于现在使用modal弹出动画是从底部弹出的，所以需要自定义
(8)自定义消失动画:自定义消失动画和自定义弹出动画遵守的协议都一样，其实我们可以在home控制器搞一个变量isPresented(是否弹出)，默认设置为false，当在设置自定义弹出动画的方法中让isPresented = ture，当设置自定义消失动画的方法中让isPresented = false，然后专门抽取两个方法(一个是自定义弹出动画、自定义消失动画)，然后在代理方法中使用三目运算符【isPresented ？执行定义弹出动画 : 执行自定义消失动画】
注意:当做消失的动画时，注意临界值不要为0，不然动画不会产生效果，会以下就消失了，可以设置0.0001，只要不是0就行，动画消失后，需要把消失的view从父控件中移除
(9)抽取所有自定义转场的动画到一个类中，创建一个名为XYPopoverAnimation的类继承自NSObject的类，并将所有自定义转场动画的代码剪切到这个类中，并修改扩展的类名为自定义的类XYPopoverAnimation，并且在Home控制器中，懒加载创建XYPopoverAnimation对象，设置为转场动画的代理
(10)弹出动画的尺寸应该由外界决定，而不是写死在XYPresentationController控制器中，比如Home界面中需要弹出，那这个属性应该由Home决定，所以可以在XYPresentationController对外提供一个属性来设置弹出的frame(这个frame传给XYPopoverAnimation)，然后再在XYPopoverAnimation也提供这个属性，再将这个属性传递出去(在定义这个frame时可以定义为CGRect.Zero)
(11)目前titleView的按钮存在一个问题:就是当点击containerView上的蒙版dismiss弹出的view时，titleView上的按钮的箭头不会发送变化，因为点击蒙版时并没有执行点击按钮中的方法，也就不会改变按钮的选中状态
分析步骤:根据做的是弹出的动画还是消失的动画来决定按钮的箭头朝上还是朝下(也就是选中状态)，这样做的话就不能在按钮的点击事件中设置按钮的选中状态，分析:只要XYPopoverAnimation最清楚是弹出动画还是消失动画，最终是要闭包和代理来传递都可以动画【消失】还是【弹出】，这里使用闭包来传递
解决方案:使用闭包将XYPopoverAnimation中的动画【消失】【弹出】状态传递出去，
	(1)先在XYPopoverAnimation类中定义一个闭包: var callBack : ((要接收的参数 : Bool)) -> ())? ,由于闭包没有进行初始化赋值，需要设为可选类型
	(2)在定义【弹出】和【消失】动画的方法中，将是否弹出传递给闭包
	(3)重写init()方法，参数也设置为一个闭包类型(为了外界创建当前类时，将闭包传递出去)，然后实现init方法时，将闭包参数赋值给当前类中定义的闭包
	(4)在Home控制器创建XYPopoverAnimation对象时，将闭包赋值给按钮的选中状态即可
	(5)循环引用问题解决
	注意:
		(1)如果自定义了一个构造函数，但是没有对默认的构造函数init()进行重写，那么自定义的构造函数会覆盖默认的构造函数
		(2)在闭包中如果使用当前对象的属性或调用方法，需要加self，目前有两个地方需要加self，一是当在函数中出现歧义的时候，二是在闭包中如果使用当前对象的属性或调用方法
		(3)强制解包一般是在等号右边才需要强制解包，因为当等号右边为nil时，左边的属性可能会无法接受nil，左边不需要强制解包
```


#####集成cocoaPods
```
cd到项目所在文件夹，使用git init命令创建Podfile
建议使用Xcode打开Podfile，不建议使用文本编辑器打开，容易出现符合类似的错误
注意:swift只支持动态库，不再支持静态库，添加完框架后，如果要使用框架需要导入框架(比如import SDWebImage),因为cocoaPods导入的框架和项目文件不在同一个命名空间(也就是包)中，所以需要导入框架头文件
```


#####AFNetworking单例封装
```
在公司真实开发中一般不会直接使用AFNetworking，而是在使用之前会对这个框架进行一层封装(封装为网络工具类)，然后直接使用封装好的网络工具类
封装方式:单独创建一个小项目对AFN进行封装为工具类，如果要使用的话，直接将封装的工具类拖到项目就好了，这样方便我们在小项目中进行测试，建议按照这样的封装思路
	(1)新建一个项目，使用cocoaPods集成AFN
	(2)创建一个类继承自AFHTTPSessionManager，起名为XYNetworkTools,然后在工具类中导入import AFNetworking
	(3)将网络工具类设计为单例模式:
	定义一个静态常量 static let shareInstance : XYNetworkTools = XYNetworkTools()
	实现通过shareInstance拿到的永远只有一个实例，但是XYNetworkTools()拿到的不是单例
	注意:swift中let是线程安全的，即使多线程同时访问let修饰的常量，也会保证常量只会创建一次
	(4)封装请求方法: 
设计方法时，看外界需要什么就添加对应的参数
```

#####登录功能:OAuth授权
```
当用户点击登录按钮，跳到登录界面，用户输入正确的账号和密码，点击登录后，进入用户界面
由于我们是模仿新浪微博，无法拿到真实账号和密码
使用OAuth授权的方式
分析:用户没有登录账号时，显示的界面是访客视图，当用户点击登录按钮时modal出一个登录控制器，在登录控制器上添加一个WebView，让WebView去加载网页，这个网页是新浪官方提供的，让用户在新浪官方提供的这个网页输入账号密码，点击登录按钮后，WebView会发送账号和密码到新浪服务器中，服务器验证账号密码是否正确，如果账号或密码错误，服务器会返回字段‘账号或密码错误’，如果账号密码正确，服务器会返回AccessToken(令牌)，以后都是拿AccessToken去服务器请求数据,其实服务器在返回AccessToken之前是返回给我们code，然后我们通过code去换取AccessToken的
```



```
- 开发步骤
- 1.注册成为新浪的开发者http://open.weibo.com
- 2.创建modal出来的网页，在Main文件中新建一个【OAuth(授权)】文件夹，将所有授权相关的都放在这个文件夹中，创建一个OAuthViewController类继承自UIViewController，由于OAuthViewController中主要是一个WebView，可以使用XIB搭建
- 3.到BaseViewController类中监听登录按钮的事件中，创建OAuthViewController，并把其包装为导航控制器(不然很难看)，然后modal出来，
- 4.设置OAuth控制器的导航条的按钮(关闭按钮、填充按钮、导航标题)
- 5.在XIB上添加一个WebView，设置约束上下左右为0，注意不要因为有导航栏有设置WebView顶部的约束为64，不需要这样，因为WebView本身继承自UIScrollView，scrollView在导航控制器下会有顶部64的内边距，将WebView的背景颜色设置为白色
- 6.先将WebView拖线到控制器，让WebView加载网页
 - 拼接登录页面的URLString
 - 新建一个常量文件XYConstant，专门用来存储常量的，将请求的固定参数写成常量
-7.设置WebView的代理，监听开始加载网页、网页加载完成、网页加载失败，当开始加载网页时开启指示器，加载完成后dismiss指示器，加载失败时提示加载失败
-8.当点击填充时，自动将账号和密码填充到账号和密码框
   -(1).书写js代码(javascript) -- 注意:jave和javascript没有任何关系
    let jsCode = "document.getElementById('userId').value='你的账户';document.getElementById('passwd').value='你的密码';"
   -(2).执行js代码
    webView.stringByEvaluatingJavaScript(from: jsCode)
-9.获取授权的code，需要拿到WebView加载的网页，判断网页的url中是否有code=，有code=就截取出code=后面的字符串，找到后不用再找了
-10.发送POST请求获取(使用code换取)AccessToken，最前请求其他数据必须使用AccessToken，发送网络请求的参数在【微博开发者网站】--->【OAuth2.0授权认证】---> 【OAuth2/access_token】里面有
-11.创建模型类UserAccount，用于保存服务器返回AccessToken里面的数据，创建init()构造方法，通过KVC将字典转为模型
-12.过期日期的处理
-13.请求用户登录的信息，使用AccessToken请求用户的信息
  注意:当用户登录成功了，将用户的账号信息保存，当用户重新启动程序时先判断有没有这个账号，再判断AccessToken有没有过期
-14.在请求完用户信息后，将UserAccount模型对象保存到沙盒的Document文件中，采用归档、解档的方法NSKeyedArchiver类，注意:使用NSKeyedArchiver，必须让需要归档或解档的类遵守NSCoding协议，重写解档和归档方法
-15.判断isLogin是true还是false，不应该是写死的，应该将UserAccount解档后来判断，同时判断UserAccount解档后是否有值和AccessToken是否过期
-16.封装一个UserAccountViewModal视图模型类，专门用来从沙盒中读写的功能用户信息，因为在loadView方法中写太多的代码不好，注意:如果这个不需要使用KVC的方法，可以让这个类谁都不要继承，这样更轻量级。
   (1)类中重写init()方法，这样当别的对象使用这个类的时候，创建出来就有从沙盒中读写的功能,在init方法中读取用户信息
   (2)将沙盒路径定义为计算属性
   (3)为了保证从沙盒中读取的UserAccount类对象不会销毁掉，这里定义一个成员变量，将沙盒中读取的对象赋值给这个成员变量
   (4)定义isLogin计算属性，返回值为Bool，先判断从沙盒中取出UserAccount类对象是否为nil，如果为nil直接返回false，再判断有没有过期日期，如果没有直接返回fasle，最后判断日期有没有过期

```
  
#####登录成功后进入欢迎界面

- 布局欢迎界面
 
```
 - 1.创建欢迎控制器，继承自UIViewController，使用XIB描述控制器view(如何选择XIB和storyboad，当需要描述多个界面且界面直接有业务逻辑时使用storyboard，如果只描述一个页面使用xib)
   -注意:swift中如果方法中有枚举值不想传，可以直接写[]中括号，如果传多个值，可以[]中写
  - 2.布局完XIB后，给欢迎界面做动画，动画主要是让用户头像从底部跳到顶部,并且让【欢迎回来】文本跟随跳动，其实设置头像距离底部的约束，在设置label距离头像底部的约束，最后改变头像约束值即可
```
- 显示欢迎界面及逻辑
- 当用户登录成功，保存完用户数据到沙盒后显示欢迎界面

```
- 1.显示欢迎界面之前，退出之前的控制器OAuth授权界面，注意dismiss时不要有动画
- 2.退出OAuth界面后，再显示欢迎界面，将欢迎界面设置窗口的跟控制器
注意:这里需要将account对象赋值给XYUserAccountViewModel中的account，因为此时XYUserAccountViewModel中的account为nil，XYUserAccountViewModel是单例对象，只会创建一次，第一次创建时去沙盒中没有找到用户信息的pllist，单例第二次不会被创建，也就不会执行去沙盒中解档了
- 3.欢迎界面动画执行完毕后，切换到主界面，设置窗口的根控制器为MainViewcontroller，注意: MainViewcontroller是从storyboard中搭建的，需要通过storyboard加载
- 4.设置用户每次进入程序时先加载欢迎界面，欢迎界面动画结束后再加载主界面，所以需要在Appdelegate中设置窗口的根控制器
- 最佳方法:在Appdelegate中声明一个defaultViewController计算属性，拿到XYUserAccountViewModel中的isLogin属性，使用三目运算符判断，当用户登录后加载欢迎界面，没登录就加载主界面(主界面是从storyboard中搭建的tabBarController，主界面在没登录时会显示访客视图)
- 未解决的问题:欢迎界面中修改头像底部与控制器view底部的约束，更新约束后，控制器view也跟随做运动做动画
- 已解决: 之前未解决的原因:这个项目我使用Xcode8写的，从XIB加载的view时，在viewDidLoad和ViewWillAppear方法中拿到的控制器view的frame是不准确的，所以最终在viewDidAppear或ViewWillLayoutSubViews方法中更新子控件的约束，就解决了。
```
#####请求首页数据
```
- 微博开发者网站的文档中有各种接口，这里找到【微博接口】，找到【获取当前登录用户及其关注用户的最新微博】接口
- 1.在网络工具类中封装请求【首页】数据的方法，该方法中添加闭包参数，请求完成时回调此闭包参数
- 2.在Home首页控制器中请求数据
- 3.拿到数据后，转模型，创建Statues微博模型
- 4.服务器返回的某些属性不可以直接使用，需要对数据进行处理，比如【source】属性我们需要处理的话，可以定义一个变量用来接收处理后的【scurceText】属性，再直接定义【source】为计算属性，使用didSet监听【source】属性值发生变化，只要发送变化就处理，并把处理后的结果赋值给【scurceText】
   -注意:由于【source】是可选类型，需要使用guard判断为nil时就返回，但是由于服务器肯还好返回空的字符串，比如“”,所以还要判断!+ ""的情况，guard同时判断两个条件可以使用【where】关键字将两个条件连接到一起相当于逻辑与&&
   -微博来源处理:对字符串进行处理(截串):(1)先获取要截取的起始位置和要截取的字符串的长度，(2)截取字符串，并将截取后的字符串赋值给定义的属性
   -微博创建时间的处理:
- 5.获取用户数据，创建XYUserItem用户模型类，保存用户的头像、昵称、是否是VIP、认证等级、等相关用户的个人信息
- 6.处理用户个人数据
- 7.创建视图模型，专门用于处理模型数据的，这样模型类就变得很简洁
   - 步骤:在视图模型中声明一个statues微博模型，声明所有要处理的模型属性，目的是处理完成后把结果赋值给这些属性，然后自定义构造函数，给构造函数添加一个微博模型参数，方便别人创建时给这个模型赋值的， 在构造函数中进行处理，需要注意的是构造函数中如果使用对可选类型进行解包时，不要使用guard，因为使用guard时为nil就return，就没法对下一个属性进行处理了
- 8.在控制器中直接声明一个数组(保存viewModel视图模型的)，请求完数据后，先将字典转换为微博模型，再在创建视图模型时将微博模型作为参数传进去，最后加载cell的数据时直接从视图模型的相关属性中取【这是因为[视图模型]中包含[微博模型]，[微博模型]中又包含[用户信息]的模型，所以拥有视图模型就拥有了另外的模型】
- 9.布局cell:在storyboard添加所有可以显示的控件，并添加约束，注意:微博正文的宽度约束可以先设置一个固定的，然后拖线到cell类中通过代码更改宽度约束为屏幕的宽度减去左右两边的间距，如果在storyboard中直接设置左右两边的间距，在外面计算正文的高度不准确
- 10.自定义cell类继承UITableViewCell来管理storyboard上的cell
- 11.给所有的控件拖线到cell类中，设置数据，在cell类中声明一个viewModel视图模型属性，给这个属性添加属性监听器，当属性发送改变时立即给控件设置值，跟OC中的重写模型set方法一样
- 12.布局cell底部工具栏：底部工具栏是是一个UIView，给UIView设置个灰色背景，这个工具栏添加约束，距离Cell底部、左边、右边间距为0，高度为40(根据需求)，给工具栏上添加3个UIButton，设置三个Button距离顶部为1(目的是留出灰色的线)， 宽高相同，且平分工具栏的宽度，注意:设置按钮高度约束时需要固定，让按钮的底部与cell底部留出20的距离，这样又留出了分割线效果
- 13.根据约束自动动态计算cell的高度:必须多给微博正文Label添加一个距离底部工具栏顶部的约束为10，这样最终会根据子控件确定父控件的宽度和高度
     注意:(1)要想根据约束计算cell的高度，还需要设置两个属性:       tableView.rowHeight = UITableViewAutomaticDimension（自动计算高度）
    tableView.estimatedRowHeight = 200(估算高度),(2)子控件必须设置准确的约束依赖
- 14.获取配图数据，配图数据是在服务器返回的status数组中的pic数组，因为这个pic数组保存又是字典，使用时取值麻烦，所以需要在视图模型中进行处理，在viewModel中增加一个picURLs，遍历pic数组中所有的字典，取出字典中key对应URL，然后将所有的URL添加到picURLs中
- 15.展示配图:由于配图使用类似九宫格的布局，这里可以用CollectionView展示
    -(1).在Home.storyboard中Cell上拖一个collectionView，注意:需要将【微博文本内容】控件底部距离【工具条】的约束删除掉
    -(2).给collectionView添加约束:左侧与头像对其，与微博文本内容控件有10的间距，但是宽度和高度的约束不能确定，需要根据图片确定，当没有图片时宽度和高度约束为0，但是不添加宽度和高度会报红色，可以先随便设置个宽度和高度约束，然后根据图片动态的计算图片宽度和高度约束
    -(3).对collectionView的宽度和高度约束拖线到HomeViewCell中
    -(4).在cell的viewModel的属性监听器didSet中计算，不要在awakeFromNib中计算，因为awakeFromNib只会调用一次，而collectionView的宽度和高约束需要根据【图片的个数】动态进行计算而会改变的
    -(5).计算配图尺寸方式：单独抽取一个方法来计算collectionView的Size:配图的情况可以分为【没有配图】【四张配图】【其他配图】
    -(6).这里会报很多约束相关的警告:
        -原因:【没有配图】的情况collectionView的高度约束设置为0了，本身我们在给collectionView添加了距离底部工具条顶部的约束为10，又设置了微博文本内容控件的底部约束距离collectionView顶部为0，当collectionView高度约束为0时，这两个约束就会产生冲突
        -解决方法:改变约束的优先级，可以设置collectionView距离底部工具条顶部约束的优先级小于1000即可，系统会优先使用优先级高的约束，把另外一个忽略掉
    -(7).展示collectionView上配图的数据，正常情况下应该设置collectionView的父控件为数据源的，但是由于collectionView的父控件TableViewCell类中已经做了很多其他逻辑运算和代码，这样写在TableViewCell类中会很乱，所以这里可以自定义PicCollectionView类继承自UICollectionView来管理这个配图控件
    -(8).设置PicCollectionView数据源为自己，注意:storyboard中无法设置数据源为自己的，因为苹果不建议这么做，可以通过代码来设置，由于数据源只需要设置一次，可以在awakeFromNib中设置，并遵守数据源方法
    -(9).注册PicCollectionView的cell:可以通过storyboard中点击PicCollectionView的cell，给其添加identifier重用标识即可，在PicCollectionView的cellForItemAtIndexPath数据源方法中通过这个identifier重用标识取出对应的cell
    -(10).当前PicCollectionView的cell大小不正确，需要设置collectionView的流水布局来设置itemSzie
     -步骤:(1)先将storyboard中的collectionView拖线到HomeViewCell中，在awakeFromNib中取出collectionView对应的collectionViewLayout，定义用一个常量接收，(注意:取出的布局是普通布局，并没有itemSize属性)，再强制转换为流水布局类型
          (2)直接将之前计算的每个配图的宽度和高度其实就是item的宽度高度，直接拷贝过来就行
    -(11).在PicCollectionView中定义一个picURLs数组，然后监听这个数组didSet发生改变，只要发送改变就调用PicCollectionView的reload()方法刷新数据，然后在PicCollectionView的数据源方法中直接通过picURLs数组获取数据即可
    -(12).在HomeViewCell类中给PicCollectionView中定义的picURLs数组传递数据
    -(13).自定义PicCollectionView的cell类，在这个cell类中添加imageView，让这个imageView现实配图即可，由于这个cell比较简单，所以不建议单独创建一个文件来管理这个类，可以直接在PicCollectionView类中添加这个cell类继承自UICollectionViewCell，在storyboard中让自定义的cell来管理PicCollectionView的cell，就可以在storyboard中给这个cell添加imageView了，最后设置约束为距离cell上下左右为0即可
    -(14).给cell设置数据，将PicCollectionView的cell上的imageView拖线到管理它的这个cell类中，给这个cell类定义一个模型属性picURL，监听picURL属性didSet发生改变后就给imageView设置图片，注意:需要给picURL传值，可以在PicCollectionView类中取出picURLs中每一行对应的picURL
```
#####缓存配图

######单张配图时picCollectionView的尺寸
 ```
-分两种情况:(1)服务器返回了图片的真实宽高，(2)服务器未提供图片的真实宽高
-第一种情况:服务器返回了图片的真实宽高
(1)在计算配图picCollectionView时:判断picURLs的数组元素个数只要1个的情况，就是单张配图，然后确定图片的宽度，按比例计算图片的高度，然后返回计算好的图片宽高即可，这种方法非常简单
(2)服务器未提供图片的真实宽高时:当前项目新浪服务器就未返回图片宽度和高度数据,这种情况做起来非常麻烦，需要先将图片下载缓存起来，再拿到这个图片，获取图片的宽度和高度，当公司服务器未返回这些数据时，可以直接找服务器人员沟通加上这些数据，以下详细步骤
   -(1).在Home控制器中请求数据的方法中下载并缓存图片，注意:一定要下载完图片在刷新表格
   -(2).下载图片比较复杂，单独抽取一个方法下载，给这个方法提供一个【viewModels数组】参数，遍历viewModels数组中是所有的viewModel，再遍历viewModel中picURLs，获取所有picURL(图片的URL)
   -(3).获取完图片的URL后，使用SDWebImageManager.sharedManager对象提供的downloadImageWithURL方法下载图片
   -(4).所有图片下载缓存完成后刷新表格:刷新表格必须在图片下载完成后刷新，这里需要注意的是SDWebImageManager下载图片是在异步执行的，可以使用GCD的Group队列，#1.首先在缓存图片之前创建一个【组队列】dispatch_group_create，#2.在开始下载一张图片时让下载操作进入组，#3.下载完成后离开组，所以异步操作执行完成后，#4.通知【组】并开启【主队列】刷新表格【这样就实现所有图片下载完成后再刷新表格】
   -(5).计算单张图片时picCollectionView的宽度和高度
        -1.使用SDWebImageManager通过url字符串从磁盘沙盒中取出图片(因为内存中可能会被销毁)
        -2.直接返回图片的尺寸，注意:SDWebImage内部将图片压缩的一杯，所以返回时，返回图片的width * 2，height * 2
        -3.此时存在一个问题:当单张图片时，collectionView的流水布局的itemSize会发生变化，这种情况会报警告，所以流水布局的itemSize也要动态进行设置，可以在计算picCollectionView尺寸的方法中设置，当是单张图片时，让水流布局的itemSize为图片的width * 2，height * 2，另外除单张图片的情况时，流水布局的itemSize为其他情况计算好的尺寸
 ```
 
#####转发微博
######获取转发微博的数据
```
- (1).在微博模型中添加一个转发微博属性retweeted_status,对应的类型是字典类型
- (2).由于字典用起来不方便，且转发的微博retweeted_status本身也是微博，所以可以在声明这个属性时候，声明为XYStatus微博模型类型
- (3).在自定义构造函数中，将转发微博字典转换为微博模型类
```
######展示转发微博正文
```
-(1).在storyboard中在【微博正文label】下面添加一个【转发微博正文label】，删除之前【微博正文label】底部与【配图collectionView】顶部的约束，
-(2).给【转发微博正文label】添加约束，左侧与iconView左侧对其，顶部与【微博正文label】底部直接约束为15，【转发微博正文label】需要换行设置line为0，由于【微博正文label】的宽度已经在代码中计算好了，所以这里可以直接让【转发微博正文label】的宽度等于【微博正文label】的宽度约束
-(3).为了让子控件决定父控件的高度，选中配图collectionView设置它顶部距离【转发微博正文label】底部的约束为10即可，如果此时约束报红色的话，可以修复下约束
-(4).将【转发微博正文label】控件拖线到HomeViewCell中，然后给这个控件设置数据，需要先判断【转发微博模型】的text不为nil的时候才设置数据，当然还有判断为【转发微博模型】的text为nil的时候就讲【转发微博label】的text设置为nil，防止循环利用，有if就有else
-(5).获取转发微博中screenName，在转发微博正文前面加上@用户名
- 多个条件用where分开，多个可选绑定校验用,逗号分隔
```
######展示转发微博配图
```
-(1).逻辑分析:一个微博配图与转发微博的配图只会显示其中一个，当存放微博配图的数组的元素个数count不为0的时候，就是有微博配图，此时显示微博配图，当存放微博配图的数组的元素个数count为0的时候，就没有微博配图，此时就显示转发微博的配图
使用三目运算符就可以搞定
-(2).转发微博的背景
 - 转发微博的背景可以给其添加一个UIView，让它成为独立的控件，好控制器背景的hidden属性，有转发微博时显示，没有转发微博时隐藏
 - 背景注意是用在转发微博上面的，这里把背景放在【转发微博Label】和【配图控件】下面即可
 - 给背景添加约束，底部距离工具栏为0，距离控制器view左右都为0，顶部距离微博正文为8(因为微博正文的间距为15)，所以我们让背景的线居中显示
 - 控制器背景hidden属性，拖线到HomeViewCell中，当有转发微博时显示，没有转发微博时隐藏
```
####约束细节调整
```
  建议:细节方面的调整放到最后一起调整
- 问题1:当没有配图时，微博正文与底部工具栏之间的间隙变为20，正常应该是10的，这是因为我们设置了微博正文距离配图的约束10，而配图又设置了距离工具栏的约束为10，虽然设置了配图距离工具栏的约束优先级较低，但是当没有配图的时候距离底部工具栏的约束还是存在的，所以导致最终两个约束都存在正好为20
- 解决方法:将配图距离底部工具栏的约束拖线HomeViewCell中，当没有配图时设置约束为0，当有配图时设置约束为10

- 问题2：与问题1情况相同，当没有转发微博时，配图和微博正文控件之间的间隙变为了25，正常我们希望为10，这是因为转发微博正文是一个label，他与微博正文之间的间距为15约束，而转发微博正文又与配图有10的约束间距，那没转发微博时，转发微博Label之间的这两个间距约束还是存在的，所以导致间距为25了
- 解决方法:当没有转发微博时，将转发微博正文距离顶部的约束设置为0，有转发微博的时候再设置为15
```



####计算cell的高度

######分析:目前cell的高度是根据约束自动计算出来的，而且此前设置约束出现了冲突，修改内容约束优先级为250才解决的，内容约束默认优先级为251，我们一般很少使用它，所以，我们不让子控件决定父控件的高度
```
-1.恢复内容约束为251
-2.删除工具条底部的约束，不让子控件决定父控件的高度
-3.此时已经不能通过子控件的约束来决定父控件的高度了，就要删除tableView的自动计算高度的代码
```
######计算cell的高度
```
-1.实现tableView的代理方法heightForRowAtIndexPath
-2.设置tableView的估算高度:这样会先调用cellForRowAtIndexPath再调用heightForRowAtIndexPath方法
-3.在给cell传递视图模型的时候就要确定cell的高度，所以计算cell的高度要在HomeViewCell的视图模型属性的监听器中设置
-4.计算cell高度的步骤:
	-(1).先调用layoutIfNeed()方法进行强制布局
	-(2).拿到storyboard中工具栏的最大Y值，将工具栏拖线到HomeViewCell中即可拿到，因为工具栏的最大Y值就是cell的高度
	-(3).在viewModel模型视图中定义一个cellHeight变量，将计算好的cell的高度直接复制给它保存起来即可
	-(4).由于HomeViewCell的视图模型属性添加了属性监听器，每次有新的cell出现时都会调用，也就会重复计算cell的高度，这样不好，所以可以在计算cell高度时先判断定义在viewModel中的cellHeight变量是否为0，如果为0再进行强制布局，再计算cell的高度
	-(5)在tableView的代理方法heightForRowAtIndexPath中，先通过viewModels数组获取viewModel模型，再返回viewModel中的cellHeight即可
下拉刷新

- 苹果提供了UIRefreshController刷新控件，其继承自UITableViewController
- 苹果的Xcode在编译时会自动把OC代码转换为Swift代码
```
#####当前项目为了方便使用MJRefresh即可解决
```
-1.给HomeViewController添加下拉刷新控件，使用MJRefreshNormalHeader创建Header
-2.设置header各种状态下的文字
-3.让Header加载最新的数据，在当前类中定义一个加载最新数据的方法
-4.设置tableView的mj_header为创建的header
-5.开始刷新，有下拉刷新以后就没必要主动调用加载新数据的方法了
-6.让header直接调用HomeViewController中加载数据的方法
-7.请求完成数据后结束刷新
-当前存在的问题：每次下拉刷新时并没有请求到最新的数据，这是因为我们之前请求数据传递的参数不够，我们只传了access_token，只会请求最新的20条微博，但是我们并不想要最新的微博，而是要比当前界面上第一条微博新的微博
- 分析:每天微博都有一个ID，比较新的微博ID肯定比现在的微博要大
-解决方法:需要再传递一个参数since_id，这个参数会返回比当前时间更新，微博id更大的微博，比如我们传了一个since_id为2000001，那么服务器返回的微博的ID肯定要比这个ID要大的
  -(1)请求数据前要获取since_id，让其默认为0，如果加载新数据就是下拉刷新时，since_id等于viewModels数组中第一个viewModel的微博模型的mid属性，如果第一个viewModel的微博模型的mid属性取不到就让since_id为0
  -(2)在网络工具类封装的请求数据方法上增加一个Int类型since_id参数，然后在拼接参数时将其拼接进去，并在拼接参数时将since_id包装为字符串类型
  -(3)在HomeViewController中请求网络就使用这个新的方法请求
  -(4)当下拉刷新数据时，将新的数据应该放在viewModels数组的最前面，而不是直接添加到后面，所以不能使用append方法，这样会导致数据有误，但是如果使用inset方法将新数据插入到模型数组的前面也存在问题:比如当我们一次请求2条数据回来时，会取出最新的一条先inset插入到模型数组的前面，到第二条数据在插入时，就会插入到最新的那条数据的前面了，这样就导致数据顺序错误，因为这2条数据第1条先插入的最新，但是第2条数据却插入在最上面了
  -(5)插入数据最好的解决方法:(将要插入的数据放在数组中，将这个数组直接拼接在原来的数组的前面)1.在遍历服务器返回的数据之前，先定义一个临时数组变量，2.每次遍历服务器返回的新数据时，转换为模型后，将模型先存放临时数组tempViewModels中，3.将临时数组tempViewModels直接拼接在大数组viewModels的前面，swift中可以这样self. viewModels = tempViewModels + viewModels
  -(6)注意:在缓存图片时只缓存最新的图片即可，不用缓存大数组中最新的数据
上拉加载更多

-1.给tableView添加一个mj_footer，使用MJRefreshAutoFooter 创建一个下拉刷新控件，MJRefreshAutoFooter创建的footer没有上拉加载更多的文字
-2.添加一个方法loadMoreStatus，这个方法调用请求数据的方法即可，给这个方法传false，isNewData属性就会为false，就不会加载最新的数据，就会去加载更多
-3.请求更多数据还需要传max_id参数，若传此参数服务器返回的的微博ID会小于当前显示的最后一条微博ID，根据是否是最新数据来判断
-4.在网络工具类中在请求数据方法中，再添加一个参数max_id,然后把这个参数拼接到请求参数中
-5.此时要对转换完成的模型添加数组进行处理，根据isNewData来判断，如果是新数据就添加到大数组的前面，旧数据就添加到后面
添加下拉刷新时提示更新微博数据提示的Label

-1.在HomeViewController中定义一个懒加载的提示UILabel变量
-2.在控制器viewDidLoad方法中将这个Label添加到导航控制器的View上，并设置frame，最好不要添加到tableView上，因为拖动tableView时，这个Lable也会跟着tableView一起滚动，当前需求不需要Label跟着tableView一起滚动
 注意:再添加提示Label到导航控制器上时，不要使用addSubviews方法，使用insertSubviews方法传入到导航控制器上就是1，此时如果看到看到导航栏的颜色和状态栏的颜色都有提示Lable颜色模糊的效果时，这是因为默认情况下导航栏有毛玻璃效果(高斯模糊)，他会将后面的模糊化，那么后面的颜色就会扩散开
-3.设置提示Lable的frame，并设置默认情况下就隐藏，在刷新表格后就显示提示Lable，并设置动画，完成动画后隐藏
```
