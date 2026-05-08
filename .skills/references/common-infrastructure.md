# EZR OpenAPI 公共基础设施

本文件提供所有EZR OpenAPI接口对接所需的公共代码和配置。

---

## 目录

1. [代码标记规范](#代码标记规范)
2. [获取接入信息](#获取接入信息)
3. [加密工具类](#加密工具类)
4. [客户端文件](#客户端文件)
5. [配置文件](#配置文件)
6. [环境说明](#环境说明)
7. [项目结构介绍](#项目结构介绍)
8. [接口调用流程](#接口调用流程)
9. [标准请求/响应结构](#标准请求响应结构)

---

## 代码标记规范

为避免重复生成相同接口代码，生成前需扫描项目代码检查是否已存在接口标记：

- **标记格式**：在方法summary注释首行添加 `[EZR-API:编号] 接口名称`
- **标记示例**：
  ```csharp
  /// <summary>
  /// [EZR-API:1000] 上传门店信息
  /// 上传门店信息（接口编号：1000）
  /// </summary>
  ```
- **扫描范围**：`*.Bussiness` 业务层、`*.ApiModel` 实体层、`*.Utils` 工具层
- **生成规则**：若已存在该接口编号的标记，则跳过生成；否则生成代码并在summary首行添加标记

---

## 获取接入信息

联系 EZR 技术支持获取以下信息：

| 参数 | 说明 |
|------|------|
| AppId | 品牌编码 |
| Token | 访问令牌 |
| AppSystem | 系统来源标识 |
| EncryptType | 加密方式（0=SHA1，1=MD5） |

---

## 加密工具类

### 检查流程

校验项目Util层或帮助类代码中，SHACryptUtils、MD5CryptUtils工具类是否存在：
- 如果存在：直接使用
- 如果不存在：优先校验其他命名空间

**需校验的命名空间**：
- `EZR.Open.Util.Security`
- `EZP.Framework.Util.Security`

如果都不存在，创建以下文件：

### SHACryptUtils.cs

```csharp
using System;
using System.Text;

namespace EZR.OP.ThirdPartyProxy.Utils
{
    public class SHACryptUtils
    {
        /// <summary>
        /// SHA1加密
        /// </summary>
        public static string SHA1(string input, bool isUpper = true)
        {
            using (var sha1 = System.Security.Cryptography.SHA1.Create())
            {
                var bytes = Encoding.UTF8.GetBytes(input);
                var hash = sha1.ComputeHash(bytes);
                var sb = new StringBuilder();
                foreach (var b in hash)
                {
                    sb.Append(b.ToString("x2"));
                }
                return isUpper ? sb.ToString().ToUpper() : sb.ToString();
            }
        }
    }
}
```

### MD5CryptUtils.cs

```csharp
using System;
using System.Text;

namespace EZR.OP.ThirdPartyProxy.Utils
{
    public class MD5CryptUtils
    {
        /// <summary>
        /// MD5加密
        /// </summary>
        public static string MakeMD5(string input, bool isUpper = true)
        {
            using (var md5 = System.Security.Cryptography.MD5.Create())
            {
                var bytes = Encoding.UTF8.GetBytes(input);
                var hash = md5.ComputeHash(bytes);
                var sb = new StringBuilder();
                foreach (var b in hash)
                {
                    sb.Append(b.ToString("x2"));
                }
                return isUpper ? sb.ToString().ToUpper() : sb.ToString();
            }
        }
    }
}
```

---

## 客户端文件

### 检查流程

检查项目业务层中是否存在 `IEZROpenApiClient` 接口和 `EZROpenApiClient` 实现类：
- 如果存在：直接使用
- 如果不存在：根据项目实际命名空间创建

### IEZROpenApiClient.cs

```csharp
using EZP.Framework.Util.Model;
using System.Collections.Generic;

namespace EZR.OP.ThirdPartyProxy.Bussiness.OpenApi.Client
{
    public interface IEZROpenApiClient
    {
        /// <summary>
        /// 请求 OpenApi 接口通用方法
        /// </summary>
        BaseApiResInfo<TRes> PostOpenApi<TReq, TRes>(string url, TReq req, Dictionary<string, string> headers = null, bool isThrowExp = true);
    }
}
```

### EZROpenApiClient.cs

```csharp
using EZP.Framework.Util.Logging;
using EZP.Framework.Util.Model;
using EZP.Framework.Util.Serialize;
using Microsoft.Extensions.Configuration;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Net;
using System.Net.Http;

namespace EZR.OP.ThirdPartyProxy.Bussiness.OpenApi.Client
{
    public class EZROpenApiClient : IEZROpenApiClient
    {
        private static readonly ILog Log = LogManager.GetLogger(typeof(EZROpenApiClient));
        private static readonly IHttpClientFactory _clientFactory = EZP.Framework.Util.HttpClientFactory.EzpFrameworkDI.GetRequiredService<IHttpClientFactory>();
        private static readonly IConfiguration _configuration = EZP.Framework.Util.HttpClientFactory.EzpFrameworkDI.GetRequiredService<IConfiguration>();

        /// <summary>
        /// 请求 OpenApi 接口通用方法
        /// </summary>
        public BaseApiResInfo<TRes> PostOpenApi<TReq, TRes>(string url, TReq req, Dictionary<string, string> headers = null, bool isThrowExp = true)
        {
            var timestamp = DateTime.Now.ToString("yyyyMMddHHmmss");
            var baseUrl = _configuration["OpenApiOptions:BaseUrl"];
            var cfg = GetOpenApiCfg();

            // 生成签名：根据EncryptType选择加密方式
            var encryStr = $"AppId={cfg.Code}&Timestamp={timestamp}&Token={cfg.AccessToken}";
            var sign = cfg.EncryptType == 0 
                ? SHACryptUtils.SHA1(encryStr) 
                : MD5CryptUtils.MakeMD5(encryStr, true);

            // 构造表单参数
            var formParam = new Dictionary<string, string>
            {
                { "AppId", cfg.Code },
                { "AppSystem", cfg.AppSystem },
                { "Timestamp", timestamp },
                { "Sign", sign },
                { "Args", FastJsonSerializer.ToJson(req) }
            };

            var res = PostForm<TRes>(baseUrl, url, formParam, headers);

            if (res == null)
            {
                Log.BusiError($"请求异常，未返回内容");
                return new BaseApiResInfo<TRes>
                {
                    Status = false,
                    StatusCode = 500,
                    Msg = "接口未返回内容",
                    Timestamp = DateTime.Now.ToString("yyyyMMddHHmmss"),
                    Result = default(TRes)
                };
            }
            else if (res.Status)
            {
                return res;
            }
            else
            {
                if (!isThrowExp) return res;
                throw new Exception(res.Msg);
            }
        }

        private BaseApiResInfo<T> PostForm<T>(string baseUrl, string url, Dictionary<string, string> formParam, Dictionary<string, string> headers)
        {
            var encodedItems = formParam.Select(i => WebUtility.UrlEncode(i.Key) + "=" + WebUtility.UrlEncode(i.Value));
            var postContent = new StringContent(string.Join("&", encodedItems), null, "application/x-www-form-urlencoded");

            var client = _clientFactory.CreateClient(nameof(EZROpenApiClient));
            client.Timeout = new TimeSpan(0, 0, 8);

            if (client.BaseAddress == null)
                client.BaseAddress = new Uri(baseUrl);

            if (headers != null && headers.Any())
            {
                foreach (var header in headers)
                {
                    if (client.DefaultRequestHeaders.Contains(header.Key))
                        client.DefaultRequestHeaders.Remove(header.Key);
                    client.DefaultRequestHeaders.Add(header.Key, header.Value);
                }
            }

            var response = client.PostAsync(url, postContent).Result;
            string resJson = response.IsSuccessStatusCode ? response.Content.ReadAsStringAsync().Result : "";

            Log.Debug($"请求地址：{url}, 响应结果:{resJson}，响应状态：{(int)response.StatusCode}");
            return FastJsonSerializer.FromJson<BaseApiResInfo<T>>(resJson);
        }

        private OpenApiCfgInfo GetOpenApiCfg()
        {
            return new OpenApiCfgInfo
            {
                Code = _configuration["OpenApiOptions:AppId"],
                AccessToken = _configuration["OpenApiOptions:Token"],
                AppSystem = _configuration["OpenApiOptions:AppSystem"],
                EncryptType = int.TryParse(_configuration["OpenApiOptions:EncryptType"], out int encryptType) ? encryptType : 0
            };
        }
    }

    internal class OpenApiCfgInfo
    {
        public string Code { get; set; }
        public string AccessToken { get; set; }
        public string AppSystem { get; set; }
        public int EncryptType { get; set; }
    }
}
```

---

## 配置文件

检查 `appsettings.json` 和 `appsettings.Development.json` 是否包含 `OpenApiOptions` 节点。

### 配置参数

| 配置项 | 必填 | 说明 |
|--------|------|------|
| BaseUrl | 是 | 测试环境 `https://open-q1.ezrpro.com/`，生产环境 `https://open-tp.ezrpro.com/` |
| AppId | 是 | 品牌编码，由EZR分配 |
| Token | 是 | 访问令牌，由EZR分配 |
| AppSystem | 是 | 系统来源标识 |
| EncryptType | 是 | 加密方式：0=SHA1，1=MD5 |
| EnvTag | 否 | 环境标签，指定请求的容器版本 |

### 配置示例

```json
{
  "OpenApiOptions": {
    "BaseUrl": "https://open-q1.ezrpro.com/",
    "AppId": "EZP-YourBrandCode",
    "Token": "YOUR_TOKEN_HERE",
    "AppSystem": "POS",
    "EncryptType": 0,
    "EnvTag": ""
  }
}
```

---

## 环境说明

| 环境 | 地址 |
|------|------|
| 测试环境 | https://open-q1.ezrpro.com |
| 生产环境 | https://open-tp.ezrpro.com |

---

## 项目结构介绍

| 项目名称 | 描述 | 备注 |
|----------|------|------|
| `*.ApiHost、*.Site` | 接口层 | Web API宿主（入口点，.NET 5.0） |
| `*.ApiModel` | 实体层 | 数据模型、DTO（.NET Standard 2.0） |
| `*.Bussiness` | 业务层 | 业务逻辑（.NET Standard 2.0） |
| `*.Intergration` | 领域层 | 调用其他业务grpc方法 |
| `*.Repository` | 数据仓储层 | 数据库访问（.NET Standard 2.0） |
| `*.Utils` | 工具层 | 通用帮助类（.NET Standard 2.0） |
| `*.Test` | 单元测试层 | 单元测试实例 |

### 开发流程

1. **ApiModel层**：生成请求/响应Model实体
2. **Business层**：创建Service接口和实现类
3. **ApiHost层**：创建Controller（API接口不需要）
4. **自动注册**：Autofac自动装配依赖项

---

## 接口调用流程

> **重要注意**：API接口不需要新建Controller，新建业务接口调用类即可。

```
1. 检查 IEZROpenApiClient、EZROpenApiClient 是否存在，不存在则创建
2. 检查 appsettings.json 配置 OpenApiOptions 节点，缺失则添加
3. 构造业务参数（Args）
4. 从DI容器获取 EZROpenApiClient，若不存在则手动创建
5. 调用接口方法发送请求
6. 解析响应数据
```

---

## 标准请求/响应结构

### 外层请求参数

所有接口的外层请求参数结构一致：

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| AppId | string | 是 | 品牌编码，由EZR分配 |
| Sign | string | 是 | 请求签名 |
| Timestamp | string | 是 | 时间戳，格式 `yyyyMMddHHmmss` |
| AppSystem | string | 是 | 系统来源标识 |
| Args | string | 是 | 业务参数JSON字符串 |

### 标准响应格式

```json
{
    "Status": true,
    "StatusCode": "0",
    "Msg": "成功",
    "Sign": "响应签名",
    "Timestamp": "20260323143052",
    "Result": {}
}
```

| 参数名 | 类型 | 说明 |
|--------|------|------|
| Status | bool | 请求是否成功 |
| StatusCode | string | 状态码，`"0"` 表示成功 |
| Msg | string | 响应消息 |
| Sign | string | 响应签名 |
| Timestamp | string | 响应时间戳 |
| Result | object | 业务返回数据 |