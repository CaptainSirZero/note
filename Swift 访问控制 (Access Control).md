### Swift 访问控制 (Access Control)
[文章链接](https://medium.com/@jerrywang0420/access-control-教學-swift-3-ios-4d93ee567eb0)

>在Swift中有范文控制,目的就是为了保护类的私有属性和方法,不暴露给外部,只提供需要给外部展示的方法API(Application Programming Interface, 应用程序界面),供外部使用这个class所提供的功能
>
>在Access Control 里面有两个概念 module 和 source file
>
>Module: 
>>一个module可代表一个bundle ID下的 App, 一个framework.因此在一个App中, import了很多个framework, 每一个framework内即一个module,frame以外的世界,App其实也可以理解为一个module
>
>Source file:
>>就是每一个.swift文件,例如下图的四个source file
>>
>>>![](/Users/tingbo/Documents/随手笔记/Sourcefile.png)
>
>访问控制,影响了module和module之间, module和Source file之间, Source file和Source file之间
>
>swift中访问控制可分为5个层级,由层级最高到层级低依次为
>
>**open  -->  public  -->  internal --> fileprivate -->private**
>
>![Swift访问控制层级](/Users/tingbo/Documents/随手笔记/Swift访问控制层级.png)
>
>**open (开放,可重写,可集成)**
>> 用于声明class以及class内所有的成员,层级最高,所有module和Source file 都可以获取到,且可以subclass和override这个被open修饰的class
> 
>**public (开放,不可重写,不可集成)**
>> 用于声明class以及class内所有的成员,所有module和Source file 都可以获取到,但是如果是修饰class,则不能够subclass和override 被public修饰的class,相当于只能readonly
>
>**internal (module内部使用)**
>> 在声明的module内所有的Source file文件都可以获取到,module以外无法获取得到.module内所有Source file ,如果是class, 则可以 subclass 和 override 这个class.
>
>**fileprivate(只能在相应的swift文件中使用)**
>> 声明的Source file内可取得,若为class,可subclass和override
>> 可以用来修饰属性,然后再extension中使用
>
>**private(只有在自己的类中可以使用,在extension中都不可以)**
>> 在类代码范围内才可以使用,也就是{}内,若为class,可subclass和override
>
> private 与 fileprivate 对比如下


```
class ViewController: UIViewController {

    fileprivate var property_file : String = ""
    private var property_private : String = ""
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        property_file = "hello~"
        property_private = "hello~~"
        
        // Do any additional setup after loading the view, typically from a nib.
    }
}

extension ViewController{
    func someFunction(){
        property_file = "hi~~"
        //無法取得property_private
    }
}
```
> 使用注意: 
>>内部成员的层级不能高于最外围的成员

```
fileprivate class someClass {
	fileprivate let name : String = "json"
	private let age : Int = 14
	open let height : Int = 180 //这个open关键字层级高于fileprivate,会报错的

}
```




