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

