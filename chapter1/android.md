# Android SDK 接入

## 1.安装SDK

点击  下载配置文件

最新版Datatist SDK 支持Android API Version &gt;= 10 \(Android 2.3.3+\)

datatist Android SDK 支持2种安装方式

### 安装方式A：Eclipse安装

通过Eclipse将下载的“datatist-sdk.jar”文件放至工程的libs文件夹中。

### 安装方式B：Gradle 编译环境

在Android studio项目中创建libs文件夹，将下载的“datatist-sdk.jar”文件放至libs文件夹中。在module级别的build.gradle文件中添加依赖：

```
dependencies {
 repositories {
 jcenter()
 }
 ……
 compile files('libs/datatistsdk.jar')
}
```

完成导入后在APP里添加继承DatatistApplication：

```
public class YourApplication extends DatatistApplication
```

在YourApplication中完成两个继承自DatatistApplication 的成员函数：

· public String setTrackerUrl\(\) – 返回URL地址

· public String setSiteId\(\) – 返回分配的SiteId

```
@Override
public String setTrackerUrl() {
 return "http://******";
}
@Override
public String setSiteId() {
 return "your_siteID";
}
```

## 2.权限配置

| 权限 | 用途 |
| :--- | :--- |
| INTERNET | 允许应用访问网络 |
| READ\_PHONE\_STATE | 允许应用访问手机状态 |
| ACCESS\_NETWORK\_ATATE | 允许应用获取网络信息状态，如当前网络连接是否有效 |
| ACCESS\_WIFI\_STATE | 允许应用访问WiFi网络状态 |
| CHANGE\_CONFIGURATION | 允许当前应用改变配置 |

## 3.符号定义

@NonNull：被用来标注给定的参数或者返回值不能为null。

在module级别的build.gradle添加依赖关系：

```
dependencies {
    compile 'com.android.support:support-annotations:22.2.0'
}
```

## 4.初始化SDK

为了确保数据采集准确性，在application类中创建管理TrackerKernel。

```
private void initDatatist() {
    // TrackerKernel对象对基本信息进行设置，
    // setChannelId：设置渠道信息
    TrackerKernel trackerKernel = DatatistApp.this.getTracker();
    if(trackerKernel != null) {
        trackerKernel.setChannelId(15);//渠道信息
    }
}
```

## 5.基础配置

##### userID：

使用userID可以帮助Tracker收集同用户在不同设备和浏览器上的信息，以长期定位追踪用户信息以及支持跨设备和浏览器行为的关联。userID是一个非空的字符串，比如用户名，邮箱地址，手机号 等唯一识别此用户。userID在不同设备和浏览器上必须是相同的。可以将userID进行加密后进行传输。

示例：

```
Track.track().tracker().setUserId("your userID");
```

建议设置userID的位置如下：

· 在初始化Tracker后即设置已注册客户保存的userID

· 在注册流程结束后设置新注册用户的userID

· 在登录流程结束后设置已有用户登录的userID

##### 页面采集：

进入某页面时采集所需数据，用于分析页面来源，统计浏览的页面路径，名称，访问时长，所使用的浏览器等信息

接口声明：

```
public Pageview trackPageview(Map<String, String> udVariable)
```

参数说明：

udVariable: 客户可扩展的自定义变量，以MAP对象的形式进行存储；

示例：

```
final Map<String, String> eventMap = new HashMap<>(); 
eventMap.put("xxxx "," xxxx ");
TrackerKernel tracker = ((YourApplication) YourActivity.this.getApplication()).getTracker(); 
Track.track().pageview() 
    .setTitle("页面标题")
    .setUrl\(“当前activity路径”\)
     .trackPageview\(eventMap\)
     .submit\(tracker\);
```

建议设置页面采集的位置如下：

·建议将页面采集接口放在父类（BaseActivity）中实现

## 6.基础事件

### 6.1搜索采集

说明：点击搜索按钮时，采集搜索的关键词及关键词来源，如热门推荐词或历史记录，可以用map添加自定义变量 。

##### 接口声明：

```
public Search trackSearch(@NonNull String keyword, boolean recommendationSearchFlag, boolean historySearchFlag, Map<String, String> udVariable)
```

##### 参数说明：

keyword：搜索关键词

recommendationSearchFlag：使用推荐搜索关键词的标志，值为true/false

historySearchFlag：使用搜索历史记录来进行搜索，值为true/false

udVariable: 客户可扩展的自定义变量，以MAP对象的形式进行存储；

示例：

```
final Map<String, String> eventMap = new HashMap<>();
eventMap.put("范围","热带"); 
eventMap.put("价格","大于100");
 eventMap.put("附加","运输方便"); 
eventMap.put("方式","物流"); 
TrackerKernel tracker = ((YourApplication) YourActivity.this.getApplication()).getTracker();
 Track.track().search() 
    .trackSearch("水果之王", true, true, eventMap) 
    .submit(tracker);
```

### 6.2用户注册

说明：用户注册成功的回调中，先调用setUserId接口，再调用用户注册接口，采集注册信息，可以用map添加自定义变量 。

##### 接口声明：

```
public Register trackRegister(@NonNull String uid, @NonNull String type, boolean authenticated, Map<String, String> udVariable)
```

##### 参数说明：

uid: 用户注册的用户ID；

type: 用户类型；

authenticated: 是否已认证的标识，值为true/false

udVariable: 客户可扩展的自定义变量，以MAP对象的形式进行存储；

示例：

```
final Map<String, String> eventMap = new HashMap<>();
eventMap.put("用户公司"," xxx公司"); 
eventMap.put("xxxx "," xxxx ");
TrackerKernel tracker = ((YourApplication) YourActivity.this.getApplication()).getTracker();
Track.track() 
    .register() 
    .trackRegister(userId,"VIP",true, eventMap) 
    .submit(tracker);
```

### 6.3用户登录

说明：用户登录成功的回调中，先调用setUserId接口，再调用用户登录接口，采集登录信息，可以用map添加自定义变量 。

##### 接口声明：

```
public Login trackLogin(@NonNull String uid, Map<String, String> udVariable)
```

##### 参数说明：

uid: 用户ID

udVariable: 客户可扩展的自定义变量，以MAP对象的形式进行传输；

示例：

```
final Map<String, String> eventMap = new HashMap<>();
eventMap.put("用户类型","企业用户");
 eventMap.put("用户公司"," xxx公司");
 TrackerKernel tracker = ((YourApplication) YourActivity.this.getApplication()).getTracker(); 
Track.track().login() 
    .trackLogin("uid-2017101609320300",eventMap) 
    .submit(tracker);
```

### 6.4产品访问页

说明：进入商品详情页后调用，采集商品信息，可以用map添加自定义变量 。

##### 接口声明：

```
public ProductPage trackProductPage(@NonNull String sku, @NonNull String productCategory1, @NonNull String productCategory2,@NonNull String productCategory3, double productOriginalPrice, double productRealPrice, Map<String, String> udVariable)
```

##### 参数说明：

sku: 页面产品的SKU信息；

productCategory1: 一级目录；

productCategory2: 二级目录，没有值就留空；

productCategory3: 三级目录，没有值就留空；

productOriginalPrice: 商品原价

productRealPrice: 商品优惠后的价格

udVariable: 客户可扩展的自定义变量，以MAP对象的形式进行传输；

示例：

```
final Map<String, String> eventMap = new HashMap<>();
eventMap.put("参与活动"," XX促销"); 
eventMap.put("xxxx ","xxxx "); 
TrackerKernel tracker = ((YourApplication) YourActivity.this.getApplication()).getTracker();
 Track.track().productPage() 
    .trackProductPage("商品sku","果蔬", "柑橘类", "澳大利亚脐橙", 100.00, 67.00, eventMap) 
    .submit(tracker);
```

### 6.5加入购物车

说明：点击加入购物车按钮时调用，采集加入购物车的商品信息，可以用map添加自定义变量 。

##### 接口声明：

```
public Cart trackAddCart(@NonNull String sku, long productQuantity, double productRealPrice, Map<String, String> udVariable)
```

##### 参数说明：

sku: 被加入购物车的SKU商品信息；

productQuantity: 被加入购物车的SKU商品数量；

productRealPrice: 被加入购物车的商品单价

udVariable: 客户可扩展的自定义变量，以MAP对象的形式进行传输；

示例：

```
final Map<String, String> eventMap = new HashMap<>();
eventMap.put("商品类型","买一赠一活动商品"); 
eventMap.put("xxxx "," xxxx "); 
TrackerKernel tracker= ((YourApplication) YourActivity.this.getApplication()).getTracker(); 
Track.track().cart() 
    .trackAddCart(sku, 10000, 67.00, eventMap) 
    .submit(tracker);
```

### 6.6生成订单

说明：提交订单时调用，设置订单中的必要参数，采集订单信息，可以用map添加自定义变量 。

##### 接口声明：

```
public Order trackOrder(@NonNull OrderInfo orderInfo,@NonNull CouponInfo couponInfo,@NonNull ProductInfo productInfo,Map<String, String> udVariable)
```

##### 参数说明：

OrderInfo: 订单信息，是一个MAP对象，其中包含项5项具体的参数：

·orderID: string订单号

·orderAMT: double 订单总价

·shipAMT: double 运费总价

·shipAddress: string 收货地址

·shipMethod: string 配送方式

CouponInfo: 优惠券信息，是一个MAP数组，其中包含2项具体的参数：

·couponType: string 优惠券类型

·couponAMT: string优惠券金额

ProductInfo: 产品信息，是一个MAP数组，其中包含6项具体的参数：

·productSKU: string 产品SKU

·productTitle: string 产品名称

·productRealPrice: double产品实际成交价

·productOriPrice: double产品原价

·productQuantity: double产品数量

·productSourceSku: string 活动商品来源（例如赠品产品。传原商品sku，标识原商品的绑定关系）

·udVariable: 客户可扩展的自定义变量，以MAP对象的形式进行传输；

示例：

```
final Map<String, String> eventMap = new HashMap<>();
OrderInfo orderInfo = new OrderInfo("ORDERID ********", 124502.023,332.00, "上海市徐汇区宜山路333号汇鑫国际603", "顺丰快递"); 
CouponInfo couponInfo = new CouponInfo();
 CouponItem couponItem = new CouponItem("满20减1", 1.00);
 couponInfo.add(couponItem); 
couponItem = new CouponItem("1元红包", 1.00);
 couponInfo.add(couponItem);  
ProductInfo productInfo = new ProductInfo();
 ProductItem productItem = new ProductItem("产品SKU1","榴莲",67.00,100.00,3,"");
 productInfo.add(productItem); 
productItem = new ProductItem("sku:43557566343546700","菠萝",35.00,45.00,10,"");
 productInfo.add(productItem);
 productItem = new ProductItem("sku:43557566343547653","iPhone X",8900,9000,2,"");
 productInfo.add(productItem);

  eventMap.put("目的地","上海"); 
eventMap.put("客户类型","代理");
 eventMap.put("客户","SPEEDO"); 
TrackerKernel tracker = ((YourApplication) YourActivity.this.getApplication()).getTracker();
 Track.track().order() 
    .trackOrder(orderInfo, couponInfo, productInfo, eventMap) 
    .submit(tracker);
```

### 6.7支付订单

说明：支付成功时调用，采集支付信息，可以用map添加自定义变量 。

##### 接口声明：

```
public Payment trackPayment(@NonNull String orderID, String payMethod, boolean payStatus, double payAMT, Map<String, String> udVariable)
```

##### 参数说明：

orderID: 订单号

payMethod: 支付渠道

payStatus: 支付状态，值为true/false

payAMT: 支付总金额

udVariable: 客户可扩展的自定义变量，以MAP对象的形式进行传输；

示例：

```
final Map<String, String> eventMap = new HashMap<>();
eventMap.put("参加活动","免配送费"); 
eventMap.put("xxxx "," xxxx "); 
TrackerKernel tracker = ((YourApplication) YourActivity.this.getApplication()).getTracker(); 
Track.track().payment() 
    .trackPayment("orderId-24316435A88Y2","支付宝&微信", true, 124502.023, eventMap) 
    .submit(tracker);
```

### 6.8预充值

说明：充值成功时调用，采集充值信息，可以用map添加自定义变量 。

##### 接口声明：

```
public PreCharge trackPreCharge(double chargeAMT, @NonNull String chargeMethod, double couponAMT, boolean payStatus, Map<String, String> udVariable)
```

##### 参数说明：

chargeAMT: 充值金额

chargeMethod: 充值渠道

couponAMT: 充值优惠金额

payStatus: 支付状态，值为true/false

udVariable: 客户可扩展的自定义变量，以MAP对象的形式进行传输；

示例：

```
final Map<String, String> eventMap = new HashMap<>();
eventMap.put("充值目的地","上海");
 eventMap.put("充值客户类型","个人");
 eventMap.put("充值客户ID","P2017101700329");
 TrackerKernel tracker = ((YourApplication) YourActivity.this.getApplication()).getTracker(); 
Track.track().preCharge() 
    .trackPreCharge(1000.00,"微信充值",10,true,eventMap) 
    .submit(tracker);
```

## 7.自定义事件

### 7.1自定义事件

说明：在任意位置添加采集代码，记录用户行为或其他你所关注的数据，可以用map添加自定义变量 。

##### 接口声明：

```
public Event trackEvent(@NonNull String eventName, Map<String, String> udVariable)
```

##### 参数说明：

eventName: 事件名称

udVariable: 客户可扩展的自定义变量，以MAP对象的形式进行传输；

示例：

```
final Map<String, String> eventMap = new HashMap<>();
eventMap.put("活动ID","HDL1111"); 
eventMap.put("活动标题","双十一特惠"); 
TrackerKernel tracker = ((YourApplication) YourActivity.this.getApplication()).getTracker();
 Track.track().event() 
    .trackEvent("活动信息",eventMap) 
    .submit(tracker);
```

### 7.2建议自定义埋点事件--下载渠道

说明：APP在多渠道发布时，建议添加埋点，用于多渠道分析，建议用自定义事件加上此埋点

示例：

```
final Map<String, String> eventMap = new HashMap<>();    
eventMap.put("渠道ID","HDL1111"); 
eventMap.put("渠道名称","应用宝");
 TrackerKernel tracker = ((YourApplication) YourActivity.this.getApplication()).getTracker(); 
Track.track().event() 
    .trackEvent("下载渠道",eventMap) 
    .submit(tracker);
```

### 7.3建议自定义埋点事件--启动页

说明: 在APP打开时2秒的启动页时，建议添加埋点，用于标识APP的启动，用户行为的开始。其中”init”参数请勿修改！

示例：

```
final Map<String, String> eventMap = new HashMap<>();
eventMap.put("xxxxx","xxxxx");
 TrackerKernel tracker = ((YourApplication) YourActivity.this.getApplication()).getTracker();
 Track.track().event() 
    .trackEvent("init",eventMap) 
    .submit(tracker);
```



