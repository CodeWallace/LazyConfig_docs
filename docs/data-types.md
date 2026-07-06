# 数据类型

LazyConfig 支持丰富的数据类型，包括基础类型、Unity 类型、集合类型和自定义类型。

## 📋 类型清单

### 基础类型

| 类型 | Excel 写法 | 说明 | 默认值 |
|------|------------|------|--------|
| `int` | `int` | 32 位整数 | 0 |
| `short` | `short` | 16 位整数 | 0 |
| `byte` | `byte` | 8 位无符号整数 | 0 |
| `uint` | `uint` | 32 位无符号整数 | 0 |
| `long` | `long` | 64 位整数 | 0 |
| `float` | `float` | 单精度浮点数 | 0.0f |
| `double` | `double` | 双精度浮点数 | 0.0 |
| `string` | `string` | 字符串 | null |
| `bool` | `bool` | 布尔值 | false |

### Unity 类型

| 类型 | Excel 写法 | 说明 | 默认值 |
|------|------------|------|--------|
| `Vector2` | `Vector2` | 二维向量 | (0, 0) |
| `Vector3` | `Vector3` | 三维向量 | (0, 0, 0) |
| `Vector4` | `Vector4` | 四维向量 | (0, 0, 0, 0) |
| `Color` | `Color` | 颜色（RGBA） | (0, 0, 0, 1) |
| `Quaternion` | `Quaternion` | 四元数 | (0, 0, 0, 1) |

### 数组类型

| 类型 | Excel 写法 | 说明 |
|------|------------|------|
| `int[]` | `int[]` | 整数数组 |
| `float[]` | `float[]` | 浮点数数组 |
| `string[]` | `string[]` | 字符串数组 |
| `bool[]` | `bool[]` | 布尔数组 |
| `Vector3[]` | `Vector3[]` | 三维向量数组 |
| `Vector4[]` | `Vector4[]` | 四维向量数组 |

### 列表类型

| 类型 | Excel 写法 | 说明 |
|------|------------|------|
| `List<int>` | `List<int>` | 整数列表 |
| `List<float>` | `List<float>` | 浮点数列表 |
| `List<string>` | `List<string>` | 字符串列表 |
| `List<Vector3>` | `List<Vector3>` | 三维向量列表 |

### 字典类型

| 类型 | Excel 写法 | 说明 |
|------|------------|------|
| `Dictionary<int,string>` | `Dictionary<int,string>` | 整数键字符串值字典 |
| `Dictionary<string,float>` | `Dictionary<string,float>` | 字符串键浮点数值字典 |
| `SerializableDictionary<K,V>` | `SerializableDictionary<K,V>` | 可序列化字典（Unity Inspector 可编辑） |

### 枚举类型

使用 `#enum` Schema 标记声明：

| classType |
|-----------|
| #enum\[Game.ClassType] int |
| 职业类型 |
| Warrior |

## 📝 Excel 数据格式

### 基础类型示例

| id | name | attack | isActive |
|----|------|--------|----------|
| int | string | float | bool |
| ID | 名称 | 攻击力 | 是否激活 |
| 1 | 战士 | 100.5 | true |
| 2 | 法师 | 80.0 | false |

### 数组类型示例

| id | name | skillIds | tags |
|----|------|----------|------|
| int | string | int[] | string[] |
| ID | 名称 | 技能ID列表 | 标签列表 |
| 1 | 战士 | [1,2,3] | [近战,物理,坦克] |
| 2 | 法师 | [4,5] | [远程,魔法,输出] |

### 列表类型示例

| id | name | statGrowth |
|----|------|------------|
| int | string | List\<float> |
| ID | 名称 | 属性成长值 |
| 1 | 战士 | [1.0, 0.8, 1.2, 1.5] |
| 2 | 法师 | [0.5, 1.5, 0.8, 0.6] |

### Unity 类型示例

| id | name | spawnPosition | hairColor | rotation |
|----|------|---------------|-----------|----------|
| int | string | Vector3 | Color | Quaternion |
| ID | 名称 | 出生位置 | 头发颜色 | 旋转角度 |
| 1 | 战士 | (10.0, 0.0, 5.0) | (1.0, 0.5, 0.0, 1.0) | (0.0, 0.0, 0.0, 1.0) |
| 2 | 法师 | (20.0, 0.0, 8.0) | (0.0, 0.5, 1.0, 1.0) | (0.0, 90.0, 0.0, 0.707) |

### 字典类型示例

| id | name | attributes |
|----|------|------------|
| int | string | Dictionary\<string,float> |
| ID | 名称 | 属性字典 |
| 1 | 战士 | {HP:1000,ATK:100,DEF:50} |
| 2 | 法师 | {HP:500,ATK:200,DEF:20} |

## 🔍 类型转换规则

### 整数类型

- Excel 中的数字会自动转换为对应的整数类型
- 如果超出范围会导致溢出（导出时会警告）
- 空单元格转换为 0

### 浮点类型

- Excel 中的数字会自动转换为浮点数
- 支持科学计数法（如 `1.5e10`）
- 空单元格转换为 0.0

### 字符串类型

- 直接读取 Excel 中的文本内容
- 空单元格转换为 null
- 特殊字符需要转义

### 布尔类型

| Excel 值 | 转换结果 |
|----------|----------|
| `true` | true |
| `false` | false |
| `1` | true |
| `0` | false |
| `是` | true |
| `否` | false |
| `Y` | true |
| `N` | false |

### 数组和列表类型

- 使用方括号 `[]` 包裹
- 元素用逗号分隔
- 支持嵌套的 Unity 类型

### Unity 类型

- 使用圆括号 `()` 包裹
- 分量用逗号分隔
- 分量顺序必须与类型定义一致

## 🎯 自定义类型

通过实现 `CustomTypeParserBase` 可以扩展自定义类型：

### 步骤

1. 创建一个继承自 `CustomTypeParserBase` 的 ScriptableObject 类
2. 实现 `Parse(string value)` 方法
3. 创建 Asset 实例并注册到 `EditorConfig.asset`

### 示例

```csharp
using UnityEngine;
using LazyConfig.Editor.CustomType;

public class SkillEffectParser : CustomTypeParserBase
{
    public override object Parse(string value)
    {
        string[] parts = value.Split(',');
        if (parts.Length >= 2)
        {
            return new SkillEffect(
                int.Parse(parts[0]),
                parts[1].Trim()
            );
        }
        return null;
    }
}
```

### 在 Excel 中使用

| id | name | effect |
|----|------|--------|
| int | string | SkillEffect |
| ID | 名称 | 技能效果 |
| 1 | 烈焰斩 | 1,燃烧 |

## ⚠️ 注意事项

1. **类型大小写**：Excel 中的类型名不区分大小写
2. **格式一致性**：同一列的数据格式必须一致
3. **空值处理**：空单元格会被解析为该类型的默认值
4. **精度问题**：浮点数可能存在精度损失，建议使用适当的小数位数
5. **数组长度**：建议保持数组长度一致，便于代码处理
6. **字典键类型**：字典的键类型建议使用 `int` 或 `string`

## 📊 类型选择建议

| 场景 | 推荐类型 | 原因 |
|------|----------|------|
| ID 字段 | `int` | 高效查找，O(1) |
| 血量/蓝量 | `int` | 精确数值，无精度损失 |
| 百分比/概率 | `float` | 需要小数精度 |
| 坐标/位置 | `Vector3` | Unity 原生支持 |
| 颜色 | `Color` | RGBA 四通道 |
| 多个 ID | `int[]` | 简单数组 |
| 动态数量 | `List<T>` | 可增删 |
| 键值对 | `Dictionary<K,V>` | 键值映射 |
| 自定义结构 | 自定义类型 | 灵活扩展 |