---
name: ezr-openapi-creator
description: 一键生成EZR OpenAPI接口对接代码。当用户提到"生成接口代码"、"对接EZR接口"、"生成OpenAPI代码"、"EZR接口对接"、"一键生成接口"、"对接1000接口"、"对接会员接口"、"对接门店接口"、"对接积分接口"、"对接券接口"、"对接订单接口"时必须使用此skill。支持单接口模式（输入接口编号）和场景模式（选择业务场景，批量生成多个接口代码）。适用于.NET 5项目。
---

# EZR OpenAPI 接口代码生成器

一键生成EZR OpenAPI接口对接代码，支持单接口和场景两种模式。

---

## 工作流程

### 第一步：询问用户选择模式

使用 `AskUserQuestion` 工具询问用户选择生成模式：

| 模式 | 说明 |
|------|------|
| **单接口模式** | 输入接口编号，根据对应接口文档生成单个接口的完整代码 |
| **场景模式** | 选择业务场景（支持多选），批量生成该场景下所有接口的代码 |

### 第二步：根据模式执行

#### 单接口模式流程

1. **获取接口编号** - 用户输入接口编号（如 1000、1203、1302）
2. **检查公共基础设施** - 执行公共基础设施检查流程（见下方）
3. **查找并读取接口文档** - 在 `references/api/` 或 `references/callback/` 目录下查找对应文档
4. **解析接口文档** - 提取接口基础信息、业务参数、响应结构、示例代码
5. **生成完整代码** - 包含业务服务类、请求模型，添加 `// [EZR-API:编号]` 标记
6. **编译验证** - 执行 `dotnet build` 编译项目，如有异常则修复后重新编译

#### 场景模式流程

1. **输出完整场景列表** - 直接输出以下场景列表表格，等待用户输入选择的场景序号或名称（可多选，如输入"1,4,6"或"会员开卡场景,积分场景,优惠券场景"）：

| 序号 | 场景名称 | API接口 | 回调接口 | 说明 |
|------|----------|---------|----------|------|
| 1 | 会员开卡场景 | 1190, 1201, 1202, 1203 | 1210, 12105 | 会员注册、线下开卡、修改会员、会员查询 |
| 2 | 会员扩展场景 | 1225, 1226, 1227, 1228, 1229, 1230 | - | 标签管理、扩展属性、消费习惯 |
| 3 | 等级场景 | 1207 | 1214 | 上传等级变动、等级变动推送 |
| 4 | 积分场景 | 1188, 1204, 1206, 1224, 1232, 1233, 1234, 1244 | 1211 | 积分查询、流水上传、扣减、锁定/解锁/消费 |
| 5 | 订单场景 | 1247, 1220 | - | 会员交易数据分页、会员销售数据上传 |
| 6 | 优惠券场景 | 1300, 1302, 1307, 1308, 1309, 1311, 1313, 1323 | 12131, 1213, 1217, 1219, 1310, 1312 | 券核销、券库管理、券发放 |

2. **解析用户选择** - 根据用户输入的序号或场景名称，确定需要生成的场景列表
3. **检查公共基础设施** - 执行一次公共基础设施检查流程
4. **批量生成代码** - 为每个接口读取文档并生成代码
5. **整合输出** - 整合所有接口代码到输出文件
6. **编译验证** - 执行 `dotnet build` 编译项目，如有异常则修复后重新编译

---

## 接口索引表

### 常用接口快速索引

| 编号 | 接口名称 | 类型 | 文档路径 |
|------|----------|------|----------|
| 1190 | 会员注册 | API | `references/api/1190会员注册接口.md` |
| 1201 | 会员线下开卡 | API | `references/api/1201会员线下开卡接口.md` |
| 1203 | 会员查询 | API | `references/api/1203会员查询接口.md` |
| 1204 | 积分查询 | API | `references/api/1204积分查询接口.md` |
| 1206 | 积分流水上传 | API | `references/api/1206积分流水上传接口.md` |
| 1302 | 在线核销券 | API | `references/api/1302在线核销券接口.md` |
| 1307 | 会员券获取 | API | `references/api/1307会员券获取接口.md` |
| 1309 | 新增券库 | API | `references/api/1309新增券库接口.md` |
| 1210 | 会员开卡推送 | 回调 | `references/callback/1210会员开卡推送.md` |
| 1211 | 积分变动推送 | 回调 | `references/callback/1211积分变动推送.md` |
| 1213 | 优惠券发放推送 | 回调 | `references/callback/1213优惠券发放推送.md` |

---

## 对接场景清单

### 场景列表

| 场景名称 | API接口 | 回调接口 | 说明 |
|----------|---------|----------|------|
| **会员开卡场景** | 1190, 1201, 1202, 1203 | 1210, 12105 | 会员注册、线下开卡、修改会员、会员查询、会员开卡推送、会员注销推送 |
| **会员扩展场景** | 1225, 1226, 1227, 1228, 1229, 1230 | - | 添加修改标签、修改扩展属性、获取标签、扩展信息查询、批量操作标签、消费习惯 |
| **等级场景** | 1207 | 1214 | 上传等级变动、等级变动推送 |
| **积分场景** | 1188, 1204, 1206, 1224, 1232, 1233, 1234, 1244 | 1211 | 积分批量更新、积分查询、积分流水上传、在线扣减积分、积分锁定/解锁/消费、积分分页查询、积分变动推送 |
| **订单场景** | 1247, 1220 | - | 会员交易数据分页、会员销售数据上传 |
| **优惠券场景** | 1300, 1302, 1307, 1308, 1309, 1311, 1313, 1323 | 12131, 1213, 1217, 1219, 1310, 1312 | 券核销相关、券库管理、券发放、券推送回调 |

### 各场景接口详情

#### 会员开卡场景

| 编号 | 接口名称 | 类型 | 文档路径 |
|------|----------|------|----------|
| 1190 | 会员注册 | API | `references/api/1190会员注册接口.md` |
| 1201 | 会员线下开卡 | API | `references/api/1201会员线下开卡接口.md` |
| 1202 | 修改会员信息 | API | `references/api/1202修改会员信息.md` |
| 1203 | 会员查询 | API | `references/api/1203会员查询接口.md` |
| 1210 | 会员开卡推送 | 回调 | `references/callback/1210会员开卡推送.md` |
| 12105 | 会员注销推送 | 回调 | `references/callback/12105会员注销推送.md` |

#### 会员扩展场景

| 编号 | 接口名称 | 类型 | 文档路径 |
|------|----------|------|----------|
| 1225 | 添加修改会员标签 | API | `references/api/1225添加修改会员标签接口.md` |
| 1226 | 修改会员扩展属性 | API | `references/api/1226修改会员扩展属性.md` |
| 1227 | 获取会员标签信息 | API | `references/api/1227获取会员标签信息.md` |
| 1228 | 会员扩展信息查询 | API | `references/api/1228会员扩展信息查询.md.md` |
| 1229 | 批量操作会员标签 | API | `references/api/1229批量操作会员标签接口.md` |
| 1230 | 会员消费习惯 | API | `references/api/1230会员消费习惯接口.md` |

#### 等级场景

| 编号 | 接口名称 | 类型 | 文档路径 |
|------|----------|------|----------|
| 1207 | 上传等级变动 | API | `references/api/1207上传等级变动.md` |
| 1214 | 等级变动推送 | 回调 | `references/callback/1214等级变动推送.md` |

#### 积分场景

| 编号 | 接口名称 | 类型 | 文档路径 |
|------|----------|------|----------|
| 1188 | 会员最终积分批量更新 | API | `references/api/1188会员的最终积分批量更新接口.md` |
| 1204 | 积分查询 | API | `references/api/1204积分查询接口.md` |
| 1206 | 积分流水上传 | API | `references/api/1206积分流水上传接口.md` |
| 1224 | 在线扣减积分 | API | `references/api/1224在线扣减积分接口.md` |
| 1232 | 积分锁定 | API | `references/api/1232积分锁定接口.md` |
| 1233 | 积分解锁 | API | `references/api/1233积分解锁接口.md` |
| 1234 | 消费已锁定积分 | API | `references/api/1234消费已锁定积分接口.md` |
| 1244 | 查询会员积分分页 | API | `references/api/1244查询会员积分分页接口.md` |
| 1211 | 积分变动推送 | 回调 | `references/callback/1211积分变动推送.md` |

#### 订单场景

| 编号 | 接口名称 | 类型 | 文档路径 |
|------|----------|------|----------|
| 1247 | 会员交易数据分页 | API | `references/api/1247会员交易数据分页接口.md` |
| 1220 | 会员销售数据上传 | API | `references/api/1220会员销售数据上传接口.md` |

#### 优惠券场景

| 编号 | 接口名称 | 类型 | 文档路径 |
|------|----------|------|----------|
| 1300 | 上传券核销信息 | API | `references/api/1300上传券核销信息接口.md` |
| 1302 | 在线核销券 | API | `references/api/1302在线核销券接口.md` |
| 1307 | 会员券获取 | API | `references/api/1307会员券获取接口.md` |
| 1308 | 券核销取消 | API | `references/api/1308券核销取消接口.md` |
| 1309 | 新增券库 | API | `references/api/1309新增券库接口.md` |
| 1311 | 券库查询 | API | `references/api/1311券库查询接口.md` |
| 1313 | 第三方批量发券 | API | `references/api/1313第三方批量发券接口.md` |
| 1323 | 拉粉发券 | API | `references/api/1323拉粉发券接口.md` |
| 12131 | 券批量推送 | 回调 | `references/callback/12131券批量推送.md` |
| 1213 | 优惠券发放推送 | 回调 | `references/callback/1213优惠券发放推送.md` |
| 1217 | 券核销状态回调 | 回调 | `references/callback/1217券核销状态回调.md` |
| 1219 | 券转增推送 | 回调 | `references/callback/1219券转增推送.md` |
| 1310 | 券库信息推送 | 回调 | `references/callback/1310券库信息推送.md` |
| 1312 | 券库禁用推送 | 回调 | `references/callback/1312券库禁用推送.md` |

---

## 文档路径规则

接口文档位于 `references` 目录下：

- **API接口**：`references/api/{编号}{名称}.md`
- **回调接口**：`references/callback/{编号}{名称}.md`
- **公共基础设施**：`references/common-infrastructure.md`

---

## 代码生成规范

### 分层规范（重要！必须严格遵守）

> **⚠️ 严禁将模型类放在 Bussiness 层**，必须严格遵循项目分层架构：

| 层级 | 命名空间 | 说明 |
|------|----------|------|
| **Model层** | `{项目前缀}.Model.Dto.OpenApi` | 所有请求/响应模型、回调参数实体 |
| **Bussiness层** | `{项目前缀}.Bussiness.OpenApi` | 业务服务类（Service/Manager），只含业务逻辑 |

**错误做法**：将模型类放在 `Bussiness/OpenApi/` 下
**正确做法**：模型类放在 `Model/Dto/OpenApi/` 下，Bussiness 层通过 `using {项目前缀}.Model.Dto.OpenApi;` 引用

### 文件结构

生成的代码应包含以下文件：

| 文件类型 | 放置层级 | 命名规则 | 说明 |
|----------|----------|----------|------|
| 请求/响应模型 | **Model层** | `VipModels.cs`、`VipCallbackModels.cs` | 业务请求/响应参数实体，集中存放在 `Model/Dto/OpenApi/` |
| 业务服务类 | **Bussiness层** | `{场景名}Service.cs` | 同一场景的所有接口方法集中在一个Service文件中 |
| 回调Manager类 | **Bussiness层** | `VipCallbackManager.cs` | 回调推送业务处理逻辑 |
| 回调Controller | **ApiHost层** | `CVipController.cs` | 回调接口路由（严格按文档示例命名） |

### 场景与服务命名映射

同一场景的接口代码集中在一个Service文件中：

| 场景名称 | Service命名 | 包含接口 |
|----------|-------------|----------|
| 会员开卡场景 | `VipService.cs` | 1190, 1201, 1202, 1203 |
| 会员扩展场景 | `VipExtendService.cs` | 1225, 1226, 1227, 1228, 1229, 1230 |
| 等级场景 | `VipGradeService.cs` | 1207 |
| 积分场景 | `VipBonusService.cs` | 1188, 1204, 1206, 1224, 1232, 1233, 1234, 1244 |
| 订单场景 | `VipOrderService.cs` | 1247, 1220 |
| 优惠券场景 | `CouponService.cs` | 1300, 1302, 1307, 1308, 1309, 1311, 1313, 1323 |

### 回调接口 Controller 命名规范

> **重要**：回调接口的 Controller 命名和路由必须**严格按照具体接口文档中的示例代码**生成，不得自行命名。

生成回调接口代码时，必须：
1. **读取对应的接口文档** - 在 `references/callback/` 目录下找到具体接口文档
2. **严格按照文档中的示例代码** - Controller 名称、路由、继承基类、方法名必须完全一致
3. **不得自行修改** - 即使项目中有其他 Controller 使用不同模式，也必须按接口文档示例生成

常见回调接口 Controller 命名示例（仅供参考，实际以接口文档为准）：

| 接口类型 | Controller命名 | 路由前缀 | 参考文档 |
|----------|----------------|----------|----------|
| 会员类回调 | `CVipController` | `/api/CVip/` | `1210会员开卡推送.md` |
| 优惠券类回调 | `CCoupController` | `/api/CCoup/` | `1213优惠券发放推送.md` |

### 代码标记

每个接口方法必须在 `<summary>` 注释内首行添加标记，防止重复生成：

```csharp
/// <summary>
/// [EZR-API:编号] 接口名称
/// 接口详细描述
/// </summary>
```

> **重要**：标记 `[EZR-API:编号]` 必须放在 `<summary>` 内部的第一行，不是单独的注释行。

### 服务类模板（场景模式）

### 服务类模板

> **⚠️ 必须使用 `AutofacExt.Resolve<IEZROpenApiClient>()` 获取客户端**，不得使用示例模板中的 `EzpFrameworkDI.GetRequiredService`（这是另一个项目的命名空间）。

```csharp
using System.Collections.Generic;
using EZP.Framework.Util.Model;
using {项目前缀}.Model.Dto.OpenApi;
using {项目前缀}.Utils;

namespace {项目前缀}.Bussiness.OpenApi
{
    /// <summary>
    /// {场景名称}服务（接口编号：{编号列表}）
    /// </summary>
    public static class {场景名}Service
    {
        private static readonly IEZROpenApiClient _openApiClient = AutofacExt.Resolve<IEZROpenApiClient>();

        /// <summary>
        /// [EZR-API:编号1] 接口名称1
        /// {方法描述}
        /// </summary>
        public static BaseApiResInfo<{返回类型}> {方法名1}({请求类型1} request)
        {
            // 参数验证逻辑
            return _openApiClient.PostOpenApi<{请求类型1}, {返回类型}>("{接口地址1}", request);
        }

        /// <summary>
        /// [EZR-API:编号2] 接口名称2
        /// {方法描述}
        /// </summary>
        public static BaseApiResInfo<{返回类型}> {方法名2}({请求类型2} request)
        {
            return _openApiClient.PostOpenApi<{请求类型2}, {返回类型}>("{接口地址2}", request);
        }
    }
}
```

### 模型文件模板（必须放在 Model 层）

```csharp
using System.Collections.Generic;

namespace {项目前缀}.Model.Dto.OpenApi
{
    // ========== {接口编号1} {接口名称1} ==========

    /// <summary>
    /// [EZR-API:{接口编号1}] {接口名称1}请求参数
    /// </summary>
    public class {请求名1}Request
    {
        /// <summary>
        /// {参数描述}
        /// </summary>
        public {类型} {参数名} { get; set; }
    }

    /// <summary>
    /// [EZR-API:{接口编号1}] {接口名称1}响应结果
    /// </summary>
    public class {响应名1}Result
    {
        /// <summary>
        /// {字段描述}
        /// </summary>
        public {类型} {字段名} { get; set; }
    }
}
```

---

## 公共基础设施检查

> **重要**：生成接口代码前必须执行此检查流程。只有当检测到文件不存在时，才读取 `references/common-infrastructure.md` 获取具体代码模板。

### 检查流程（按顺序执行）

#### 1. 代码标记检查

扫描项目代码检查是否已存在接口标记，避免重复生成：
- **标记格式**：在 `<summary>` 注释内首行添加 `[EZR-API:编号] 接口名称`
- **标记示例**：
  ```csharp
  /// <summary>
  /// [EZR-API:1000] 上传门店信息
  /// 上传门店信息（接口编号：1000）
  /// </summary>
  ```
- **扫描范围**：`*.Bussiness` 业务层、`*.ApiModel` 实体层、`*.Utils` 工具层、`*.ApiHost` 接口层
- **规则**：若已存在该接口编号的标记，则跳过生成

#### 2. 加密工具类检查

检查项目Util层是否存在以下工具类：
- `SHACryptUtils` - SHA1加密工具
- `MD5CryptUtils` - MD5加密工具
- **校验命名空间**：`EZR.Open.Util.Security`、`EZP.Framework.Util.Security`

若不存在，读取 `references/common-infrastructure.md` 获取代码模板并创建。

#### 3. 客户端文件检查

检查项目业务层是否存在：
- `IEZROpenApiClient` 接口
- `EZROpenApiClient` 实现类

若不存在，读取 `references/common-infrastructure.md` 获取代码模板并创建。

#### 4. 配置文件检查

检查以下配置文件是否包含 `OpenApiOptions` 节点及必要参数：
- `appsettings.json`
- `appsettings.Development.json`

> **重要**：配置节点必须**直接拷贝** `references/common-infrastructure.md` 文件中"配置示例"章节的完整 JSON 片段，不得自行修改配置值或使用占位符。

若不存在或参数缺失，从 `references/common-infrastructure.md` 拷贝以下配置示例：

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

#### 配置要点说明

| 要点 | 说明 | ⛔ 禁止 |
|------|------|---------|
| 配置位置 | 在 `appsettings.json` 和 `appsettings.Development.json` 中添加 | ❌ 不得改为Apollo、Consul等其他配置中心 |
| 配置格式 | **直接拷贝** `common-infrastructure.md` 中的完整 JSON 示例 | ❌ 不得自行修改配置结构或使用占位符 |
| 文档示例 | 必须按文档示例添加 | ❌ 文档示例 > 项目现有风格 |

---

## 使用示例

### 单接口模式示例

**用户输入**：
```
对接1203接口
```

**输出**：
- 读取 `references/api/1203会员查询接口.md`
- 判断所属场景：会员开卡场景
- 生成/追加到 `VipService.cs`（添加会员查询方法）
- 生成/追加到 `VipModels.cs`（添加请求/响应模型）

### 场景模式示例

**用户选择**：
```
会员开卡场景、积分场景
```

**输出**：
- **会员开卡场景**：生成 `VipService.cs`（包含1190, 1201, 1202, 1203四个方法）
- **积分场景**：生成 `VipBonusService.cs`（包含1188, 1204, 1206, 1224, 1232, 1233, 1234, 1244八个方法）
- **回调处理**：若选择回调接口，生成 `CallbackService.cs`（包含对应回调方法）
- 模型文件：`VipModels.cs`、`VipBonusModels.cs`

---

## 输出格式

生成完成后，输出以下信息：

1. **生成的文件清单**：按场景列出Service文件和Models文件
2. **代码使用示例**：每个场景的调用示例代码
3. **配置提醒**：提醒检查公共基础设施和配置文件

---

## 编译验证流程

> **重要**：代码生成完成后必须执行编译验证，确保生成的代码能够正常编译。

### 执行步骤

1. **执行编译命令** - 在项目根目录执行：
   ```bash
   dotnet build
   ```

2. **检查编译结果** - 查看编译输出：
   - ✅ **编译成功**：输出 "Build succeeded"，继续后续流程
   - ❌ **编译失败**：分析错误信息，修复问题

3. **处理编译异常** - 当编译失败时：
   - **读取错误信息**：分析编译输出的错误详情
   - **定位问题文件**：根据错误信息找到对应文件和行号
   - **修复问题**：
     - 缺少命名空间引用 → 添加 `using` 语句
     - 类型不匹配 → 修正类型定义
     - 缺少依赖 → 添加必要的项目引用或包
     - 语法错误 → 修正代码语法
   - **重新编译**：修复后再次执行 `dotnet build` 验证

### 常见编译问题及解决方案

| 问题类型 | 错误示例 | 解决方案 |
|---------|---------|---------|
| 缺少using | `The type 'BaseApiResInfo' could not be found` | 添加 `using EZP.Framework.Util.Model;` |
| 缺少List | `The type 'List<T>' could not be found` | 添加 `using System.Collections.Generic;` |
| 命名空间错误 | `The namespace '...' could not be found` | 检查并修正命名空间路径 |
| 类型冲突 | `Ambiguous reference` | 明确指定完整命名空间 |
| 缺少项目引用 | `Project reference not found` | 添加项目引用或检查csproj文件 |

### 编译成功后的输出

编译成功后，输出：
```
✅ 编译验证通过，生成的代码已成功编译
```

---

## 开始生成

请选择生成模式：

| 模式 | 说明 |
|------|------|
| **单接口模式** | 输入接口编号（如：1000, 1203, 1302） |
| **场景模式** | 选择业务场景，批量生成多个接口 |