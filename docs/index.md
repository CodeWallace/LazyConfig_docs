# LazyConfig

> 商业级 Excel → JSON → Config 自动化配置工具，专为 Unity 游戏开发打造

## ✨ 核心亮点

### 🎯 专为游戏开发设计
LazyConfig 是一款成熟的配置表解决方案，从 Excel 到运行时配置的完整链路自动化，让开发者专注于游戏逻辑而非配置管理。

### 🏗️ 三层解耦架构
- **Tool 层**：纯 C# 实现，不依赖 Unity，支持 CI/CD 集成
- **Runtime 层**：Unity 运行时配置加载与访问
- **Editor 层**：可视化编辑器窗口，一键导出

### 📦 强类型代码生成
自动生成 partial class 配置类和管理类，包含 XML 文档注释，IDE 智能提示完整。

### 🔗 跨表关联系统
`#link[Table,Field]` 语法实现一对一关联，自动生成 `GetXxx()` 访问方法。

### ✅ 数据校验引擎
ID 唯一性、引用完整性、数值范围、路径校验，导出时即时发现问题。

### 🔐 多种加密方案
支持 XOR、MD5、自定义加密器，内置 AES 示例，运行时自动识别解密。

### ⚡ 异步加载
支持同步、异步、回调、并行多种加载模式，不卡主线程。

### 🔌 灵活扩展
ScriptableObject 式自定义类型解析器、校验器、加密器，扩展简单。

### 📑 多 Sheet 支持
单个 Excel 文件支持多个工作表，每个 Sheet 自动分别导出为独立的配置表。

## 🚀 快速开始

1. **安装**：Package Manager → Add package from disk → 选择 `package.json`
2. **打开窗口**：`Window → Lazy → Export`
3. **准备 Excel**：按规范编写配置表（字段名、类型、描述三行表头）
4. **导出配置**：点击 Export Selected 生成 JSON + C# 代码
5. **运行时使用**：`var config = SkillConfigs.Instance.Get(id)`

> 📦 **下载 Unity 示例工程**：[LazyConfigSamples.unitypackage](https://github.com/CodeWallace/LazyConfig_docs/raw/main/LazyConfigSamples.unitypackage)

## 📖 文档导航

- [快速开始](getting-started.md) — 安装和基本使用
- [Excel 格式规范](excel-format.md) — 配置表编写规则
- [Schema 语法](schema.md) — #link / #enum / #range / #checkpath
- [数据类型](data-types.md) — 支持的所有数据类型
- [运行时 API](runtime-api.md) — 配置访问和加载方式
- [扩展机制](extension.md) — 自定义解析器、校验器、加密器
- [使用示例](examples.md) — 实际代码示例

## 📊 技术特性

| 特性 | 说明 |
|------|------|
| Unity 支持 | 2020.3+ / 2022+ / 6.x |
| .NET 支持 | .NET Framework 4.x / .NET Standard 2.0+ |
| Excel 格式 | .xlsx / .xls / .xlsb / .csv |
| 导出产物 | JSON + C# partial class |
| 加密方式 | XOR / MD5 / Custom (AES) |
| 加载模式 | 同步 / 异步 / 回调 / 并行 |
| 代码规范 | C# 编码规范，显式访问修饰符 |

## 📄 许可证

MIT License

---

**LazyConfig** — 让配置表开发更高效、更可靠。