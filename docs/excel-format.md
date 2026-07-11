# Excel 格式规范

LazyConfig 对 Excel 配置表有严格的格式要求，遵循这些规范可以确保导出过程顺利进行。

## 📋 基本格式

### 三行表头约定

每张配置表必须包含三行表头：

| 行号 | 用途 | 说明 |
|------|------|------|
| **第 1 行** | 字段名 | 变量名，使用 camelCase 命名 |
| **第 2 行** | 字段类型 | 数据类型，如 `int`, `string`, `float` |
| **第 3 行** | 描述 + Schema | 字段描述和可选的 Schema 标记 |
| **第 4 行起** | 数据行 | 实际配置数据 |

### 字段名规范

- **必须**使用 `camelCase` 命名（首字母小写，后续单词首字母大写）
- **必须**包含 `id` 字段，类型为 `int`，作为主键
- 不能包含空格、特殊字符
- 不能使用 C# 关键字

### 类型行规范

- 类型名必须与支持的类型列表完全匹配（不区分大小写）
- 集合类型使用泛型语法，如 `List<int>`、`Dictionary<string,float>`
- 数组类型使用方括号，如 `int[]`、`Vector3[]`

### 描述行规范

- 描述文本在前，Schema 标记在后，用空格分隔
- 多个 Schema 标记可以组合使用，用空格分隔
- 描述可以为空，只写 Schema 标记

## 📊 完整示例

| id | name | damageId | cooldown | manaCost | effectId | classType | skillIds | tags | spawnPosition |
|----|------|----------|----------|----------|----------|-----------|----------|------|---------------|
| int | string | int | float | int | int | int | int[] | string[] | Vector3 |
| 技能ID | 技能名称 | #link\[Damage,id] 伤害类型 | #range\[0,300] 冷却时间(秒) | #range\[0,999] 魔法消耗 | #link\[SkillEffect,id] 技能效果 | #enum\[Game.ClassType] 职业类型 | 技能ID列表 | 标签列表 | 出生位置 |
| 1 | 烈焰斩 | 1 | 5.0 | 20 | 1 | 0 | [1,2,3] | [火,近战,攻击] | (0,0,0) |
| 2 | 冰霜新星 | 2 | 8.0 | 35 | 2 | 1 | [4,5] | [冰,远程,控制] | (10,0,5) |

## 📑 多 Sheet 支持

单个 Excel 文件可以包含多个工作表（Sheet），每个 Sheet 会被视为独立的配置表。

### 导出规则

| 场景 | 导出规则 | 示例 |
|------|----------|------|
| **单 Sheet** | 按文件名导出，不添加 Sheet 名称 | `Character.xlsx` → `Character.json`、`CharacterConfigs.cs` |
| **多 Sheet** | 每个 Sheet 单独导出，命名格式 `FileName_SheetName` | `GameData.xlsx`（Sheet1、Sheet2）→ `GameData_Sheet1.json`、`GameData_Sheet2.json` |

### 多 Sheet 示例

假设 `GameData.xlsx` 包含两个 Sheet：

| Sheet 名称 | 内容 | 导出文件 |
|------------|------|----------|
| `Item` | 物品配置 | `GameData_Item.json`、`GameDataItemConfigs.cs` |
| `Equipment` | 装备配置 | `GameData_Equipment.json`、`GameDataEquipmentConfigs.cs` |

### 注意事项

1. 每个 Sheet 都必须遵循三行表头约定
2. 每个 Sheet 必须有独立的 `id` 字段
3. 跨表关联可以在不同 Sheet 之间建立（使用完整的 `FileName_SheetName` 格式）

## 🔍 数据行格式

### 基础类型

| 类型 | 示例 | 说明 |
|------|------|------|
| `int` | `123` | 整数 |
| `float` | `3.14` | 浮点数 |
| `string` | `Hello World` | 字符串 |
| `bool` | `true` / `false` | 布尔值 |

### 数组类型

使用方括号 `[]` 包裹，元素用逗号分隔：

| skillIds | tags |
|----------|------|
| int[] | string[] |
| 技能ID列表 | 标签列表 |
| [1,2,3] | [火,近战,攻击] |
| [4,5] | [冰,远程,控制] |

### 列表类型

与数组格式相同：

| statGrowth |
|------------|
| List\<float> |
| 属性成长值 |
| [1.0, 1.2, 0.8, 0.9] |

### Unity 类型

| 类型 | 格式 | 示例 |
|------|------|------|
| `Vector2` | `(x, y)` | `(1.5, 2.0)` |
| `Vector3` | `(x, y, z)` | `(10.0, 0.0, 5.0)` |
| `Vector4` | `(x, y, z, w)` | `(1.0, 0.5, 0.3, 1.0)` |
| `Color` | `(r, g, b, a)` | `(1.0, 0.5, 0.0, 1.0)` |
| `Quaternion` | `(x, y, z, w)` | `(0.0, 0.0, 0.0, 1.0)` |

### 枚举类型

直接写枚举值名称或整数数值：

| classType |
|-----------|
| #enum\[Game.ClassType] int |
| 职业类型 |
| Warrior |
| 1 |

## ⚠️ 注意事项

1. **编码问题**：建议使用 UTF-8 编码保存 Excel 文件，确保中文正常显示
2. **空值处理**：空单元格会被解析为默认值（0、null、false）
3. **格式一致性**：同一列的数据格式必须一致
4. **特殊字符**：字符串中包含逗号、方括号等特殊字符时需要转义
5. **ID 唯一性**：每张表的 `id` 字段值必须唯一，导出时会自动校验

## 📝 推荐实践

### 目录结构

```
Assets/LazyConfig/
├── Excel/
│   ├── Character.xlsx      # 角色配置
│   ├── Skill.xlsx          # 技能配置
│   ├── Damage.xlsx         # 伤害类型配置
│   ├── SkillEffect.xlsx    # 技能效果配置
│   └── Level.xlsx          # 关卡配置
```

### 命名规范

- Excel 文件：`PascalCase`（如 `Character.xlsx`）
- 字段名：`camelCase`（如 `maxHp`）
- Sheet 名称：`PascalCase`（如 `Item`）

### 版本控制

建议将 Excel 文件纳入版本控制（Git），方便团队协作和历史追溯。