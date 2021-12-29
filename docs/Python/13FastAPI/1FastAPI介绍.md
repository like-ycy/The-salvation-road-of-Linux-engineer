# FastAPI介绍



![](1FastAPI介绍.assets/logo-teal.png)

FastAPI 是一个用于构建 API 的现代、快速（高性能）的 web 框架，使用 Python 3.6+ 并基于标准的 Python 类型提示。

关键特性:

- **快速**：可与 **NodeJS** 和 **Go** 比肩的极高性能（归功于 Starlette 和 Pydantic）。[最快的 Python web 框架之一](https://fastapi.tiangolo.com/zh/#_11)。
- **高效编码**：提高功能开发速度约 200％ 至 300％。*
- **更少 bug**：减少约 40％ 的人为（开发者）导致错误。*
- **智能**：极佳的编辑器支持。处处皆可自动补全，减少调试时间。
- **简单**：设计的易于使用和学习，阅读文档的时间更短。
- **简短**：使代码重复最小化。通过不同的参数声明实现丰富功能。bug 更少。
- **健壮**：生产可用级别的代码。还有自动生成的交互式文档。
- **标准化**：基于（并完全兼容）API 的相关开放标准：[OpenAPI](https://github.com/OAI/OpenAPI-Specification) (以前被称为 Swagger) 和 [JSON Schema](https://json-schema.org/)。

## FastAPI 特性

**FastAPI** 提供了以下内容：

### 基于开放标准

- 用于创建 API 的 [**OpenAPI**](https://github.com/OAI/OpenAPI-Specification) 包含了 路径操作，请求参数，请求体，安全性等的声明。
- 使用 [**JSON Schema**](https://json-schema.org/) (因为 OpenAPI 本身就是基于 JSON Schema 的) 自动生成数据模型文档。
- 经过了缜密的研究后围绕这些标准而设计。并非狗尾续貂。
- 这也允许了在很多语言中自动**生成客户端代码**。

### 自动生成文档

交互式 API 文档以及具探索性 web 界面。因为该框架是基于 OpenAPI，所以有很多可选项，FastAPI 默认自带两个交互式 API 文档。

- [**Swagger UI**](https://github.com/swagger-api/swagger-ui)，可交互式操作，能在浏览器中直接调用和测试你的 API 。
- 另外的 API 文档：[**ReDoc**](https://github.com/Rebilly/ReDoc)

### 更主流的 Python

全部都基于标准的 **Python 3.6 类型**声明（感谢 Pydantic ）。没有新的语法需要学习。只需要标准的 Python 。

### 编辑器支持

整个框架都被设计得易于使用且直观，所有的决定都在开发之前就在多个编辑器上进行了测试，来确保最佳的开发体验。

在最近的 Python 开发者调查中，我们能看到 [被使用最多的功能是 "自动补全"](https://www.jetbrains.com/research/python-developers-survey-2017/#tools-and-features)。

整个 **FastAPI** 框架就是基于这一点的。任何地方都可以进行自动补全。

你几乎不需要经常回来看文档。

### 简洁

任何类型都有合理的**默认值**，任何和地方都有可选配置。所有的参数被微调，来满足你的需求，定义成你需要的 API。

但是默认情况下，一切都能 **“顺利工作”**。

### 验证 

- 校验大部分（甚至所有？）的 Python **数据类型**，包括：
  - JSON 对象 (`dict`).
  - JSON 数组 (`list`) 定义成员类型。
  - 字符串 (`str`) 字段，定义最小或最大长度。
  - 数字 (`int`, `float`) 有最大值和最小值， 等等。
- 校验外来类型， 比如:
  - URL.
  - Email.
  - UUID.
  - ... 及其他.

所有的校验都由完善且强大的 **Pydantic** 处理。

### 安全性及身份验证

集成了安全性和身份认证。杜绝数据库或者数据模型的渗透风险。

OpenAPI 中定义的安全模式，包括：

- HTTP 基本认证。
- **OAuth2** (也使用 **JWT tokens**)。在 [OAuth2 with JWT](https://fastapi.tiangolo.com/zh/tutorial/security/oauth2-jwt/) 查看教程。
- API 密钥，在:
  - 请求头。
  - 查询参数。
  - Cookies, 等等。

加上来自 Starlette（包括 **session cookie**）的所有安全特性。

所有的这些都是可复用的工具和组件，可以轻松与你的系统，数据仓库，关系型以及 NoSQL 数据库等等集成。

### 依赖注入

FastAPI 有一个使用非常简单，但是非常强大的**依赖注入**系统。

- 甚至依赖也可以有依赖，创建一个层级或者 **“图” 依赖**。
- 所有**自动化处理**都由框架完成。
- 所有的依赖关系都可以从请求中获取数据，并且**增加了路径操作**约束和自动文档生成。
- 即使在依赖项中被定义的*路径操作* 也会**自动验证**。
- 支持复杂的用户身份认证系统，**数据库连接**等等。
- **不依赖**数据库，前端等。 但是和它们集成很简单。

### 无限制 "插件"

或者说，导入并使用你需要的代码，而不需要它们。

任何集成都被设计得被易于使用（用依赖关系），你可以用和*路径操作*相同的结构和语法，在两行代码中为你的应用创建一个 “插件”。

### 测试 

- 100% 测试覆盖。
- 代码库 100% 类型注释。
- 用于生产应用。

## Starlette 特性

**FastAPI** 和 [**Starlette**](https://www.starlette.io/) 完全兼容 (并基于)。所以，你有的其他的 Starlette 代码也能正常工作。`FastAPI` 实际上是 `Starlette` 的一个子类。所以，如果你已经知道或者使用 Starlette，大部分的功能会以相同的方式工作。

通过 **FastAPI** 你可以获得所有 **Starlette** 的特性 (FastAPI 就像加强版的 Starlette)：

- 令人惊叹的性能。它是 [Python 可用的最快的框架之一，和 **NodeJS** 及 **Go** 相当](https://github.com/encode/starlette#performance)。
- **支持 WebSocket** 。
- **支持 GraphQL** 。
- 后台任务处理。
- Startup 和 shutdown 事件。
- 测试客户端基于 `requests`。
- **CORS**, GZip, 静态文件，流响应。
- 支持 **Session 和 Cookie** 。
- 100% 测试覆盖率。
- 代码库 100% 类型注释。

## Pydantic 特性 [¶](https://fastapi.tiangolo.com/zh/features/#pydantic)

**FastAPI** 和 [**Pydantic**](https://pydantic-docs.helpmanual.io/) 完全兼容 (并基于)。所以，你有的其他的 Pydantic 代码也能正常工作。

兼容包括基于 Pydantic 的外部库， 例如用与数据库的 ORMs, ODMs。

这也意味着在很多情况下，你可以将从请求中获得的相同对象**直接传到数据库**，因为所有的验证都是自动的。

反之亦然，在很多情况下，你也可以将从数据库中获取的对象**直接传到客户端**。

通过 **FastAPI** 你可以获得所有 **Pydantic** (FastAPI 基于 Pydantic 做了所有的数据处理)：

- **更简单**
  - 没有新的模式定义 micro-language 需要学习。
  - 如果你知道 Python types，你就知道如何使用 Pydantic。
- 和你的IDE/linter/brain适配:
  - 因为 pydantic 数据结构仅仅是你定义的类的实例；自动补全，linting，mypy 以及你的直觉应该可以和你验证的数据一起正常工作。
- 更快
  - 在 [基准测试](https://pydantic-docs.helpmanual.io/#benchmarks-tag) 中，Pydantic 比其他被测试的库都要快。
- 验证复杂结构
  - 使用分层的 Pydantic 模型，Python `typing` 的 `List` 和 `Dict` 等等。
  - 验证器使我们能够简单清楚的将复杂的数据模式定义、检查并记录为 JSON Schema。
  - 你可以拥有深度**嵌套的 JSON** 对象并对它们进行验证和注释。
- 可扩展
  - Pydantic 允许定义自定义数据类型或者你可以用验证器装饰器对被装饰的模型上的方法扩展验证。
- 100% 测试覆盖率。
