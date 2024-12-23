---
icon: unlock-keyhole
---

# 接口认证

## 访问凭证获取

DocFlow使用[TextIn](https://www.textin.com/)账号。  

请在[TextIn](https://www.textin.com/)上注册后，在 [TextIn首页-账户与充值-账号与开发者信息 ](https://www.textin.com/console/dashboard/setting)页面获取 `x-ti-app-id` 和 `x-ti-secret-code`，用于请求认证。

## 请求认证

DocFlow接口支持两种请求认证方法：

1. 简易认证。该方法简单，但安全性有限，通常用于快速集成以体验DocFlow流程和效果。
2. 签名认证。该方法复杂，但安全性较高。可以避免访问凭证被中间人获取和篡改请求。


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

以签名方式认证请求，需要传入3个 HTTP Header:

| Header           | 说明                                    |
| ---------------- | --------------------------------------- |
| `x-ti-app-id`    | TextIn 开发者信息中获取的 `x-ti-app-id` |
| `x-ti-timestamp` | Unix Epoch 时间戳，秒                   |
| `x-ti-signature` | 请求签名，计算方法见下文。              |

#### 签名计算

签名计算方法为：

```
signature = lower(hex(HMAC_SHA256(signing_key, string_to_sign)))
```

其中：

1.  `lower()` 为字母转小写函数
2. `hex()`将字节数组转为16进制字符串
3. `HMAC_SHA256` 为密码哈希函数，可以参考各开发语言库
4. `signing_key = HMAC_SHA256(x-ti-secret-code, epoch)`。其中`x-ti-secret-code`为TextIn开发者凭证。 `epoch`为Unix Epoch时间戳(秒)。
5. `string_to_sign`，下文详细说明

##### string_to_sign

`string_to_sign` 由是以下内容拼接而成的字符串：

```
"HTTP方法" + "\n"
"请求URL" + "\n"
"排序后的URL参数" + "\n"
"sha256(HTTP请求体)"
```

其中：

1. HTTP方法为大写，例如`GET`、`POST`

2. 请求URL，URL中的路径部分(不含协议和域名)，例如`/api/app-api/sip/platform/v2/file/upload`

3. URL参数排序，是对所有请求参数按参数名的字典序（ ASCII 码）升序排序。参数值不参与排序。例如：

   假设参数是`workspace_id=12345&batch_num=54321&file_name=invoice.pdf`，
   排序后结果是`batch_num=54321&file_name=invoice.pdf&workspace_id=12345`。

   注意：

   1. 参数值在进行签名计算时，不需要 url 编码
   2. 参数以 `&` 拼接，最后没有`&`

#### 示例

{% tabs %}

{% tab title="Python" %}

{% code title="auth.py" overflow="wrap" lineNumbers="true" %}

```python
import hashlib
import hmac
import time
import json

def calculate_signature(app_id, secret_code, method, path, params, body_payload):
    # 1. 获取当前时间戳
    timestamp = int(time.time())
    
    # 2. 计算signing_key
    signing_key = hmac.new(
        secret_code.encode('utf-8'),
        str(timestamp).encode('utf-8'),
        hashlib.sha256
    ).digest()
    
    # 3. 准备string_to_sign
    # 3.1 对参数排序
    if params:
        sorted_params = "&".join(f"{k}={v}" for k, v in sorted(params.items()))
    else:
        sorted_params = ""
        
    # 3.2 计算请求体的sha256
    # 将JSON对象转换为规范的字符串格式计算sha256
    body_str = json.dumps(body_payload, separators=(',', ':'))
    body_hash = hashlib.sha256(body_str.encode('utf-8')).hexdigest()
    
    # 3.3 组装string_to_sign
    string_to_sign = f"{method}\n{path}\n{sorted_params}\n{body_hash}"
    
    # 4. 计算最终签名
    signature = hmac.new(
        signing_key,
        string_to_sign.encode('utf-8'),
        hashlib.sha256
    ).hexdigest()
    
    return {
        "x-ti-app-id": app_id,
        "x-ti-timestamp": str(timestamp),
        "x-ti-signature": signature
    }

# 使用示例
app_id = "your_app_id"
secret_code = "your_secret_code"

# 请求参数
method = "POST"
path = "/api/app-api/sip/platform/v2/file/upload"
params = {}

# 请求体payload
body_payload = {
    "workspace_id": "workspace123",
    "category": "invoice",
    "batch_number": "202412190001",
    "file_name": "invoice.pdf",
    "file_data": "base64编码的文件内容"
}

# 计算签名
headers = calculate_signature(
    app_id=app_id,
    secret_code=secret_code,
    method=method,
    path=path,
    params=params,
    body_payload=body_payload
)

print("生成的请求头：")
print(headers)
```

{% endcode %}

{% endtab %}

{% endtabs %}