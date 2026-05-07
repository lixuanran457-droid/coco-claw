# COCO-CLAW 项目完善说明

## 项目概述

COCO-CLAW 是一个 AI 技能商城系统，包含三个仓库：
- **coco-claw-web**: 用户前端 (Nuxt.js 3)
- **coco-claw-admin**: 管理后台 (React + Ant Design)
- **coco-claw-api**: 后端服务 (Spring Boot 3)

---

## 新增功能

### 1. API密钥管理

#### 数据库表
```sql
-- API密钥表 (api_key)
- id: 密钥ID
- user_id: 用户ID
- name: 密钥名称
- api_key: 完整密钥
- prefix: 密钥前缀（用于显示）
- group_name: 分组名称
- status: 状态 (0-禁用, 1-启用)
- last_used_at: 最后使用时间
- note: 备注
```

#### API接口

| 接口 | 方法 | 说明 |
|------|------|------|
| `/api/apikey/create` | POST | 创建API密钥 |
| `/api/apikey/list` | GET | 获取密钥列表 |
| `/api/apikey/{id}` | DELETE | 删除密钥 |
| `/api/apikey/{id}/status` | PUT | 切换密钥状态 |

### 2. 使用记录

#### 数据库表
```sql
-- 使用记录表 (usage_record)
- id: 记录ID
- user_id: 用户ID
- api_key_id: API密钥ID
- model: 使用的模型
- input_tokens: 输入Token数
- output_tokens: 输出Token数
- total_tokens: 总Token数
- cost: 消费金额
- latency_ms: 响应延迟
- ip_address: 请求IP
- status: 状态 (0-失败, 1-成功)
```

#### API接口

| 接口 | 方法 | 说明 |
|------|------|------|
| `/api/usage/records` | GET | 获取使用记录 |
| `/api/usage/stats` | GET | 获取使用统计 |
| `/api/usage/today` | GET | 获取今日使用量 |

### 3. Token管理

#### 数据库表
```sql
-- 用户Token表 (user_token)
- balance: Token余额
- total_consumed: 累计消耗
- total_recharged: 累计充值

-- 套餐表 (package)
- name: 套餐名称
- price: 价格
- token_amount: Token数量
- duration_days: 有效期

-- 用户订阅表 (user_subscription)
- package_id: 套餐ID
- token_quota: Token配额
- token_used: 已使用
- expire_time: 过期时间
```

#### API接口

| 接口 | 方法 | 说明 |
|------|------|------|
| `/api/token/balance` | GET | 获取Token余额 |
| `/api/token/packages` | GET | 获取套餐列表 |
| `/api/token/subscription` | GET | 获取当前订阅 |
| `/api/token/subscribe` | POST | 订阅套餐 |
| `/api/token/dashboard` | GET | 获取面板数据 |

### 4. 兑换码

#### 数据库表
```sql
-- 兑换码表 (redeem_code)
- code: 兑换码
- type: 类型 (1-Token, 2-套餐)
- token_amount: Token数量
- package_days: 套餐天数
- max_use_count: 最大使用次数
- used_count: 已使用次数
```

#### API接口

| 接口 | 方法 | 说明 |
|------|------|------|
| `/api/redeem/use` | POST | 使用兑换码 |
| `/api/redeem/history` | GET | 获取兑换记录 |
| `/api/redeem/validate` | GET | 验证兑换码 |

### 5. 第三方绑定

#### 数据库表
```sql
-- 第三方绑定表 (user_binding)
- provider: 平台 (github, dingtalk, asktoken)
- provider_user_id: 第三方用户ID
- nickname: 昵称
- avatar: 头像
```

#### API接口

| 接口 | 方法 | 说明 |
|------|------|------|
| `/api/binding/list` | GET | 获取绑定列表 |
| `/api/binding/{provider}` | DELETE | 解除绑定 |

---

## 部署指南

### 1. 数据库初始化

执行数据库脚本：
```bash
cd coco-claw-api/sql

# 依次执行建表脚本
mysql -u root -p coco_claw < 01_schema.sql
mysql -u root -p coco_claw < 02_category_data.sql
mysql -u root -p coco_claw < 03_skill_data.sql
mysql -u root -p coco_claw < 04_user_data.sql
mysql -u root -p coco_claw < 05_system_config.sql
mysql -u root -p coco_claw < 06_api_tables.sql  # 新增
```

### 2. 后端部署

```bash
cd coco-claw-api

# 配置环境变量
cp .env.example .env
# 编辑 .env 配置数据库和Redis

# 打包
mvn package -DskipTests

# 运行
java -jar target/coco-claw-api.jar
```

### 3. 前端部署

#### 用户前端 (coco-claw-web)
```bash
cd coco-claw-web

# 安装依赖
npm install

# 配置环境变量
cp .env.example .env.local
# 编辑 .env.local:
#   NUXT_PUBLIC_API_BASE=http://your-api-server/api

# 开发模式
npm run dev

# 生产构建
npm run build
npm run preview
```

#### 管理后台 (coco-claw-admin)
```bash
cd coco-claw-admin

# 安装依赖
npm install

# 配置环境变量
cp .env.example .env.local
# 编辑 .env.local:
#   VITE_API_BASE_URL=http://your-api-server/api
#   VITE_MOCK_MODE=false

# 开发模式
npm run dev

# 生产构建
npm run build
```

---

## 环境配置

### 后端配置 (application.yml)

```yaml
server:
  port: 8080

spring:
  datasource:
    url: jdbc:mysql://localhost:3306/coco_claw
    username: root
    password: your_password

  redis:
    host: localhost
    port: 6379
    password: your_redis_password

nacos:
  discovery:
    server-addr: your-nacos-server:8848
```

### 前端环境变量

```bash
# coco-claw-web/.env.local
NUXT_PUBLIC_API_BASE=http://localhost:8080/api

# coco-claw-admin/.env.local
VITE_API_BASE_URL=http://localhost:8080/api
VITE_MOCK_MODE=false
```

---

## 开发说明

### 前端API调用

前端使用 `$fetch` 进行API调用，需要在请求时携带Cookie：

```typescript
// 示例：在 Vue/React 中调用 API
const response = await $fetch('/api/apikey/list', {
  method: 'GET',
  credentials: 'include' // 重要：携带Cookie
})
```

### 后端认证

后端使用JWT进行认证，Token通过Cookie传输：

```java
// 从请求中获取用户ID
String token = request.getHeader("Authorization");
if (token != null && token.startsWith("Bearer ")) {
    token = token.substring(7);
    Long userId = JwtUtil.getUserId(token);
}
```

---

## 项目结构

```
coco-claw/
├── coco-claw-web/          # 用户前端
│   ├── composables/        # API调用 composables
│   │   ├── useApiKey.ts    # API密钥
│   │   ├── useUsage.ts     # 使用记录
│   │   ├── useToken.ts     # Token管理
│   │   ├── useRedeem.ts    # 兑换码
│   │   └── useBinding.ts   # 第三方绑定
│   ├── pages/
│   │   ├── my-api.vue      # 用户控制台（已完善）
│   │   └── login.vue       # 登录页面
│   └── stores/             # 状态管理
│
├── coco-claw-admin/        # 管理后台
│   ├── src/
│   │   ├── pages/
│   │   │   ├── ApiKeyManage.tsx  # API密钥管理
│   │   │   ├── TokenManage.tsx   # Token管理
│   │   │   └── UsageManage.tsx   # 使用记录
│   │   └── services/
│   │       ├── apikey.ts        # API服务
│   │       └── token.ts         # Token服务
│   └── src/App.tsx        # 路由配置
│
└── coco-claw-api/         # 后端服务
    ├── sql/
    │   └── 06_api_tables.sql    # 新增数据库表
    └── src/main/java/com/cococlown/
        └── controller/
            ├── ApiKeyController.java   # API密钥
            ├── UsageController.java     # 使用记录
            ├── RedeemController.java   # 兑换码
            ├── TokenController.java     # Token管理
            └── BindingController.java   # 第三方绑定
```

---

## 上线检查清单

- [ ] 数据库脚本已执行
- [ ] 后端服务已启动
- [ ] Nacos配置已同步
- [ ] Redis连接正常
- [ ] 前端API地址已配置
- [ ] 跨域配置已启用
- [ ] JWT密钥已配置
- [ ] 支付接口已配置（可选）

---

## 注意事项

1. **API密钥安全**：API密钥创建后只返回一次，请提示用户妥善保管
2. **Token计费**：使用记录会自动记录，需要配置各模型的计费价格
3. **兑换码**：可批量生成，需要设置有效期和最大使用次数
4. **第三方绑定**：需要配置各平台的OAuth应用信息

---

**最后更新**: 2026-05-08
