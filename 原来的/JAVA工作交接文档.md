# 项目模块

## 互联网医院

互联网医院负责模块  

用户在提交病例信息的时候



## 百度健康

**百度健康对接人 金威威**

百度健康,是作的运营平台无前端就只作订单处理

**核心逻辑**

定时任务一个是查询百度健康没有载入G3的订单

定时任务查看商品是否发货如果发货就调用百度健康发货接口批量发货

定时任务:定时的去执行库存更新

还有一个O2O模块是栋哥负责的,我这边提供了商品查询

| 项目名称          | 端口  | 服务器 | 项目部署地址                 |
| ----------------- | ----- | ------ | ---------------------------- |
| thirdpartydocking | 58005 | 101    | /home/java/wangjiaxiang/bdjk |

核心代码路径

```java
 com.zzjdyf.thirdpartydocking.timer.order.BaiDuOrderTimer
```

代码核心方法

```java
	@GetMapping("order")
	@Scheduled(cron = " 0 0/40 * * * ?")
	//订单批量入账方法
	private void order() {
		BaiDuOrderInfoPO baiDuOrderInfoPO = injectOrderInfo(BaiDuOrderConstant.STATUS, null);
		if (baiDuOrderInfoPO == null) {
			return;
		}
		if (baiDuOrderInfoPO.getData() == null) {
			return;
		}
		List<PlatformB2cOrderInfo> b2cOrderInfos = new ArrayList<PlatformB2cOrderInfo>();
		List<BaiDuOrderInfo> orderInfos = baiDuOrderInfoPO.getData().getData();
		//记录发送信息
		List<BaiduOrder> baiduOrders = new ArrayList<BaiduOrder>();
		for (BaiDuOrderInfo orderInfo : orderInfos) {
			String orderId = orderInfo.getOrderId();
//			System.out.println(orderId);
//			if (!"1011653698262154296".equals(orderId)) {
//				continue;
//			}
			//参数非空检测
			Boolean detection = detection(orderInfo);
			//参数拼接,调用G3过账
			if (!detection) {
				continue;
			}
			PlatformB2cOrderInfo b2cOrderInfo = jointB2cOrderInfo(orderInfo);
			if (b2cOrderInfo == null) {
				continue;
			}
			BaiduOrder byCode = baiduOrderDao.queryByCode(b2cOrderInfo.getOrderCode());
			if (byCode != null) {
				continue;
			}
//			System.out.println("*************");
//			System.out.println(JSONObject.toJSON(b2cOrderInfo));
			b2cOrderInfos.add(b2cOrderInfo);
			BaiduOrder baiduOrder = storageOrderInfo(b2cOrderInfo);
			baiduOrders.add(baiduOrder);
		}
		if (0 == baiduOrders.size()) {
			return;
		}
//		System.out.println("*************");
//		System.out.println(JSONObject.toJSON(b2cOrderInfos));
//		String omsStr = OmsRequestTool.doPostOms("http://127.0.0.1:31005/oms/limit/base/platform/order/b2c/infos", b2cOrderInfos);
		String omsStr = OmsRequestTool.doPostOms(OmsApi.PlatformOrderApi.receiveB2cOrdersApi.ABSOLUTE_URL, b2cOrderInfos);
		OptResult optResult = JSONObject.parseObject(omsStr, OptResult.class);
		if (optResult.getSuccess()) {
			baiduOrderDao.addOrderInfos(baiduOrders);
		} else {
			logger.error("入账失败\n" + optResult + "\n" + JSONObject.toJSON(b2cOrderInfos));
		}
		logger.info("入账结果" + optResult);
	}
```

```java
	/**
	 * 更新发货信息
	 */
	@Scheduled(cron = "0 0/10 * * * ?")
	private void updateMailInfo() {
		//查询出所有没有快递信息的订单
		List<BaiduOrder> baiduOrders = baiduOrderDao.queryByMailTag();
		if (baiduOrders.size() == 0) {
			return;
		}
		//百度健康快递信息
		MailInfoData mailInfoData = new MailInfoData();
		List<MailInfo> mailInfos = new ArrayList<MailInfo>();
		StringBuffer mailCodeLong = new StringBuffer();
		for (BaiduOrder baiduOrder : baiduOrders) {
			MailInfo mail = new MailInfo();
			OmsOrderB2cHelpInfo omsMailInfo = queryMailInfo(baiduOrder.getOrderCode());
			//如果发现
			if (omsMailInfo == null) {
				continue;
			}
			if (mailInfos.size() == 9) {
				break;
			}
			//信息更新到数据库
			BaiduOrder mailInfo = mailCompanyName(baiduOrder, omsMailInfo);
			baiduOrderDao.updateMailInfo(mailInfo);
			mailCodeLong.append(mailInfo.getOrderCode() + "/t");
			//同步更新百度健康
			mail.setDeliverName(mailInfo.getMailName());
			mail.setDeliverNo(mailInfo.getOrderMailCode());
			mail.setOrderId(mailInfo.getOrderCode());
			mail.setStoreId(BaiDuOrderConstant.STORE_ID);
			//药过期时间 批次号
			//mailJointInfos(mail);
			mailInfos.add(mail);
		}
		//注入百度健康发货参数
		if (mailInfos.size() != 0) {
			logger.info("本次入账订单" + mailCodeLong.toString());
			mailInfoData.setData(mailInfos);
			orderAddDeliver(mailInfoData);
		}
	}
```

详细的可以看代码里面具体详细操作

中间发货的时候牵扯到发货的批次号/货号 可能会出现批次号货号不存在.导致药品上传货号失败,目前不太影响平台还没强管制

后面可能需要修改,里面代码个人感觉写的比较杂乱具体可以在看

但是封装好了对接逻辑详细在百度健康第三方对接方面去看

## 企微助手

| 项目名称 | 端口  | 服务器 | 项目部署地址                 |
| -------- | ----- | ------ | ---------------------------- |
| dkyj     | 58005 | 101    | /home/java/wangjiaxiang/qywx |

(代客寄药)（门店欢迎语）（督导店长查店打分）（群活码）

业务简介

初衷是作待客寄药 后期对这个业务进行逐渐扩展,越来约多就塞一个项目里面了

### 代客寄药

**待客寄药项目状态，目前被暂停了极少应用**

是用来让门店寄快递,给用户方便操作的一个平台

业务逻辑是员工填如购买信息,然后提交给运营审核,审核后调取快递API接口呼叫快递人员

并同步快递单号到本地数据库信息里面

待客寄药项目代码controller包地址 

com.zzjdyf.dkyj.controller.dkjy

### **企业微信欢迎语**

Controller包地址com.zzjdyf.dkyj.controller.dkjy.greet

详细代码根据Controlller可以去往下看Service实现,方法注释基本都有

### **群活码**

Controller包地址 com.zzjdyf.dkyj.controller.crowd

**以上三个项目用的很少基本不在进行应用**

### 督导打分

**此项目还是经常应用  项目主要使用部门(审计督察部) 对接人    宋军 工号**024389

项目简介

用来让督导(地区经理,大区总)来让他们日常检查门店,对门店进行打分的功能主要应用

功能简介

督导登入后可以输入门店编号来进行查店,然后来判断督导当前位置与门店记录的地理位置信息距离是否过远,这个距离是可以后端规定的

代码位置展示

```java
com.zzjdyf.dkyj.service.xd.info.supervisor.impl.SupervisorServiceImpl
   	/**
	 * 输入门店编号查看门店下人员
	 * @param storeCode 门店编号
	 * @param lat 经度
	 * @param lng 维度
	 * @return
	 */
	public EntityResult<MemberInformationDO> findPersonnelList(String storeCode,Double lat,Double lng) {
		if (lat == null || lng == null){
			return ResultUtil.entResult(new OptResult(31002,"请检查手机定位,以及检查企业微信是否有位置权限"));
		}
		//经纬度开始
		EntityResult<OmsStorePlaceInfoDTO> omsStore = omsStoreService.queryStorePlace(storeCode);
		OmsStorePlaceInfoDTO storeInfo = omsStore.getData();
		//OMS门店是否存在
		if (storeInfo == null){
			return ResultUtil.entResult(new OptResult(31003,"G3位置信息错误请查看维护格式,或者检测门店"+storeCode+"是否存在"));
		}
//		if (storeInfo.getLat()==null || storeInfo.getLng() == null){
//			return new EntityResult<>(new OptResult(31004,"请去G3维护"+storeCode+"门店经纬度"));
//		}
		Double distance = DistanceUtil.getDistance(storeInfo.getLat(), storeInfo.getLng(), lat, lng)*1000;
		//距离判断
		if (distance>1500){
			Integer distanceMeter = distance.intValue();
			return ResultUtil.entResult(new OptResult(31004,"当前距离门店"+distanceMeter+"米,无法对该门店进行打分"));
		}
    	//返回门店员工信息
		return relevanceInfoService.findPersonnelList(storeCode);
	}
```

**然后督导在对检查项去进行打分记录数据**

在管理员接口中有个根据月份对数据进行导出的。

项目BUG

由于门店经纬度并不是我们自己维护的,可能出现地理位置不一致的情况,这时候需要经理帮忙在G3维护,因为运行过一段时间了,大多经理都会维护

个别询问的让他找宋军.

101服务器网络带宽不稳定造成卡顿，去101服务器上面检查一下服务是否存在就可以了 ,如果服务器存在就对外说网络问题就好了

**坑---**

数据存储记录目前数据存储记录是通过另外一个项目进行存储的,因为当时说要使用Orcale数据,数据导表交给信息部进行处理
如果这块要重新编写的话建议直接写道mysql里面,直记录总记录就可以了

还有在记录的时候会记录各项分数,督导对这个分数不使用,只使用总分,所以说可以不进行记录

具体的话就看代码对应的Controller包地址

```java
//下面的所有服务
com.zzjdyf.dkyj.service.xd
```



## **附近门店**

项目简介

附近门店是用一个二维码来获取附近范围两公里的门店方便用来用户加店长企业微信

调用逻辑比较简单附近门店接口,在oms项目里面就有然后通过门店名称查询企业微信API接口获取门店员工信息

筛选出店长展示就好了 

| 项目名称            | 端口  | 服务器 | 项目部署地址            |
| ------------------- | ----- | ------ | ----------------------- |
| **wechatthepublic** | 55005 | 101    | /home/java/wangjiaxiang |

项目技术 SpringBoot  html+js(前后端分离) redis  企业微信API  myabtis

主要的Controller类

com.zzjdyf.wechatthepublic.wx.member.controller.WeChatMemberController

由于项目并不复杂,所以不作太多详细解释

# 对接三方接口文档交接

对接过的三方接口,以及简易对接经验

这里简单距离 详细看代码进行修改

## 企业微信

https://developer.work.weixin.qq.com/document/path/90664

对接过的API

通过部门获取部门下所有人

通过工号查询个人详细信息

通过机器人发送消息信息

设置欢迎语

通过token实现企业微信用户授权登入

调用企业微信接口的时候需要获取对应的token

```java
对应的ID还有Token都在项目里面记录的有,是已经写好的可以直接用    
//需要企业ID 应用密钥
    @Override
    public void updateToken() {
        //请求的URL																 对应的ID					还有应用teoken
        final String url = "https://qyapi.weixin.qq.com/cgi-bin/gettoken?corpid=" + corpid + "&corpsecret=" + corpsecret;
        String reslut = HttpClient4.doGet(url);
        //使用json解析
        AccessTokenResponse accessTokenResponse = JSONUtil.fromJson(reslut, AccessTokenResponse.class);
        //设置过期时间
        getValue().set(EnterpriseWeChatRedisConstant.QY_WECHAT_ACCESSON_TOKEN, accessTokenResponse.getAccess_token(),7200, TimeUnit.SECONDS);
    }
//注意 但是用到调用其他模块服务的时候可能需要不生成不通的Token  对接的时候可以去企业微信里面询问他们技术客服
    //请求微信接口 拿到返回的json数据
    @Override
    public String getMessageToken() {
        String token = getValue().get(EnterpriseWeChatRedisConstant.QY_WECHAT_MESSAGE_TOKEN);
        if (null == token) {
            updateMessageToken();
            return getValue().get(EnterpriseWeChatRedisConstant.QY_WECHAT_MESSAGE_TOKEN);
        } else {
            return token;
        }
    }
//比如这个updateToken是负责部门组织架构的接口调用token信息      getMessageToken就是负责调用机器人发送消息的token
```





**企业微信对接**

## 支付宝

支付宝支付接口

支付宝办理线上会员卡接口

## 百度健康

查询订单

修改订单状态(发货)

修改商品库存接口

同步OMS库存信息 

举例

```java
	private void orderAddDeliver(MailInfoData mailInfoData) {
        //需要这个来决定请求的那个接口在内部拼上了请求地址
		Signature bceSignature = B2CAppSigner.getBceSignature(UrlConstant.ORDER_DELIVER_GOODS);
		String data = JSONObject.toJSONString(mailInfoData);
		String request = HttpClient4.doPost(bceSignature.getUrl(), data, bceSignature);
		logger.info("本地入账返回值" + request);
	}
```

请求头核心类

```java
com.zzjdyf.thirdpartydocking.util.baidu.b2c.B2CAppSigner
```

一开始写好需要载入不通的URl就直接

Signature bceSignature = B2CAppSigner.getBceSignature(“*************”);

```java
//请求列表举例
public interface UrlConstant {

    //门店列表
    String STORE_LIST = "/store/list";

    //批量添加门店
    String GOODS_ADD = "/goods/add";
}
```

## 顺丰

快递发送 （输入地址 以及寄件人信息收件人信息呼叫 

https://open.sf-express.com/#/login

记录文档,顺丰网页上的技术客服还说比较给力的,大多数问题技术客服都能帮忙解决

对接项目 代客寄药

```java
com.zzjdyf.dkyj.service.mail.impl.SFMailServiceImpl
```

对接记录大多都在一下Service服务里面



# 关于数据库常用数据查询

使用工具 **Navicat**

## 老商城常用查询 mysql

关键字段概念

vip_id 线上用户唯一ID     real_card_no   用户线下会员卡号

链接名称新数据库  数据库名称 litemall

订单表mall_bill

```sql
通过订单编号查询订单mall_bill_code 
select * from mall_bill where mall_bill_code = '010220220706235610494994'

mall_bill_status_info 订单状态子表 表作用就是用来记录订单走过的所有状态 （有时候运营需要回复订单状态或者查看订单状态记录）
select * from mall_bill_status_info  where mall_bill_code  = '010220220706235610494994'

通过订单编号把订单状态修改为待接单
update mall_bill set  bill_state='2',bill_state_desc = '待接单',
bill_child_state = '201',bill_child_state_desc = '订单已支付,等待接单!'where mall_bill_code = 010220220715213132595176

通过订单ID把订单状态改为待接单

update mall_bill set  bill_state='2',bill_state_desc = '待接单',
bill_child_state = '201',bill_child_state_desc = '订单已支付,等待接单!'where mall_bill_id = 167436


通过VIP_ID查看优惠券领取记录
select * from mall_coupon_gant_record mcgr where  vip_id = 'CCE33599E935E886E0531603A8C0C1B3';


通过用户手机号查询转盘抽奖记录

select * from vip_integral_winning_record  where vip_phone = '19939091312'



删除用户在小程序绑定记录  有时候测试需要模拟删除 

update vip_card_package_info set bind_tag = 'F' where vip_id = 'E45C85E98E84D94BE0531603A8C0EAD4' and bind_tag = 'T';
```



## 商城常用查询 orcale  （新商城老商城通用）

**Orcale EBUSER库**

```sql
通过手机查询会员ID

SELECT * FROM OMS_BASE_VIP_INFO WHERE VIP_PHONE = '15138137832';   

删除OMS会员  一下语句需要全部执行一遍 VIP)INFO_GUID就是上面那个语句看到的会员卡ID

DELETE FROM OMS_PLATFORM_ACCOUNT_INFO opai WHERE VIP_INFO_GUID = 'E6652B3BA9B844D7E0531503A8C0B543';

DELETE FROM OMS_BASE_VIP_DELIVERY_ADDRESS obvda WHERE VIP_INFO_GUID = 'E6652B3BA9B844D7E0531503A8C0B543';

DELETE FROM OMS_BASE_VIP_CARD_BAG_INFO obvcbi WHERE VIP_INFO_GUID = 'E6652B3BA9B844D7E0531503A8C0B543';

DELETE FROM OMS_USER_CARD_PACKAGE oucp WHERE USER_INFO_GUID = 'E6652B3BA9B844D7E0531503A8C0B543';

DELETE FROM OMS_BASE_VIP_INFO obvi WHERE VIP_INFO_GUID = 'E6652B3BA9B844D7E0531503A8C0B543';

通手机号查询会员卡
SELECT * FROM ADMIN.gl_hy WHERE  phonenumber = '13592068453'; 
通过会员卡号查看手机号
SELECT phonenumber FROM ADMIN.gl_hy WHERE  ID = '163154914';
通过卡号禁用用户也可以通过手机号来进行禁用 phonenumber  禁用之后代表用户线下会员卡失效不能在使用
UPDATE ADMIN.gl_hy SET FFORBID = 0 WHERE ID = '168399180';
```

### 最常用

```sql
通过会员卡号查询用户优惠券领取记录,商城可能崩溃造成优惠券领取失败需要通过这个接口来进行检查
select 
   a.guid couponGuid,a.policyno couponNo,a.policyname couponName
   ,b.membercode vipCardNo
   ,b.password password
   ,c.name vipName
   ,case when b.tel is null then c.phonenumber else b.tel end vipPhone
   ,a.auditingdate        receiveDateTime
   ,a.begindate beginDateTime
   ,a.enddate endDateTime
   ,a.couponamount couponAmount	
   ,a.amount usedLimitAmount
   ,case when nvl(b.isuse,0)=0 then '未使用' else '使用日期'||to_char(b.usedate,'yyyy-mm-dd') end usedStatus
   ,a.notes note
   from admin.memberOnlineCouponSet a 
   left join admin.MemberOnLineCouponSetdetail b on b.MEMBERONLINECOUPONSETGUID=a.guid 
   left join admin.gl_hy c on c.id=b.MEMBERCODE 
   left join admin.gl_custom g on b.GIVINGBRANCHGUID=g.tjbh
   left join admin.gl_custom h on b.USINGBRANCHGUID=h.tjbh
   where 
		b.membercode = 'N1103672'
   order by billdate DESC;
为了预警编写的手动通过手机号发送优惠券接口。。。很不好但是很好用
https://pt.zzjdyf.com/servers/vip/admin/grant/phone?phone=15737616393  通过手机号发送优惠券
```

查询用户办卡记录

```sql
--会员新版卡
SELECT
	o.guid AS 卡号,
	to_char( g.CREATEDATE, 'YYYY-MM-dd  hh24:mi:ss' ) 办卡日期,
	o.phonenumber AS 电话,
	o.ftype 平台,
	c.tjbh AS 门店编号,
	c.mc AS 门店名称 
FROM
	ADMIN.GL_HY2O2 o
	LEFT JOIN ADMIN.GL_HY g ON o.guid = g.id
	LEFT JOIN ADMIN.GL_CUSTOM c ON g.registersuborgan = c.tjbh 
WHERE
	o.ftype = 'OMS' 
	AND ( o.isbindcard = 0 OR o.isbindcard IS NULL ) 
	AND o.timestamp = g.createdate 
	AND o.timestamp BETWEEN to_date( '2022-08-03 00:00:00', 'YYYY-MM-DD hh24:mi:ss' ) 
	AND to_date( '2022-8-29 00:00:00', 'YYYY-MM-DD hh24:mi:ss' ) 
ORDER BY
	o.timestamp DESC;
可以对查询时间进行调整~
```

## 百度健康使用到的查询

```sql
可以查询到订单是否入账,以及他的快递信息 是否发快递还有快递信息
SELECT * FROM OMS_P_B2C_ORDER_MAIN_INFO WHERE  PLATFORM_ORDER_CODE = '1011656474801156480'
```

## 关于企业微信的查询

ADMIN.C_ORGDEPTRELATION 此表作用,用来绑定公司实际组织架构和企业微信组织架构里面的部门编号

表由信息部辉总(王桐辉)来进行同步的,犹豫企业组织架构经常变动,每次不是修改而是新增就需要实时查询最新的

```sql
SELECT * from ADMIN.C_ORGDEPTRELATION where DEPTCODE = '03771248'(门店编号)
查询处最新的门店记录
SELECT t.* from(SELECT * FROM  ADMIN.C_ORGDEPTRELATION where DEPTCODE = '03712021' ORDER BY UPDATETIME DESC) t WHERE rownum = 1
根据门店编号查询门店当前地理位置信息
SELECT CODE storeCode, NAME storeName, C_BDJING lng, C_BDWEI lat 
	FROM ADMIN.ORGANIZATION_FZ
	WHERE CODE = 03711680
```

