# 扩展机制

LazyConfig 提供了灵活的扩展机制，支持自定义类型解析器、校验器和加密器。

## 📋 扩展类型

| 扩展类型 | 基类 | 说明 |
|----------|------|------|
| 自定义类型解析器 | `CustomTypeParserBase` | 解析自定义格式的数据 |
| 自定义校验器 | `CustomValidatorBase` | 自定义数据校验逻辑 |
| 自定义加密器（编辑器） | `CustomEncryptorBase` | 导出时加密数据 |
| 自定义加密器（运行时） | `RuntimeEncryptorBase` | 运行时解密数据 |

## 🔧 自定义类型解析器

### 步骤

1. 创建继承自 `CustomTypeParserBase` 的 ScriptableObject 类
2. 实现 `Parse(string value)` 方法
3. 创建 Asset 实例并注册到 `EditorConfig.asset`

### 示例

```csharp
using UnityEngine;
using LazyConfig.Editor.CustomType;

[CreateAssetMenu(fileName = "SkillEffectParser", menuName = "LazyConfig/Custom Type Parser/SkillEffect")]
public class SkillEffectParser : CustomTypeParserBase
{
    public override object Parse(string value)
    {
        string[] parts = value.Split(',');
        if (parts.Length >= 2)
        {
            int effectId = int.Parse(parts[0].Trim());
            string effectName = parts[1].Trim();
            return new SkillEffect(effectId, effectName);
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
| 2 | 冰霜新星 | 2,减速 |

### 注册到 EditorConfig

1. 在 `Assets/LazyConfig/Editor/` 目录下创建 `SkillEffectParser.asset`
2. 打开 `EditorConfig.asset`，在 `Custom Type Parsers` 列表中添加该 Asset

## ✅ 自定义校验器

### 步骤

1. 创建继承自 `CustomValidatorBase` 的 ScriptableObject 类
2. 实现 `Validate(ExcelTable table, List<ExcelTable> allTables)` 方法
3. 创建 Asset 实例并注册到 `EditorConfig.asset`

### 示例

```csharp
using UnityEngine;
using LazyConfig.Editor.CustomType;
using LazyConfig.Tool.Excel;
using System.Collections.Generic;

[CreateAssetMenu(fileName = "SkillEffectValidator", menuName = "LazyConfig/Custom Validator/SkillEffect")]
public class SkillEffectValidator : CustomValidatorBase
{
    public override List<string> Validate(ExcelTable table, List<ExcelTable> allTables)
    {
        List<string> errors = new List<string>();
        
        foreach (var row in table.DataRows)
        {
            int effectId = row.GetInt("effectId");
            if (effectId <= 0)
            {
                errors.Add($"Row {row.RowIndex}: effectId must be positive, got {effectId}");
            }
        }
        
        return errors;
    }
}
```

### 注册到 EditorConfig

1. 在 `Assets/LazyConfig/Editor/` 目录下创建 `SkillEffectValidator.asset`
2. 打开 `EditorConfig.asset`，在 `Custom Validators` 列表中添加该 Asset

## 🔐 自定义加密器

自定义加密器需要分别实现编辑器端和运行时版本。

### 编辑器端加密器

```csharp
using UnityEngine;
using LazyConfig.Editor.CustomEncryptor;

[CreateAssetMenu(fileName = "AesEditorEncryptor", menuName = "LazyConfig/Custom Encryptor/AES")]
public class AesEditorEncryptor : CustomEncryptorBase
{
    [SerializeField] private string m_key = "YourSecretKey16Bytes";
    
    public override byte[] Encrypt(byte[] data)
    {
        // AES 加密实现
        return AesCryptoUtility.Encrypt(data, m_key);
    }
}
```

### 运行时加密器

```csharp
using UnityEngine;
using LazyConfig.Runtime.Core.Encryptors;

[CreateAssetMenu(fileName = "AesRuntimeEncryptor", menuName = "LazyConfig/Runtime Encryptor/AES")]
public class AesRuntimeEncryptor : RuntimeEncryptorBase
{
    [SerializeField] private string m_key = "YourSecretKey16Bytes";
    
    public override byte[] Decrypt(byte[] data)
    {
        // AES 解密实现
        return AesCryptoUtility.Decrypt(data, m_key);
    }
}
```

### 注册加密器

1. 编辑器端：创建 `AesEditorEncryptor.asset`，在导出窗口中选择
2. 运行时：创建 `AesRuntimeEncryptor.asset`，在 `RuntimeConfig.asset` 中设置

### 加密数据格式

自定义加密器加密的数据首字节必须是 `0xCC`，用于运行时识别加密方式。

## 🔄 Pipeline 任务扩展

LazyConfig 的导出流程基于 Pipeline 模式，可以自定义扩展每个阶段。

### Pipeline 阶段

```
LoadSettingsTask → ScanExcelFilesTask → ParseExcelTask → ParseReferencesTask
    → ValidateTask → GenerateCodeTask → ExportJsonTask → RefreshTask
```

### 自定义任务

实现 `IBuildTask` 接口：

```csharp
public interface IBuildTask
{
    string Name { get; }
    bool Run(BuildContext context);
}
```

### 替换默认任务

在 `EditorConfig.asset` 中替换特定阶段的任务实现。

## 🔌 适配器模式

Editor 层通过适配器桥接 Unity 和 Tool 层：

| 适配器 | 目标接口 | 说明 |
|--------|----------|------|
| `ParserAdapter` | `Tool.ICustomTypeParser` | 自定义类型解析器适配器 |
| `ValidatorAdapter` | `Tool.IConfigValidator` | 自定义校验器适配器 |
| `EncryptorAdapter` | `Tool.IConfigEncryptor` | 自定义加密器适配器 |

## ⚠️ 注意事项

1. **ScriptableObject 生命周期**：自定义扩展都是 ScriptableObject，需要创建 Asset 实例
2. **命名空间**：确保自定义类的命名空间正确，与生成的代码一致
3. **序列化**：自定义类型需要支持 Unity 序列化才能在 Inspector 中显示
4. **错误处理**：解析器和校验器的错误会在导出窗口中显示
5. **性能考虑**：解析器会在导出时被频繁调用，避免复杂计算
6. **加密密钥安全**：不要将加密密钥硬编码在代码中，使用配置文件

## 🎯 扩展示例

### 示例 1：自定义 Vector2Int 解析器

```csharp
using UnityEngine;
using LazyConfig.Editor.CustomType;

[CreateAssetMenu(fileName = "Vector2IntParser", menuName = "LazyConfig/Custom Type Parser/Vector2Int")]
public class Vector2IntParser : CustomTypeParserBase
{
    public override object Parse(string value)
    {
        string trimmed = value.Trim('(', ')');
        string[] parts = trimmed.Split(',');
        if (parts.Length == 2)
        {
            int x = int.Parse(parts[0].Trim());
            int y = int.Parse(parts[1].Trim());
            return new Vector2Int(x, y);
        }
        return Vector2Int.zero;
    }
}
```

### 示例 2：自定义 ID 唯一性校验器

```csharp
using UnityEngine;
using LazyConfig.Editor.CustomType;
using LazyConfig.Tool.Excel;
using System.Collections.Generic;

[CreateAssetMenu(fileName = "CustomIdValidator", menuName = "LazyConfig/Custom Validator/ID Uniqueness")]
public class CustomIdValidator : CustomValidatorBase
{
    public override List<string> Validate(ExcelTable table, List<ExcelTable> allTables)
    {
        List<string> errors = new List<string>();
        HashSet<int> ids = new HashSet<int>();
        
        foreach (var row in table.DataRows)
        {
            int id = row.GetInt("id");
            if (ids.Contains(id))
            {
                errors.Add($"Row {row.RowIndex}: Duplicate ID {id}");
            }
            ids.Add(id);
        }
        
        return errors;
    }
}
```

### 示例 3：自定义 Base64 加密器

```csharp
using UnityEngine;
using LazyConfig.Editor.CustomEncryptor;
using System;

[CreateAssetMenu(fileName = "Base64Encryptor", menuName = "LazyConfig/Custom Encryptor/Base64")]
public class Base64Encryptor : CustomEncryptorBase
{
    public override byte[] Encrypt(byte[] data)
    {
        byte[] result = new byte[data.Length + 1];
        result[0] = 0xCC; // 加密标识
        byte[] base64 = Convert.FromBase64String(Convert.ToBase64String(data));
        Array.Copy(base64, 0, result, 1, base64.Length);
        return result;
    }
}
```

## 📊 扩展开发流程

```
需求分析 → 创建基类派生类 → 实现核心方法 → 创建 Asset 实例 → 注册到配置 → 测试验证
```

### 开发建议

1. **保持简单**：解析器和校验器应该做单一职责的事情
2. **错误信息清晰**：校验器的错误信息应该包含具体的行号和原因
3. **测试覆盖**：为自定义扩展编写单元测试
4. **文档注释**：为自定义类型添加 XML 文档注释
5. **版本兼容**：确保自定义扩展与 LazyConfig 版本兼容