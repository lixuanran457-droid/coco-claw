# COCO CLAW 部署指南

## 一、项目结构

```
coco-claw/
├── coco-claw-web/      # 用户前端 (Nuxt.js)
├── coco-claw-admin/    # 管理后台 (React + Vite)
└── coco-claw-api/      # 后端服务 (Spring Boot)
```

## 二、后端部署

### 1. 导入数据库
```bash
# 登录 MySQL
mysql -u root -p

# 执行建表脚本
source sql/01_schema.sql
source sql/02_category_data.sql
source sql/03_skill_data.sql
source sql/04_user_data.sql
source sql/05_system_config.sql
```

### 2. 配置环境变量
复制 `coco-claw-api/.env.example` 为 `.env`，配置以下内容：

| 变量名 | 说明 | 示例 |
|--------|------|------|
| SPRING_DATASOURCE_URL | 数据库地址 | jdbc:mysql://localhost:3306/coco_claw |
| SPRING_DATASOURCE_USERNAME | 数据库用户名 | root |
| SPRING_DATASOURCE_PASSWORD | 数据库密码 | your_password |
| ALIPAY_APP_ID | 支付宝 AppID | 2021000000000000 |
| ALIPAY_PRIVATE_KEY | 支付宝私钥 | MIIEvQIBADANB... |
| WECHATPAY_MCHID | 微信商户号 | 1234567890 |

### 3. 启动服务
```bash
cd coco-claw-api
mvn spring-boot:run
# 或打包后运行
mvn package -DskipTests
java -jar target/coco-claw-api.jar
```

## 三、前端部署

### 1. 用户前端 (coco-claw-web)
```bash
cd coco-claw-web

# 安装依赖
npm install

# 配置环境变量
cp .env.example .env.local
# 编辑 .env.local 配置 API 地址

# 开发模式
npm run dev

# 生产构建
npm run build
npm run preview
```

### 2. 管理后台 (coco-claw-admin)
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

## 四、环境配置说明

### Mock 模式
| 环境 | MOCK_MODE | 说明 |
|------|-----------|------|
| 开发 | true | 使用本地 Mock 数据，不需要后端 |
| 生产 | false | 连接真实后端 API |

### API 地址配置
| 环境 | Web 地址 | Admin 地址 |
|------|----------|------------|
| 本地开发 | localhost:3000 | localhost:5173 |
| 后端地址 | localhost:8080 | localhost:8080 |

## 五、关键接口列表

### 系统配置接口
| 接口 | 方法 | 说明 |
|------|------|------|
| /api/config/home | GET | 获取首页所有配置 |
| /api/config/banners | GET | 获取 Banner 列表 |
| /api/config/features | GET | 获取功能入口 |
| /api/config/faqs | GET | 获取 FAQ |
| /api/config/params | GET | 获取系统参数 |

### Token 管理接口
| 接口 | 方法 | 说明 |
|------|------|------|
| /api/token/packages | GET | 获取 Token 套餐 |
| /api/token/subscribe | POST | 订阅套餐 |
| /api/token/user/balance | GET | 获取用户余额 |

### 支付接口
| 接口 | 方法 | 说明 |
|------|------|------|
| /api/payment/create | POST | 创建支付订单 |
| /api/payment/status/{id} | GET | 查询支付状态 |
| /api/payment/callback/alipay | POST | 支付宝回调 |

## 六、常见问题

### 1. 后端启动失败
- 检查数据库连接是否正确
- 检查端口 8080 是否被占用
- 查看日志中的具体错误信息

### 2. 前端无法连接后端
- 检查后端是否启动
- 检查 API 地址配置是否正确
- 检查跨域配置 (后端已配置 CORS)

### 3. 支付回调失败
- 确保回调地址公网可访问
- 检查支付宝/微信后台配置的回调地址是否正确
