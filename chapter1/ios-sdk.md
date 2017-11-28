# iOS SDK 接入

## 1.安装SDK

最新版DatatistTrackers使用ARC并支持IOS8+和OSC10.8+

datatist iOS SDK 支持3种安装方式

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

## 4.基础事件

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

说明：用户注册成功的回调中，先调用setUserId接口，再调用用户注册接口，采集注册信息，可以用map添加自定义变量 。

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

说明：用户登录成功的回调中，先调用setUserId接口，再调用用户登录接口，采集登录信息，可以用map添加自定义变量 。

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

### 4.4产品页访问

说明：进入商品详情页后调用，采集商品信息，可以用map添加自定义变量 。

##### 接口声明：

```
- (void)trackProductPage:(NSString *)sku productCategory1:(NSString *)category1 productCategory2:(NSString *)category2 productCategory3:(NSString *)category3 productOriginPrice: (double)originPrice productRealPrice:(double)realPrice udVariable:(NSDictionary *)vars;
```

API说明：采集产品也信息。

##### 参数说明：

sku: 页面产品的SKU信息；

category1: 一级目录；

category2: 二级目录，没有值就留空；

category3: 三级目录，没有值就留空；

originalPrice: 商品原价

realPrice: 商品优惠价

udVariable: 客户可扩展的自定义变量，以NSDictionary对象的形式进行传输；

样例程序：

```
// 产品页访问
[[DatatistTracker sharedInstance] trackProductPage:@"4567934501204569" productCategory1:@"电子产品" productCategory2:@"手机" productCategory3:@"安卓手机" productOriginPrice:3999.0 productRealPrice:3000.0 udVariable:@{@"event": @"productPage"}];
```

### 4.5加入购物车

说明：点击加入购物车按钮时调用，采集加入购物车的商品信息，可以用map添加自定义变量 。

##### 接口声明：

```
- (void)trackAddCart:(NSString *)sku productQuantity:(long)quantity productRealPrice:(double)realPrice udVariable:(NSDictionary *)vars;
```

API说明：采集商品加入购物车事件。

##### 参数说明：

sku: 被加入购物车的SKU商品信息；

quantity: 被加入购物车的SKU商品数量；

realPrice: 被加入购物车的商品单价

value：integer或者float的值，对name做附加说明

udVariable: 客户可扩展的自定义变量，以NSDictionary对象的形式进行传输

样例程序：

```
//购物车事件追踪
[[DatatistTracker sharedInstance] trackAddCart:@"4567934501204569" productQuantity:3 productRealPrice:3000.0 udVariable:nil];
```

### 4.6生成订单

说明：提交订单时调用，设置订单中的必要参数，采集订单信息，可以用map添加自定义变量 。

##### 接口声明：

```
- (void)trackOrder:(DatatistOrderInfo *)order couponInfo:(NSArray *)coupons productInfo:(NSArray *)products udVariable:(NSDictionary *)vars;
```

API说明：采集创建订单事件。

##### 参数说明：

DatatistOrderInfo: 订单信息，其中包含项5项具体的参数：

orderID: 订单号

orderAMT: 订单总价

shipAMT: 运费总价

shipAddress: 收货地址

shipMethod: 配送方式

couponInfo为DatatistCouponInfo数组， DatatistCouponInfo: 优惠券信息，其中包含2项具体的参数：

couponType: 优惠券类型

couponAMT: 优惠券金额

productSKU: 产品SKU

productTitle: 产品名称

productRealPrice: 产品实际成交价

productOriPrice: 产品原价

productSourceSku：活动商品来源（例如赠品产品。传原商品sku，标识原商品的绑定关系）

udVariable: 客户可扩展的自定义变量，以NSDictionary对象的形式进行传输。

样例程序：

```
DatatistOrderInfo \*orderInfo = \[DatatistOrderInfo new\]; orderInfo.orderID = @"2017101716591100"; 
orderInfo.orderAMT = 9020.0; orderInfo.shipMethod = @"顺丰快递"; 
orderInfo.shipAddress = @"上海市徐汇区宜山路333号 s汇鑫国际1号楼603\#"; 
orderInfo.shipAMT = 20.0; 
DatatistCouponInfo \*couponInfo = \[DatatistCouponInfo new\]; 
couponInfo.couponType = @"红包%";
couponInfo.couponAMT = 999.002;

DatatistCouponInfo *couponInfo2 = [DatatistCouponInfo new];
couponInfo2.couponType = @"优惠券%";
couponInfo2.couponAMT = 99.002;

DatatistProductInfo \*productInfo = \[DatatistProductInfo new\];
productInfo.sku = @"4567934501204569";
productInfo.productTitle = @"小米 Mix 2";
productInfo.productOriPrice = 3999.0;
productInfo.productRealPrice = 0.01;
productInfo.productQuantity = 3;
productInfo.productSourceSku = @"xxxxxxxxxxxxxxxxx";

DatatistProductInfo *productInfo2 = [DatatistProductInfo new];
productInfo2.sku = @"4567934501204569";
productInfo2.productTitle = @"华为mate9 ";
productInfo2.productOriPrice = 3999.0;
productInfo2.productRealPrice = 0.01;
productInfo2.productQuantity = 3;
productInfo2.productSourceSku = @"xxxxxxxxxxxxxxxxx";

[[DatatistTracker sharedInstance] trackOrder:orderInfo couponInfo:@[couponInfo, couponInfo2] productInfo:@[productInfo, productInfo2] udVariable:nil];
```

### 4.7支付订单

说明：支付成功时调用，采集支付信息，可以用map添加自定义变量 。

##### 接口声明：

```
- (void)trackPayment:(NSString *)orderId payMethod:(NSString *)method payStatus:(BOOL)pay payAMT:(double)amt udVariable:(NSDictionary *)vars;
```

API说明：采集订单支付事件 。

##### 参数说明：

orderID: 订单号

payMenthod: 支付渠道

payStatus: 支付状态

payAMT: 支付总金额

udVariable: 客户可扩展的自定义变量，以NSDictionary对象的形式进行传输

样例程序：

```
[[DatatistTracker sharedInstance] trackPayment:@"2017101716591100" payMethod:@"支付宝" payStatus:true payAMT:9000.0 udVariable:nil];
```

### 4.8预充值

说明：充值成功时调用，采集充值信息，可以用map添加自定义变量 。

##### 接口声明：

```
- (void)trackPreCharge:(double)amt chargeMethod:(NSString *)chargeMethod couponAMT:(double)coupon payStatus:(BOOL)pay udVariable:(NSDictionary *)vars;
```

API说明：采集预充值事件 。

##### 参数说明：

chargeAMT: 充值金额

chargeMethod: 充值渠道

couponAMT: 充值优惠金额

payStatus: 支付状态

udVariable: 客户可扩展的自定义变量，以NSDictionary对象的形式进行传输

样例程序：

```
[[DatatistTracker sharedInstance] trackPreCharge:600.0 chargeMethod:@"微信支付" couponAMT:200.0 payStatus:true udVariable:nil];
```

## 5.自定义事件

### 5.1自定义事件

说明：在任意位置添加采集代码，记录用户行为或其他你所关注的数据，可以用map添加自定义变量 。。

##### 接口声明：

```
- (void)trackEvent:(NSString *)name udVariable:(NSDictionary *)vars;
```

##### 参数说明：

name: 事件名称

udVariable: 客户可扩展的自定义变量，以NSDictionary对象的形式进行传输

样例程序：

```
[[DatatistTracker sharedInstance] trackEvent: @"custom"    udVariable:@{@"happenTime": @((long long)[[NSDate date] timeIntervalSince1970] * 1000)}];
```

### 5.2建议自定义埋点事件--启动页

说明: 在APP打开时2秒的启动页时，建议添加埋点，用于标识APP的启动，用户行为的开始。其中”init”参数请勿修改！

示例：

```
[[DatatistTracker sharedInstance] trackEvent: @"init " udVariable:nil];
```



