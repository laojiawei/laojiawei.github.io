title: Block
date: 2015-12-03 00:00:00
---
#Block
##block写法
+ <!-- more -->

```
block的写法:
    类型:
    返回值(^block的名称)(block的参数)

    值:
    ^(参数列表) {
        // 执行的代码
    };

```

##block基本使用
* 作用就是用来保存一段代码块,在一个方法定义,在另外一个方法去调用(用的少,可以用定义方法替代,一般在 在一个类中定义,在另外一个类中去调用 用的比较多
###定义的几种方式


``` bash
#方式一: 等号 = ^(参数){ block代码块 }
 void(^block1)(int) = ^(int a){
        NSLog(@"调用了block%d",a);
    };
```
* 注意点:如果block带有参数,在定义的时候,一定要有参数,并且变量名不能省略


```
# 方式二: 如果没有参数,参数可以省略
    void(^block2)() = ^{
       
    };
```

```
# 方式三: 等会右边 = ^返回值(参数),返回值可以省略,不管有没有返回值,都可以省略
    int(^block3)() = ^int{
        return 1;
    };

```
* 注意:如果Block有返回值,block代码块必须要有返回值

对于初学者,block的语法格式可能比较难记,可以尝试先用快捷的方式敲出格式,再根据生成的格式填入
	`block快捷方式:inline`

```
<#returnType#>(^<#blockName#>)(<#parameterTypes#>) = ^(<#parameters#>) 
{
        <#statements#>
};
```	


###block注意事项
* 在block内部可以访问block外部的变量

```
int a = 10;
 void (^myBlock)() = ^{
  NSLog(@"a = %i", a);
   } myBlock();
    输出结果: 10
```
* block内部也可以定义和block外部的同名的变量(局部变量),此时局部变量会暂时屏蔽外部

```
int a = 10;
 void (^myBlock)() = ^{
  int a = 50;
   NSLog(@"a = %i", a);
    } myBlock();
     输出结果: 50
```

* 默认情况下, Block内部不能修改外面的局部变量

```
int b = 5;
 void (^myBlock)() = ^{
  b = 20;
// 报错 NSLog(@"b = %i", b);
    };
     myBlock();
```

* Block内部可以修改使用__block修饰的局部变量

```
__block int b = 5;
void (^myBlock)() = ^{
  b = 20;
  NSLog(@"b = %i", b);
  };
myBlock();
输出结果: 20
```

* block中可以访问外面的变量

* block中可以定义和外界同名的变量, 并且如果在block中定义了和外界同名的变量, 在block中访问的是block中的变量 
* 默认情况下, 不可以在block中修改外界变量的值,因为block中的变量和外界的变量并不是同一个变量
* 如果block中访问到了外界的变量, block会将外界的变量拷贝一份到堆内存中,因为block中使用的外界变量是copy的, 所以在调用之前修改外界变量的值, 不会影响到block中copy的值

* 如果想在block中修改外界变量的值, 必须在外界变量前面加上__block ,那么会影响到外界变量的值

* 如果没有添加__block是值传递
* 如果加上__block之后就是地址传递, 所以可以在block中修改外界变量的值 


#### 下面我们通过一个小案例来演示block的基本使用场景
### block保存代码:
案例:点击cell,作出相应的动作,在cell里面利用block保存代码
* 1.在cell模型声明block属性

```
@property (nonatomic, strong) void(^block)();
```
* 2.在模型里面保存代码

```    
    // 打电话
    CellItem *item = [CellItem cellItem];
    item.title = @"打电话";
    // 定义block给它
    item.block = ^{
        NSLog(@"打电话");
    };
```
* 3.点击cell调用block

```
- (void)tableView:(UITableView *)tableView didSelectRowAtIndexPath:(NSIndexPath *)indexPath
{
    CellItem *item = self.items[indexPath.row];
   
    if (item.block) {
        item.block();
    }
}
```
### block传值
开发中,block传值大致分为顺传和逆传
> 顺传:比如A控制器把值传到Push之后的B控制器,就是顺传(定义属性)

* 很简单,你一般只需要在在B控制器的头文件声明一个属性,A控制器拿到B控制器之后,把属性赋值,B控制器就可以拿到A传过来的值使用.

>逆传:从名字就可以看出,是跟顺传反过来的,就是由B控制器反过来传回去A控制器,那么,这就没那么简单了

* 逆传我们可以使用代理,但是开发中一般不会使用了,因为实在是比较麻烦,而且代码量很多,我们一般会使用block来代替代理,从而精简代码,看起来也比较容易理解

先总结一下:`传值万能步骤:A传值B,A拿到B 就能传值  B传值A,拿到B就能传值`

####利用block逆传
* 1.在传递方头文件声明block属性

```
@property(nonatomic,strong)void(^block)(NSString*value);
```
* 2.在.m文件触发事件传值(参数)
判断一下是否有block

```
- (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event
{
    // 传值:
    if (_block) {
       _block(@"123");
    }
```
* 3.接收方拿到传递方,拿到block接收值

```
- (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event
{
    ModalViewController *modalVc = [[ModalViewController alloc] init];
   
    modalVc.block = ^(NSString *value){
        NSLog(@"利用了block,接收到值%@",value);
    };
```

```
#好,上面简单介绍了block得传值方式,基本上开发的简单传值就是这么用的,一些复制的传值也是万变不离其宗
```
##block内存管理
写在前面:为什么要理解或者掌握内存?!
#######有一位大牛说过,你要灵活运用一个东西,就必须要知道它的内存是怎样运作,是怎么管理的,才能做到灵活运用到实际开发中

>我们都知道,说到内存管理,可以分为ARC和MRC两种情况
先来补充一下知识:
如何判断当前项目是ARC,还是非ARC
    1.在ARC中,不允许调用retain,release,retainCount
    2.重写dealloc,ARC中不能调用[super dealloc]
   
   怎么进入非ARC环境 : 点击工程文件 -> buildSetting -> ARC
  

####1.ARC的block内存管理
ARC下,默认一个局部变量就是一个强指针,防止一创建就释放
block访问了一个局部变量,就会放到堆里面

 	* 只要访问了一个外部变量,,生命周期不是全局的,只会放在堆里面
   * 只要访问了一个外部变量,,生命周期是全局的,只会放在全局区

只要在block代码块里面使用了self强引用就会导致循环引用



```
# 注意:只要在block的代码中,访问外部强指针对象,就会把对象强引用

- (void)viewDidLoad {
    [super viewDidLoad];
    int a = 2;
   
#注意:只要在block的代码中,访问外部强指针对象,就会把对象强引用
#解决循环引用:将该对象使用weak或者block修饰符修饰之后再在block中使用/或者将其中一方强制制空 xxx = nil 。

    __weak typeof(self) weakSelf = self;
   
    _block = ^{
      
        // 定义强指针的self
        __strong typeof(self) strongSelf = weakSelf;
       
        dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(2 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
           
            NSLog(@"%@",strongSelf);
           
        });
       
       
        NSLog(@"%d",a);
       
    };
   
    _block(); 
}
```
_*从上面我们需要知道的是,什么是值传递,什么是指针传递*_
* 值传递: 访问局部变量 
* 指针传递:访问生命周期全局变量 
* 而被__block修饰的局部变量,就是指针传递

* 那么问题来了block是存储在堆中还是栈中?

   //默认情况下block存储在栈中,如果对block进行一个copy操作, block会转移到堆中
   //如果block在栈中, block中访问了外界的对象,那么不会对对象进行retain操作
   //但是如果block在堆中, block中访问了外界的对象,那么会对外界的对象进行一次retain
   
   //如果在block中访问了外界的对象,一定要给对象加上__block,只要加上了__block,哪怕block在堆中,也不会对外界的对象进行retain
   //如果是在ARC开发中就需要在前面加上__weak

好,知道了上面的知识,你就会明白这一段代码的意思,为什么要加上这一句`__strong typeof(self) strongSelf = weakSelf;`
我们的目的是:在要执行完那个延迟block,而且拿到对象`strongSelf`做完我们需要做的事情之后,modal出来的控制器才销毁
如果没有一个强引用(上面那一行代码),那么,当控制器dismiss之后就释放掉那我们就拿不到`weakSelf`这个对象做事情了.
###2.非ARC(MRC)的block内存管理

只要block访问了外部变量,生命周期不是全局的(是否在代码块里面),就存放在栈里面 
生命周期是全局的,block就存放在全局区 


block在非ARC中必须要使用copy  

在非ARC环境下,retain相当于strong,为什么block不用retain 
因为在非arc中,是不会存放到堆里面,过了代码块就会被释放.再访问就会坏内存访问报错 

在非ARC开发注意点:访问属性,一定要使用get,set方法,不能直接使用下划线



####bock开发场景:参数使用 
* 1.封装一个类的时候,有些时候,怎么去做由外界决定,但是由内部决定什么时候调用,把block当做一个参数去使用. 

* 2.动画block:做什么样的动画由我们决定,但是什么时候调用由系统决定. 


####bock开发场景:作为返回值使用
这里就涉及到链式编程思想了,所谓的链式编程思想: 把所有的方法调用全部通过.语法连接起来,好处:可读性非常好

```
mgr.add(5).add(5); 
其实是分两步走,先调用mgr.add的getter,返回一个block.再跟着()实现这个block 

- (CalculatorManager *(^)(int))add; 

- (CalculatorManager *(^)(int))add
{
    return ^(int value){
       
        _result += value;
       
        return self;
    }; 
}

```

##block在实际开发的应用举个例
* 定义网络请求的类

```
@interface HttpTool : NSObject
- (void)loadRequest:(void (^)())callBackBlock;
@end

@implementation HttpTool
- (void)loadRequest:(void (^)())callBackBlock
{
    dispatch_async(dispatch_get_global_queue(0, 0), ^{
        NSLog(@"加载网络数据:%@", [NSThread currentThread]);

        dispatch_async(dispatch_get_main_queue(), ^{
            callBackBlock();
        });
    });
}
@end
```

* 进行网络请求,请求到数据后利用block进行回调

```
- (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event
{
    [self.httpTool loadRequest:^{
        NSLog(@"主线程中,将数据回调.%@", [NSThread currentThread]);
    }];
}
``` 

###一些事:
* 用strong还是copy的问题?
block 使用 copy 是从 MRC 遗留下来的“传统”,在 MRC 中,方法内部的 block 是在栈区的,使用 copy 可以把它放到堆区.
★建议:在ARC中,能用strong就用strong,因为copy内部会做很多事情

* block是对象吗?
是的!苹果告诉我的\^0\^!文档说的很清楚的


```
写在最后,block之强大可谓开发利器
								------make by LJW 转载请注明出处-------
```






