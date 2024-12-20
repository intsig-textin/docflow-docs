---
icon: unlock-keyhole
---

# 接口认证

## 访问凭证获取

DocFlow使用[TextIn](https://www.textin.com/)账号。  

请在[TextIn](https://www.textin.com/)上注册后，在 [TextIn首页-账户与充值-账号与开发者信息 ](https://www.textin.com/console/dashboard/setting)页面获取 `x-ti-app-id` 和 `x-ti-secret-code`，用于请求认证。

## 请求认证

DocFlow接口支持两种请求认证方法：

1. 简易认证。该方法简单，但不安全，通常用于快速集成以体验DocFlow流程和效果。
2. 签名认证。该方法复杂，但相对安全。可以避免访问凭证被中间人获取和篡改请求。


### 简易认证

以 `x-ti-app-id`  和 `x-ti-secret-code` 作为 HTTP 头来认证。

以curl为例：

```bash
curl \
  -H "x-ti-app-id: your_app-id" \
  -H "x-ti-secret-code: your_secret_code" \
  "https://docflow.textin.com/api/app-api/sip/platform/v2/file/upload"
```

### 签名认证

TBA

