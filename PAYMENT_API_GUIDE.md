# COCO CLAW 支付与技能交付接口文档

## 更新日期: 2026-04-09

---

## 一、支付SDK已集成

### 1.1 已添加的SDK依赖

```xml
<!-- 支付宝SDK -->
<dependency>
    <groupId>com.alipay.sdk</groupId>
    <artifactId>alipay-easysdk</artifactId>
    <version>2.2.0</version>
</dependency>

<!-- 微信支付SDK -->
<dependency>
    <groupId>com.github.wechatpay-apiv3</groupId>
    <artifactId>wechatpay-java</artifactId>
    <version>1.4.4</version>
</dependency>
```

### 1.2 配置文件

在 `application.yml` 中添加了支付配置：

```yaml
payment:
  sandbox:
    enabled: true  # 沙箱环境开关

  alipay:
    app-id: ${ALIPAY_APP_ID:}
    private-key: ${ALIPAY_PRIVATE_KEY:}
    alipay-public-key: ${ALIPAY_PUBLIC_KEY:}
    notify-url: ${ALIPAY_NOTIFY_URL:}
    return-url: ${ALIPAY_RETURN_URL:}
    gateway: https://openapi-sandbox.dl.alipaydev.com/gateway.do  # 沙箱网关

  wechatpay:
    mchid: ${WECHATPAY_MCHID:}
    serial-no: ${WECHATPAY_SERIAL_NO:}
    private-key-path: classpath:cert/apiclient_key.pem
    notify-url: ${WECHATPAY_NOTIFY_URL:}
    api-key: ${WECHATPAY_API_KEY:}
    appid: ${WECHATPAY_APPID:}
```

---

## 二、支付流程

```
1. 前端调用 POST /api/payment/create
2. 后端创建Payment记录，调用支付SDK生成支付参数
3. 前端根据payParams.type渲染支付界面
   - type=html: 支付宝表单，自动提交
   - type=qrcode: 微信二维码，展示给用户扫码
4. 用户完成支付
5. 第三方支付回调 POST /api/payment/callback/{channel}
6. 后端验证签名，更新订单状态
7. 后端自动调用deliverSkillToUser()交付技能
```

---

## 三、已实现的支付服务

### 3.1 支付宝服务 (AlipayServiceImpl.java)

```java
// WAP支付（手机网站支付）
String createWapPay(Order order, Payment payment);

// PC支付（电脑网站支付）
String createPagePay(Order order, Payment payment);

// 查询交易状态
String queryTradeStatus(String orderNo);

// 关闭交易
boolean closeTrade(String orderNo);

// 申请退款
boolean refund(Order order, BigDecimal refundAmount, String reason);
```

### 3.2 微信支付服务 (WechatpayServiceImpl.java)

```java
// Native支付（二维码支付）
String createNativePay(Order order, Payment payment);

// 查询交易状态
String queryTradeStatus(String orderNo);

// 关闭交易
boolean closeTrade(String orderNo);

// 申请退款
boolean refund(Order order, BigDecimal refundAmount, String reason);
```

---

## 四、技能交付机制

### 4.1 交付流程

```
用户支付成功 → handlePaymentCallback() → deliverSkillToUser() → user_skill表
```

### 4.2 交付逻辑 (PaymentServiceImpl.java)

```java
private void deliverSkillToUser(Order order, Long userId, String email, Skill skill) {
    // 1. 确定用户标识（userId优先，email次之）
    // 2. 检查是否已有该技能（避免重复）
    // 3. 如果已有，增加使用次数
    // 4. 如果没有，创建UserSkill记录
    // 5. 复制skill.apiKey到user_skill表
    // 6. 设置默认有效期1年
}
```

### 4.3 用户技能表 (user_skill)

| 字段 | 类型 | 说明 |
|------|------|------|
| id | BIGINT | 主键 |
| user_id | BIGINT | 用户ID |
| email | VARCHAR | 邮箱（游客） |
| skill_id | BIGINT | 技能ID |
| skill_name | VARCHAR | 技能名称 |
| skill_api_key | VARCHAR | API密钥 |
| usage_count | INT | 已使用次数 |
| max_usage_count | INT | 最大使用次数（0=无限） |
| expire_time | DATETIME | 过期时间 |
| status | TINYINT | 状态 |

### 4.4 数据库脚本

执行 `coco-claw-api/src/main/resources/sql/user_skill.sql` 创建相关表。

---

## 五、用户技能API

### 5.1 获取用户技能列表

```
GET /api/user/skill/list
Header: X-User-Id: 123
Query: email=user@example.com (可选，游客查询)

Response:
{
  "code": 200,
  "data": [{
    "id": 1,
    "skillId": 10,
    "skillName": "ChatGPT-4",
    "usageCount": 5,
    "maxUsageCount": 100,
    "remainUsage": 95,
    "unlimited": false,
    "expireTime": "2027-04-09",
    "isExpired": false,
    "apiKey": "sk-xxx..."
  }]
}
```

### 5.2 获取技能详情

```
GET /api/user/skill/{skillId}
Header: X-User-Id: 123
Query: email=user@example.com

Response:
{
  "code": 200,
  "data": {
    "skillId": 10,
    "skillName": "ChatGPT-4",
    "skillDescription": "...",
    "usageCount": 5,
    "remainUsage": 95,
    "apiKey": "sk-xxx...",
    "apiEndpoint": "https://api.openai.com/v1/chat/completions",
    "apiDocumentation": "https://..."
  }
}
```

### 5.3 使用技能

```
POST /api/user/skill/use/{skillId}
Header: X-User-Id: 123

Response:
{
  "code": 200,
  "data": {
    "usageCount": 6,
    "remainUsage": 94,
    "apiKey": "sk-xxx..."
  }
}
```

### 5.4 验证技能权限

```
GET /api/user/skill/verify/{skillId}
Header: X-User-Id: 123

Response:
{
  "code": 200,
  "data": true/false
}
```

---

## 六、环境配置

### 6.1 沙箱环境（测试）

```bash
# 环境变量
ALIPAY_APP_ID=2021000000000000  # 沙箱APPID
ALIPAY_PRIVATE_KEY=...           # 沙箱私钥
ALIPAY_PUBLIC_KEY=...            # 沙箱公钥

WECHATPAY_MCHID=1234567890       # 沙箱商户号
WECHATPAY_API_KEY=...           # 沙箱API密钥
WECHATPAY_APPID=...             # 沙箱APPID

# application.yml
payment:
  sandbox:
    enabled: true
  alipay:
    gateway: https://openapi-sandbox.dl.alipaydev.com/gateway.do
```

### 6.2 生产环境

```bash
# 环境变量
ALIPAY_APP_ID=正式APPID
ALIPAY_PRIVATE_KEY=正式私钥
ALIPAY_PUBLIC_KEY=正式公钥

WECHATPAY_MCHID=正式商户号
WECHATPAY_API_KEY=正式API密钥
WECHATPAY_APPID=正式APPID

# application.yml
payment:
  sandbox:
    enabled: false
  alipay:
    gateway: https://openapi.alipay.com/gateway.do
```

---

## 七、支付状态码

| 状态码 | 订单状态 | 说明 |
|--------|----------|------|
| 0 | 待支付 | 等待用户支付 |
| 1 | 支付中 | 支付处理中 |
| 2 | 已支付 | 支付成功 |
| 3 | 支付关闭 | 超时或用户取消 |
| 4 | 退款中 | 申请退款中 |
| 5 | 已退款 | 退款完成 |
| 6 | 已取消 | 订单取消 |

---

## 八、注意事项

1. **签名验证**: 所有支付回调必须验证签名
2. **幂等性**: 回调处理需考虑重复通知
3. **日志记录**: 所有支付操作必须记录日志
4. **异常处理**: 支付失败需及时告警
5. **技能交付**: 支付成功后才调用deliverSkillToUser
6. **退款检查**: 已使用的技能不允许退款

---

## 九、沙箱测试

- 支付宝沙箱: https://openhome.alipay.com/develop/sandbox
- 微信支付沙箱: 在微信商户平台开启

---

## 十、联系支持

如有问题，请联系后端团队。
