# 快速开始

本文将引导你从零开始使用 LazyConfig，完成配置表的编写、导出和运行时访问。

## 📦 安装

### 环境要求

- **Unity 版本**：2020.3+（推荐 2022+）
- **.NET 版本**：.NET Framework 4.x 或 .NET Standard 2.0+

### 安装步骤

1. 在 Unity 中打开 **Package Manager**（Window → Package Manager）
2. 点击 **+** → **Add package from disk**
3. 选择 `LazyConfig/package.json` 文件
4. 等待导入完成

### 验证安装

安装完成后，在 Unity 顶部菜单中会出现 **Lazy** 菜单：

```
Lazy → Lazy Export Window
```

### 下载示例工程

如果你希望直接体验完整示例，可以下载 Unity 示例包：

👉 [下载 LazyConfigSamples.unitypackage](https://github.com/CodeWallace/LazyConfig_docs/raw/main/LazyConfigSamples.unitypackage)

导入方式：Unity 菜单 → Assets → Import Package → Custom Package → 选择下载的 `.unitypackage` 文件。

## 🚀 第一步：打开配置窗口

1. 点击菜单 `Lazy → Lazy Export Window`
2. 首次打开时，点击 **Create Configs** 按钮
3. 系统会自动创建：
   - `EditorConfig.asset` — 编辑器配置
   - `RuntimeConfig.asset` — 运行时配置

## 📝 第二步：准备 Excel 配置表

创建一个 Excel 文件（如 `Skill.xlsx`），按照以下格式编写：

| 行号 | 内容 | 示例 |
|------|------|------|
| 第 1 行 | 字段名 | `id`, `name`, `damageId` |
| 第 2 行 | 字段类型 | `int`, `string`, `int` |
| 第 3 行 | 描述 + Schema | `技能ID`, `技能名称`, `#link[Damage,id] 伤害类型` |
| 第 4 行起 | 数据行 | 实际配置数据 |

### 完整示例

| id | name | damageId | cooldown | manaCost | effectId |
|----|------|----------|----------|----------|----------|
| int | string | int | float | int | int |
| 技能ID | 技能名称 | #link\[Damage,id] 伤害类型 | #range\[0,300] 冷却时间 | #range\[0,999] 魔法消耗 | #link\[SkillEffect,id] 技能效果 |
| 1 | 烈焰斩 | 1 | 5.0 | 20 | 1 |
| 2 | 冰霜新星 | 2 | 8.0 | 35 | 2 |
| 3 | 闪电链 | 3 | 12.0 | 50 | 3 |

## 📤 第三步：导出配置

在 LazyConfig 编辑器窗口中：

1. **添加目录**：点击 `+ Add Directory` 添加 Excel 文件所在目录
2. **选择文件**：勾选要导出的 Excel 文件
3. **选择导出模式**：
   - **Export Selected**：导出 JSON + C# 代码（推荐）
   - **Export JSON**：仅导出 JSON
   - **Export C#**：仅导出代码
4. **点击导出**：等待导出完成

### 导出产物

导出后会生成以下文件：

```
Runtime/Configs/
├── SkillConfig.cs          # 配置数据类（partial）
├── SkillConfigs.cs         # 配置管理类（partial）
Resources/Configs/
└── Skill.json              # 配置数据 JSON
```

## 🎮 第四步：运行时使用

### 基本用法

```csharp
using Game.Config;

// 访问单例（自动初始化加载）
var skillConfigs = SkillConfigs.Instance;

// 按 ID 查询（O(1)）
var skill = skillConfigs.Get(1);
Debug.Log(skill.name);       // 烈焰斩
Debug.Log(skill.cooldown);   // 5.0

// 跨表关联访问（自动生成）
var damage = skill.GetDamage();
Debug.Log(damage.name);      // 火焰伤害

// 按条件查找
var highDamageSkills = skillConfigs.FindAll(s => s.manaCost > 30);
```

### 异步加载

```csharp
// 异步加载（推荐）
await SkillConfigs.LoadInstanceAsync();

// 或者使用回调
SkillConfigs.Instance.LoadAsync((success, errors) =>
{
    if (success)
    {
        var skill = SkillConfigs.Instance.Get(1);
    }
});
```

## 🎯 常见问题

### Q: Excel 文件放在哪里？

A: 可以放在任意目录，在编辑器窗口中添加目录路径即可。建议放在 `Assets/Plugins/LazyConfig/Excel/` 目录下。

### Q: 生成的代码在哪里？

A: 默认生成在 `Assets/Plugins/LazyConfig/Runtime/Configs/` 目录下。

### Q: JSON 文件在哪里？

A: 默认生成在 `Assets/Resources/Configs/` 目录下，运行时通过 Resources.Load 加载。

### Q: 如何修改导出路径？

A: 在 `EditorConfig.asset` 中修改 `Json Export Path` 和 `Code Export Path`。

### Q: 每张表必须有 id 字段吗？

A: 是的，每张表必须有一个名为 `id` 的 int 类型字段，作为主键。

## 📚 下一步

- [Excel 格式规范](excel-format.md) — 了解完整的配置表格式
- [Schema 语法](schema.md) — 学习 #link / #enum / #range / #checkpath
- [数据类型](data-types.md) — 查看支持的所有数据类型