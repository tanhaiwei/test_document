# iOS SDK 接入

## 1.安装SDK

最新版DatatistTrackers使用ARC并支持IOS8+和OSC10.8+

datatist支持3中安装方式

### 安装方式A：使用cocoapods安装

Podfile 文件添加：

```
platform :ios,'8.0'
target ‘项目名’ do
pod 'DatatistTracker'
end
```

执行 `pod update`或

```
pod repo update –verbose
 pod install
```

### 安装方式B：手动安装

点击  [下载配置文件](#)

解压下载的DatatistTracker.zip文件，将解压后的include文件夹, lib文件夹下的libDatatistTracker.a文件，copy到需要追踪的APP工程的根目录下。也可以根据需要自行指定放置位置。

在需要追踪APP target的Build Setting下，搜索other，将其other linker flags 改成-ObjC。![](/assets/iOS_1.png)

在需要追踪APP target的Build Setting下，搜索header search paths，将其参数指定为之前include文件夹对应放置的路径。如果是放置在根目录下，则指定为${SOURCE\_ROOT}/include路径。如果是放置到其他位置，则可以根据实际放置的路径设定参数。![](/assets/iOS_2.png)

转到Build Phases设置下,在Link Binary With Libraries中添加libDatatistTracker.a。当前的libDatatistTracker.a 已经通过lipo集成了simulator环境和iphoneos环境，可以在开发和发布环境下工作。![](/assets/iOS_3.png)

在每个需要引入数据采集的.m 文件中引入如下5个.h文件

```
#import "DatatistTransactionBuilder.h"
#import “DatatistDebugDispatcher.h”
#import “DatatistDispatcher.h”
#import “DatatistCouponInfo.h”
#import “DatatistOrderInfo.h”
#import “DatatistProductInfo.h”
#import “DatatistTransactionItem.h”
```

调用相应的API完成数据的采集工作

### 安装方式C：在swift中安装

Podfile 文件添加：

```
platform :ios,'8.0'
use_frameworks!
target ‘项目名’ do
pod 'DatatistTracker'
end
```

执行 `pod update`或

```
pod repo update –verbose
 pod install
```

在project 代码根目录中创建DatatistTracker-Bridging-Header.h文件，添加如下.h：

```
#ifndef DatatistTracker_Bridging_Header_h
#define DatatistTracker_Bridging_Header_h

#import <DatatistTracker/DatatistTracker.h>
#import <DatatistTracker/DatatistTransaction.h>
#import <DatatistTracker/DatatistTransactionBuilder.h>
#import <DatatistTracker/DatatistDebugDispatcher.h>
#import <DatatistTracker/DatatistDispatcher.h>
#import <DatatistTracker/DatatistCouponInfo.h>
#import <DatatistTracker/DatatistOrderInfo.h>
#import <DatatistTracker/DatatistProductInfo.h>
#import <DatatistTracker/DatatistTransactionItem.h>

#endif /* DatatistTracker_Bridging_Header_h */
```

转到Build Settings，在Objectiv-C Bridging Header 中 设置Bridging Header的路径：![](/assets/iOS_4.png)

至此，即可在swift中调用DatatistTracker的对象。例如：![](/assets/iOS_5.png)

## 2.初始化SDK

Appdelegate.m中的didFinishLaunchingWithOptions方法中调用来实现tracker的初始化。

```
static NSString * const DatatistProductionServerURL = @"https://xxxxxxx";
static NSString * const DatatistProductionSiteID = @"xxxxxxxxxx";
[DatatistTracker initWithSiteID:DatatistProductionSiteID baseURL:[NSURL URLWithString:DatatistProductionServerURL]];
```

showLog设置，是否打印log，用于调试。

```
[DatatistTracker sharedInstance].showLog = YES
```

## 3.基础配置

##### userID：

使用userID可以帮助Tracker收集同用户在不同设备和浏览器上的信息，以长期定位追踪用户信息以及支持跨设备和浏览器行为的关联。userID是一个非空的字符串，比如用户名，邮箱地址，手机号 等唯一识别此用户。userID在不同设备和浏览器上必须是相同的。可以将userID进行加密后进行传输。

示例：

```
[DatatistTracker sharedInstance].userID = @"your_userID";
```

建议设置userID的位置如下：

·在初始化Tracker后即设置已注册客户保存的userID

·在注册流程结束后设置新注册用户的userID

·在登录流程结束后设置已有用户登录的userID

##### 页面采集：

进入某页面时采集所需数据，用于分析页面来源，统计浏览的页面路径，名称，访问时长，所使用的浏览器等信息

```
- (void)trackPageView:(NSArray *)views title:(NSString *)title udVariable:(NSDictionary *)vars;
```

API说明： 获取每个screen的 view名称和层级路径，并且可以添加自定义参数，建议在每个view controller中调用。

参数说明：

title：页面标题

screen：一个数组，元素是view名称。

udVariable: 用户自定义参数，可以为nil

样例程序：

```
// 在 view controllers里检测screen views
- (void)viewDidAppear:(BOOL)animated {
// 推荐抓取完整的screen的路径结构。举例，对于screen/view1/view2/currentView这个screen
[[DatatistTracker sharedInstance] trackPageView:@[@"view1",@"view2"] title:@"test_title" udVariable:nil];
}
```

建议设置页面采集的位置如下：

建议将页面采集接口放在根控制器中实现

## 4.事件配置

### 4.1搜索采集

说明：点击搜索按钮时，采集搜索的关键词及关键词来源，如热门推荐词或历史记录，可以用map添加自定义变量 。

##### 接口声明：

```
- (void)trackSearch:(NSString *)keyword recommendationSearchFlag:(BOOL)recommendationFlag historySearchFlag:(BOOL)historyFlag udVariable:(NSDictionary *)vars;
```

API说明：获取搜索栏的关键词和结果信息 。

##### 参数说明：

keyword：搜索的关键词

recommendationFlag: 常用词搜索

historySearchFlag：历史搜索

udVariable:    用户自定义参数，可以为nil

样例程序：

```
// Measure the most popular keywords used for different search  operations in the app
[[DatatistTracker sharedInstance] trackSearch:@"zpf_test" recommendationSearchFlag:false historySearchFlag:false udVariable:@{@"search": @"searchBar"}];
```

### 4.2用户注册

说明：用户注册成功的回调中，先调用setUserId接口，再调用用户注册接口，采集注册信息，可以用map添加自定义变量 。

##### 接口声明：

```
- (void)trackRegister:(NSString *)uid type:(NSString *)type authenticated:(BOOL)auth udVariable:(NSDictionary *)vars;
```

API说明：采集用户注册事件。

##### 参数说明：

uid: 用户注册的用户ID；

type: 用户类型；

authenticated: 是否已认证的标识；

udVariable: 客户可扩展的自定义变量，以NSDictionary对象的形式进行存储；

样例程序：

```
[[DatatistTracker sharedInstance] trackRegister:@"1234567890" type:@"register" authenticated:true udVariable:nil];
```

### 4.3登录

说明：用户登录成功的回调中，先调用setUserId接口，再调用用户登录接口，采集登录信息，可以用map添加自定义变量 。

##### 接口声明：

```
- (void)trackLogin:(NSString *)uid udVariable:(NSDictionary *)vars;
```

API说明：采集用户登录事件 。

##### 参数说明：

uid: 用户ID

udVariable: 客户可扩展的自定义变量，以NSDictionary对象的形式进行传输

样例程序：

```
[[DatatistTracker sharedInstance] trackLogin:@"your_userid" udVariable:@{@"loginTime": @((long long)[[NSDate date] timeIntervalSince1970] * 1000)}];
```





