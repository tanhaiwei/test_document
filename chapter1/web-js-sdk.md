# Web JS SDK 接入

## 1.安装SDK

如果要跟踪H5页面访问，需要在跟踪的页面的&lt;Head&gt;或者&lt;footer&gt;内加上Datatist跟踪码，判断有userID时，请设置userID。初始化跟踪代码如下：

```
<script type="text/javascript">  
    window.dtTracker=window.dtTracker||function(){(dtTracker.q=dtTracker.q||[]).push(arguments)};  
    dtTracker.l=+new Date;   
    dtTracker.d=true;  
    dtTracker('setTrackApiUrl', 'https://xx.xx.xxx/c.gif');//请设置技术人员提供的相关url   
    dtTracker('setSiteId', 'xxxx');//请设置技术人员提供的相关SiteId  
    dtTracker('pageview');//页面捕捉
    //登录或注册成功时或cookie中有userid，则需设置userID；没有则不设置  
    //dtTracker('setUserId', 'xxxx');  

    (function() {  
        var d = document,  
            g = d.createElement('script'),  
            s = d.getElementsByTagName('script')[0];  
    g.type = 'text/javascript';  
    g.async = true;  
    g.defer = true;  
    g.src = 'http://xxx.xxx.xxx/tracker.js';  
    s.parentNode.insertBefore(g,s)  
    })();  
</script>
```

## 2.基础配置

#### userID

使用userID可以帮助Tracker收集同用户在不同设备和浏览器上的信息，以长期定位追踪用户信息以及支持跨设备和浏览器行为的关联。userID是一个非空的字符串，比如用户名，邮箱地址，手机号 等唯一识别此用户。userID在不同设备和浏览器上必须是相同的。可以将userID进行加密后进行传输。

```
dtTracker('setUserId', 'xxxx');
```

建议设置userID的位置如下：

* 在初始化Tracker后即设置已注册客户保存的userID
* 在注册流程结束后设置新注册用户的userID
* 在登录流程结束后设置已有用户登录的userID

#### 页面采集

进入某页面时采集所需数据，用于分析页面来源，统计浏览的页面路径，名称，访问时长，所使用的浏览器等信息。

##### 接口声明：

```
trackPageview(udVariable : object,callback: function|option)
```

##### 参数说明：

1.udVariable: 客户可扩展的自定义变量，以JSON对象的形式进行存储；

```
//进入某页面时采集所需数据
var mjson={  
"XXXXX":"XXXX"  
};
dtTracker('pageview',mjson);
```

建议设置页面采集的位置如下：

初始化代码中已默认调用页面捕获接口。

## 3.事件配置

### 3.1自定义事件

说明：在任意位置添加采集代码，记录用户行为或其他你所关注的数据，可以用JSON添加自定义变量。

##### 接口声明：

```
trackEvent(eventName: string,udVariable: object,callback: function|option)
```

##### 参数说明：

1.eventName:事件名称；

2.udVariable: 客户可扩展的自定义变量，以JSON对象的形式进行传输；

示例：

```
//可以在任意位置记录用户行为或其他数据  
var mjson={  
"活动ID":"1",  
"活动标题":"XXXX"  
};  
dtTracker('event','首页活动',mjson);
```

### 3.2搜索采集

说明：点击搜索按钮时，采集搜索的关键词及关键词来源，如热门推荐词或历史记录，可以用JSON添加自定义变量。

##### 接口声明：

```
trackSearch (keyword: string, recommendationSearchFlag: boolean, historySearchFlag: boolean, udVariable : object, callback: function|option)
```

##### 参数说明：

1.keyword：搜索关键词

2.recommendationSearchFlag：使用推荐搜索关键词的标志，值为true/false

3.historySearchFlag：使用搜索历史记录来进行搜索，值为true/false

4.udVariable: 客户可扩展的自定义变量，以JSON对象的形式进行存储；

示例：

```
//点击搜索按钮时，采集搜索数据  
//若无非必传项，可直接调用trackSearch("搜索的关键词");  
var mjson={  
"活动":"水果大促销",  
"XXXXX":"XXXX"  
};  
dtTracker('search','火龙果',false,false,mjson);
```

### 3.3用户注册

说明：用户注册成功的回调中，先调用setUserId接口，再调用用户注册接口，采集注册信息，可以用JSON添加自定义变量。

##### 接口声明：

```
trackRegister (uid: string, type: string, authenticated: boolean, udVariable: object , callback: function|option)
```

##### 参数说明：

1.uid: 用户注册的用户ID；

2.type: 用户类型；是否已认证的标识，值为true/false

3.authenticated: 是否已认证的标识，值为true/false

4.udVariable: 客户可扩展的自定义变量，以JSON对象的形式进行存储；

示例：

```
//用户注册成功时，采集注册信息  
var mjson={  
"用户公司":"datatist",  
"XXXXX":"XXXX"  
};  
dtTracker('register','yourUserId','企业用户',true,mjson);
```

### 3.4.用户登录

说明：用户登录成功的回调中，先调用setUserId接口，再调用用户登录接口，采集登录信息，可以用JSON添加自定义变量。

##### 接口声明：

```
trackLogin (uid:string,udVariable:object, callback:function|option)
```

##### 参数说明：

1.uid: 用户ID

2.udVariable: 客户可扩展的自定义变量，以JSON对象的形式进行传输；

示例：

```
//用户登录成功时，采集用户信息
var mjson={
"用户类型":"企业用户",
"用户公司":"datatist"
};
dtTracker('login','yourUserId',mjson);
```

### 3.**5.产品访问页**

说明：进入商品详情页后调用，采集商品信息，可以用JSON添加自定义变量。

##### 接口声明：

```
trackProductPage (sku:string, category1:String, catgory2:String, category3:String, originalPrice:double, realPrice:double, udVariable:object, callback: function|option)
```

##### 参数说明：

1.sku:页面产品的SKU信息；

2.category1:一级目录；

3.category2:二级目录，没有值就留空；

4.category3:三级目录，没有值就留空；

5.originalPrice:商品原价；

6.realPrice:商品优惠后的价格；

7.udVariable:客户可扩展的自定义变量，以JSON对象的形式进行传输；

示例：

```
//进入商品详情页后，采集商品信息
var mjson={
"参与活动":"XX促销"
};
dtTracker('productPage','skuxxxx','果蔬','柑橘类','澳大利亚脐橙',100.0,98.0,mjson);
```

### 3.6.加入购物车

说明：点击加入购物车按钮时调用，采集加入购物车的商品信息，可以用JSON添加自定义变量。

##### 接口声明：

```
trackAddCart (sku:string,quantity:int, realPrice:double, udVariable:object, callback: function|option)
```

##### 参数说明：

1.sku:被加入购物车的SKU商品信息；

2.quantity:被加入购物车的SKU商品数量；

3.realPrice:被加入购物车的商品单价

4.udVariable:客户可扩展的自定义变量，以JSON对象的形式进行传输；

示例：

```
//点击加入购物车按钮时调用
var mjson={
"商品类型":"买一赠一活动商品"
};
dtTracker('addCart','skuxxxx',2,10.0,mjson);
```

### 3.7.生成订单

说明：提交订单时调用，设置订单中的必要参数，采集订单信息，可以用JSON添加自定义变量。

##### 接口声明：

```
trackOrder (orderInfo: object, couponInfo: object, productInfo: object, udVariable: object , callback: function|option)
```

##### 参数说明：

1.orderInfo: 订单信息，是一个JSON数组，其中包含项5项具体的参数：

* orderID: string订单号
* orderAMT: double 订单总价
* shipAMT: double 运费总价
* shipAddress: string 收货地址
* shipMethod: string 配送方式

2.couponInfo: 优惠券信息，是一个JSON数组，其中包含2项具体的参数：

* couponType: string 优惠券类型
* couponAMT: double 优惠券金额

3.productInfo: 产品信息，是一个JSON数组，其中包含6项具体的参数：

* productSKU: string 产品SKU
* productTitle: string 产品名称
* productRealPrice: double 产品实际成交价
* productOriPrice: double 产品原价
* productQuantity: double产品数量
* productSourceSku: string 活动商品来源（例如赠品产品。传原商品sku，标识原商品的绑定关系）

4.udVariable: 客户可扩展的自定义变量，以JSON对象的形式进行传输。

示例：

```
//设置订单中的必要参数  
var orderInfo = {  
"orderID":"xxxxxx",  
"orderAMT":20.0,  
"shipAMT":0.0, 
"shipAddress":"上海市xxxx",  
"shipMethod":"xxx快递"  
};  
//设置优惠券
var couponInfo = 
[
    {
        "couponType": "满20减1",
        "couponAMT": 1.0
    },
    {
        "couponType": "1元红包",
        "couponAMT": 1.0
    }
]; 
//设置产品，例如买一赠一的商品
var productInfo = 
    [
    {
        "productSKU": "菠萝的SKU1",
        "productTitle": “菠萝（买一赠一苹果）",
        "productRealPrice": 12.0,
        "productOriPrice": 11.2,
        "productQuantity": 1,
        "productSourceSku": ""
    },
    {
        "productSKU": "苹果的SKU2",
        "productTitle": " 苹果（买一赠一苹果）",
        "productRealPrice": 2.0,
        "productOriPrice": 1.2,
        "productQuantity": 1,
        "productSourceSku": "菠萝的SKU1"
    }
    ]；  
//提交订单  
var mjson={  
"参与活动":"免配送费"  
};  
dtTracker('order',orderInfo ,couponInfo ，productInfo ,mjson);
```

### 3.8.支付订单

说明：支付成功时调用，采集支付信息，可以用JSON添加自定义变量。

##### 接口声明：

```
trackPayment (orderID:string, payMethod:string, payStatus:boolean, payAMT:double, udVariable:object, callback: function|option)
```

##### 参数说明：

1.orderID:订单号

2.payMethod:支付渠道

3.payStatus:支付状态，值为true/false

4.payAMT:支付总金额

5.udVariable:客户可扩展的自定义变量，以JSON对象的形式进行传输；

示例：

```
//支付成功时，记录支付信息
var mjson={  
"参与活动":"免配送费"
};  
dtTracker('payment','订单ID','xx网银',true,18.2,mjson);
```

### 3.9.预充值

说明：充值成功时调用，采集充值信息，可以用JSON添加自定义变量。

##### 接口声明：

```
trackPreCharge (chargeAMT:double, chargeMethod:string, couponAMT:double, payStatus:boolean, udVariable:object, callback: function|option)
```

##### 参数说明：

1.chargeAMT:充值金额

2.chargeMethod:充值渠道

3.couponAMT:充值优惠金额

4.payStatus:支付状态，值为true/false

5.udVariable:客户可扩展的自定义变量，以JSON对象的形式进行传输；

示例：

```
//充值成功时，记录充值信息
var mjson={  
"参与活动":"充100得120"
};  
dtTracker('preCharge',100.0,'xx网银',20.0,true,mjson);
```



