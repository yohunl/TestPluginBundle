# TestPluginBundle
本例子是用来展示通过 创建一个bundle工程来创建xcplugin插件工程的演示demo
创建的时候是基于xcode 7.2.1版本


## 概述
我们平时也使用了很多的xcode插件,虽然官方对于插件制作没有提供任何支持,但是加载三方的插件,默认还是被允许的.第三方的插件,需要存放在 ~/Library/Application Support/Developer/Shared/Xcode/Plug-ins文件夹中,后缀名必须是.xcplugin (不过其实际上是一种bundle).
所以我们创建一个插件工程,直接创建bundle工程即可,然后通过修改后缀名为.xcplugin,将其放到~/Library/Application Support/Developer/Shared/Xcode/Plug-ins目录中就可以了

第一个demo插件功能:在xcode的edit菜单中加入一个叫做 测试菜单 的项目,当点击的时候,弹出一个警告框,显示一句话,完整的工程放在[TestPluginBundle](https://github.com/yohunl/TestPluginBundle)

<!--more-->

## 详细过程
创建Bundle工程 TestPluginBundle
![](http://7xqspl.com1.z0.glb.clouddn.com/pluginDemo1.png)

工程名称就是  TestPluginBundle

### 工程设置
插件工程和普通的bundle工程还是有区别的,所以需要进行特殊的设置
#### 首先是工程的plist文件
![](http://7xqspl.com1.z0.glb.clouddn.com/pluginDemo2.png)
添加 三项
XCPluginHasUI = NO
XC4Compatible = YES
DVTPlugInCompatibilityUUIDs    这是一个数组.数组内容字符串,指示了该插件兼容的xcode版本,只有对应版本的xcode的UIID加入这个数组了,插件才能被加载,否则,即使你将插件放入xcode的插件文件夹,插件也不会被加载的
那么怎么获取你当前版本的xcode的UUID呢?在terminal中输入 defaults read /Applications/Xcode.app/Contents/Info DVTPlugInCompatibilityUUID ，terminal会返回一串字符串给你，这就是你的Xcode的DVTPlugInCompatibilityUUID.

#### 接下来是 Build Setting了
![](http://7xqspl.com1.z0.glb.clouddn.com/pluginDemo3.png)
![](http://7xqspl.com1.z0.glb.clouddn.com/pluginDemo4.png)
Installation Build Products Location 设置为 ${HOME}  [显示的时候,显示的是你的用户目录],这个是products的根目录

Installation Directory 设置为 /Library/Application Support/Developer/Shared/Xcode/Plug-ins,这个是指定你的插件安装的目录. 注意,这里填入的其实是相对目录,插件的绝对目录是这样的,例如  /Users/yohunl/Library/Application\ Support/Developer/Shared/Xcode/Plug-ins/Alcatraz.xcplugin ,最后的绝对目录是  Installation Build Products Location和Installation Directory的结合,这也是为什么两者都要设置的原因

Deployment Location 设置为 YES,这个是指示该工程不使用设置里的build location，而是用Installation Directory来确定build后放哪儿
![](http://7xqspl.com1.z0.glb.clouddn.com/pluginDemo5.png)
我们默认工程生成的相关文件放在哪.都是 Build Locations指示的,通过Deployment Location 设置为 YES告诉工程,我们不使用这个默认的设置,而是我们自定义的

Wrapper extension 设置为 xcplugin,后缀名必须为xcplugin,否则不会被加载

#### 接下来就是插件的实现过程了
在工程中添加一个文件 ,名称为  TestPluginBundle (当然,名字随便什么都可以),在其中添加代码
```objc
@implementation TestPluginBundle
+(void)pluginDidLoad:(NSBundle *)plugin {
    NSLog(@"插件运行了!");
    [TestPluginBundle sharedInstance];
}

- (instancetype)init{
    self = [super init];
    if (self) {
        [[NSNotificationCenter defaultCenter] addObserver:self
                                                 selector:@selector(didApplicationFinishLaunchingNotification:)
                                                     name:NSApplicationDidFinishLaunchingNotification
                                                   object:nil];
    }
    return  self;
}

- (void)didApplicationFinishLaunchingNotification:(NSNotification*)noti
{
    [[NSNotificationCenter defaultCenter] removeObserver:self name:NSApplicationDidFinishLaunchingNotification object:nil];

    NSMenuItem *menuItem = [[NSApp mainMenu] itemWithTitle:@"Edit"];
    if (menuItem) {
        [[menuItem submenu] addItem:[NSMenuItem separatorItem]];
        NSMenuItem *actionMenuItem = [[NSMenuItem alloc] initWithTitle:@"测试菜单" action:@selector(doMenuAction) keyEquivalent:@""];
        [actionMenuItem setTarget:self];
        [[menuItem submenu] addItem:actionMenuItem];
    }
}

- (void)doMenuAction
{
    NSAlert *alert = [[NSAlert alloc] init];
    [alert setMessageText:@"测试菜单运行"];
    [alert runModal];
}

- (void)dealloc
{
    [[NSNotificationCenter defaultCenter] removeObserver:self];
}

+ (instancetype)sharedInstance
{
    static id instance;
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        instance = [[self alloc] init];

    });
    return instance;
}
@end
```
ctrl+B来Build工程,查看路径下/Library/Application Support/Developer/Shared/Xcode/Plug-ins,可以看到我们的插件TestPluginBundle.xcplugin存在了,接下来,重启xcode
![](http://7xqspl.com1.z0.glb.clouddn.com/pluginDemo6.png)
点击 测试菜单
![](http://7xqspl.com1.z0.glb.clouddn.com/pluginDemo7.png)
可能你 会说,这样虽然是起作用了,但是,难道开发一个插件工程,没打单步调试么???,当然不是啊
编辑工程的scheme,将Executable设置为Xcode.app,意思是工程调试的时候挂载到xcode中
![](http://7xqspl.com1.z0.glb.clouddn.com/pluginDemo8.png)
将Options下面的Core Location,XPC Services,View Debugging前面的勾都去掉,否则,你调试的时候,可能会直接crash
![](http://7xqspl.com1.z0.glb.clouddn.com/pluginDemo9.png)
当设置完后,你的工程的scheme的图标会从bundle图标变为xcode的图标
![](http://7xqspl.com1.z0.glb.clouddn.com/pluginDemo10.png)
再运行(这里是运行了,不是编译了)
不出意外的话,会出现xode启动另外一个xcode,接下来和你普通的调试工程就是一样的了!

说了这么多,其实只是想让你明白一个插件的初始化的配置,调试等

上面的过程,已经有国外大神制作成了一个 工程模板了,https://github.com/kattrali/Xcode-Plugin-Template  其支持OC和Swift,当你安装它后,会在新建工程时候,看到 Xcode Plugin模板,使用这个模板创建一个新工程,以上的配置等,就都设置好了,直接运行就是一个demo了.


![](http://7xqspl.com1.z0.glb.clouddn.com/pluginDemo11.png)

### 参考文章
1. [喵神的xcode4插件制作入门](http://onevcat.com/2013/02/xcode-plugin/),文章中部分内容已经过期,不再适用,但主体是通用的,并且文章后面的后记也非常实用的干货
2. [ Xcode 6 插件开发入门：添加自己的想法和功能](http://www.cocoachina.com/ios/20150506/11765.html)
