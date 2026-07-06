# Schema 语法

Schema 标记写在 Excel 第三行（描述行），用于增强配置表的表达能力和数据校验。

## 📋 标记清单

| 标记 | 语法 | 说明 |
|------|------|------|
| `#link` | `#link[Table,Field]` | 跨表关联，自动生成 `GetXxx()` 访问方法 |
| `#ignore` | `#ignore` | 忽略该列，不生成代码和 JSON |
| `#enum` | `#enum[EnumType]` | 声明枚举类型，支持完整命名空间 |
| `#range` | `#range[min,max]` | 数值范围校验（int / float） |
| `#checkpath` | `#checkpath` | 路径存在性校验（支持自动补后缀、Resources 路径） |

## 🔗 #link — 跨表关联

### 语法

```
#link[TableName,FieldName] 描述文本
```

### 说明

建立当前表与目标表之间的关联关系，导出时会自动生成 `GetXxx()` 访问方法。

### 示例

**Skill.xlsx**：

| id | name | damageId |
|----|------|----------|
| int | string | int |
| 技能ID | 技能名称 | #link\[Damage,id] 伤害类型 |
| 1 | 烈焰斩 | 1 |

**Damage.xlsx**：

| id | name | elementType |
|----|------|-------------|
| int | string | int |
| ID | 名称 | 元素类型 |
| 1 | 火焰伤害 | 0 |

### 自动生成的代码

```csharp
public partial class SkillConfig : IConfig
{
    public int damageId => m_damageId;

    public DamageConfig GetDamage()
    {
        return DamageConfigs.Instance.Get(damageId);
    }
}
```

### 使用方式

```csharp
var skill = SkillConfigs.Instance.Get(1);
var damage = skill.GetDamage();  // 自动关联查询
Debug.Log(damage.name);  // 火焰伤害
```

### 注意事项

1. 目标表必须存在且包含指定的字段
2. 关联字段类型必须匹配（通常都是 `int`）
3. 可以建立多个关联，每个关联生成独立的访问方法
4. 支持跨文件关联

## 🎯 #enum — 枚举类型

### 语法

```
#enum[EnumType] 描述文本
```

### 说明

声明字段为枚举类型，导出时会生成枚举属性而非整数属性。

### 示例

| id | name | classType |
|----|------|-----------|
| int | string | int |
| 技能ID | 技能名称 | #enum\[Game.ClassType] 职业类型 |
| 1 | 烈焰斩 | Warrior |

### 自动生成的代码

```csharp
public partial class SkillConfig : IConfig
{
    public Game.ClassType classType => m_classType;
}
```

### 使用方式

```csharp
var skill = SkillConfigs.Instance.Get(1);
if (skill.classType == Game.ClassType.Warrior)
{
    // 战士技能逻辑
}
```

### 注意事项

1. 枚举类必须已存在于项目中
2. 支持完整命名空间（如 `Game.Enum.ClassType`）
3. Excel 中可以写枚举名称或整数值
4. 导出时会校验枚举值的有效性

## 📏 #range — 数值范围校验

### 语法

```
#range[min,max] 描述文本
```

### 说明

对 `int` 或 `float` 类型字段进行范围校验，导出时检查数据是否在指定范围内。

### 示例

| id | name | cooldown | manaCost |
|----|------|----------|----------|
| int | string | float | int |
| 技能ID | 技能名称 | #range\[0,300] 冷却时间(秒) | #range\[0,999] 魔法消耗 |
| 1 | 烈焰斩 | 5.0 | 20 |
| 2 | 冰霜新星 | 8.0 | 35 |

### 校验规则

| 规则 | 说明 |
|------|------|
| 最小值 | 包含边界（≥ min） |
| 最大值 | 包含边界（≤ max） |
| 错误提示 | 导出时显示具体的行号和数值 |

### 注意事项

1. 仅适用于 `int` 和 `float` 类型
2. min 和 max 必须是有效的数字
3. min 必须小于等于 max

## 📁 #checkpath — 路径校验

### 语法

```
#checkpath 描述文本
```

### 说明

对字符串类型字段进行路径存在性校验，确保配置中引用的资源路径有效。

### 示例

| id | name | prefabPath |
|----|------|------------|
| int | string | string |
| 技能ID | 技能名称 | #checkpath 预制体路径 |
| 1 | 烈焰斩 | Prefabs/Skill/FireSlash |

### 路径检查逻辑

路径检查器支持多种路径格式，自动尝试补全常用后缀：

| 输入格式 | 检查逻辑 |
|----------|----------|
| `D:/Projects/Game/Assets/Prefabs/Player.prefab` | 绝对路径，直接检查 |
| `Assets/Prefabs/Player` | 拼接工程根目录为绝对路径检查 |
| `Assets/Prefabs/Player.prefab` | 拼接工程根目录为绝对路径检查 |
| `Resources/Prefabs/Player` | 转换为 `Assets/Resources/Prefabs/Player` 后检查 |
| `/Prefabs/Player` | 去掉 `/` 后在所有 Resources 目录下搜索 |
| `Prefabs/Player` | 在所有 Resources 目录下搜索 |
| `Player` | 在所有 Resources 目录下搜索 + 尝试所有后缀 |

### 自动尝试的后缀

```
.prefab, .png, .jpg, .jpeg, .asset, .mat, .fbx, .anim, 
.txt, .json, .csv, .wav, .mp3, .shader, .controller, 
.mask, .rendertexture
```

### 注意事项

1. 仅适用于 `string` 类型
2. 路径检查忽略大小写差异
3. 支持 Unity Resources 目录下的资源搜索
4. 导出时会显示路径不存在的错误提示

## ⚠️ #ignore — 忽略列

### 语法

```
#ignore
```

### 说明

忽略该列，不生成代码和 JSON，仅在 Excel 中保留作为参考。

### 示例

| id | name | notes |
|----|------|-------|
| int | string | string |
| 技能ID | 技能名称 | #ignore 备注 |
| 1 | 烈焰斩 | 这是战士技能 |

### 注意事项

1. `#ignore` 标记所在的列不会出现在生成的代码和 JSON 中
2. 通常用于备注、说明等不需要运行时使用的字段

## 🔧 组合使用

多个 Schema 标记可以组合使用：

| id | name | damageId | cooldown |
|----|------|----------|----------|
| int | string | int | float |
| 技能ID | 技能名称 | #link\[Damage,id] #range\[1,99] 伤害类型 | #range\[0,300] 冷却时间 |
| 1 | 烈焰斩 | 1 | 5.0 |

## 📊 Schema 标记优先级

1. `#ignore` — 最高优先级，忽略整列
2. `#enum` — 枚举类型声明
3. `#link` — 跨表关联
4. `#range` — 数值范围校验
5. `#checkpath` — 路径校验

## ✅ 校验流程

导出时的校验顺序：

1. **ID 唯一性校验**：检查每张表的 id 字段是否唯一
2. **引用完整性校验**：检查 `#link` 关联的目标数据是否存在
3. **枚举有效性校验**：检查 `#enum` 字段值是否为有效枚举值
4. **数值范围校验**：检查 `#range` 字段值是否在范围内
5. **路径存在性校验**：检查 `#checkpath` 字段值对应的资源是否存在

所有校验失败都会在编辑器窗口中显示详细的错误信息，包括：
- 文件名和 Sheet 名称
- 行号和列号
- 具体的错误描述