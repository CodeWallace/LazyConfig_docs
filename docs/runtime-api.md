# 运行时 API

LazyConfig 提供了丰富的运行时 API，支持多种加载模式和配置访问方式。

## 📦 配置管理类

每个配置表都会自动生成对应的管理类，命名格式为 `{TableName}Configs`。

### 访问单例

```csharp
var skillConfigs = SkillConfigs.Instance;
```

单例访问会自动触发初始化和加载，如果配置尚未加载会同步加载。

### 查询方法

| 方法 | 说明 | 示例 |
|------|------|------|
| `Get(int id)` | 按 ID 获取配置（O(1)） | `skillConfigs.Get(1)` |
| `Find(Predicate<T> match)` | 按条件查找第一条 | `skillConfigs.Find(s => s.name.Contains("火"))` |
| `FindAll(Predicate<T> match)` | 按条件查找所有 | `skillConfigs.FindAll(s => s.manaCost > 30)` |
| `Contains(int id)` | 检查是否包含指定 ID | `skillConfigs.Contains(1)` |
| `GetFirst()` | 获取第一条配置 | `skillConfigs.GetFirst()` |
| `GetLast()` | 获取最后一条配置 | `skillConfigs.GetLast()` |
| `GetRange(int startIndex, int count)` | 获取指定范围的配置 | `skillConfigs.GetRange(0, 5)` |

### 属性

| 属性 | 说明 | 类型 |
|------|------|------|
| `Items` | 获取所有配置（只读集合） | `IReadOnlyCollection<T>` |
| `IsLoaded` | 是否已加载完成 | `bool` |
| `IsLoading` | 是否正在加载中 | `bool` |
| `Count` | 配置数量 | `int` |

### 示例

```csharp
using Game.Config;

// 按 ID 查询
var skill = SkillConfigs.Instance.Get(1);
Debug.Log(skill.name);

// 条件查找
var fireSkills = SkillConfigs.Instance.FindAll(s => s.name.Contains("火"));

// 遍历所有配置
foreach (var s in SkillConfigs.Instance.Items)
{
    Debug.Log(s.name);
}

// 检查加载状态
if (SkillConfigs.Instance.IsLoaded)
{
    Debug.Log("配置已加载");
}
```

## 🔗 跨表关联访问

使用 `#link` Schema 标记的字段会自动生成 `GetXxx()` 访问方法。

### 示例

```csharp
var skill = SkillConfigs.Instance.Get(1);

// 自动生成的关联访问方法
var damage = skill.GetDamage();      // 获取伤害类型配置
var effect = skill.GetEffect();      // 获取技能效果配置

if (damage != null)
{
    Debug.Log(damage.name);
}
```

## ⚡ 加载模式

### 同步加载

```csharp
// 同步初始化所有配置（会阻塞主线程）
ConfigSystem.Instance.Initialize();

// 同步加载单个配置
var skillConfigs = SkillConfigs.Instance;  // 访问时自动同步加载
```

### 异步加载

```csharp
// 异步初始化所有配置
await ConfigSystem.Instance.InitializeAsync();

// 异步加载单个配置
await SkillConfigs.LoadInstanceAsync();
```

### 回调加载

```csharp
ConfigSystem.Instance.InitializeAsync((success, errors) =>
{
    if (success)
    {
        Debug.Log("加载成功");
    }
    else
    {
        foreach (string error in errors)
        {
            Debug.LogError(error);
        }
    }
});
```

### 并行加载

```csharp
// 并行异步加载所有配置（最快）
await ConfigSystem.Instance.InitializeAsyncParallel();
```

## 🔄 异步/同步混合模式

LazyConfig 支持异步预加载和同步访问的混合模式：

```csharp
// 启动时异步预加载（不阻塞）
ConfigSystem.Instance.InitializeAsync();

// 直接访问时，如果异步未完成，会自动掐断并同步加载
var skill = SkillConfigs.Instance.Get(1);  // 自动处理加载状态
```

### 加载状态流转

```
未加载 → 异步加载中 → 已加载
           ↓
      同步访问触发
           ↓
      同步加载完成 → 已加载
```

## 🗂️ ConfigSystem 全局管理

`ConfigSystem` 提供全局配置管理功能。

### 注册配置

```csharp
// 手动注册配置（通常由生成的代码自动完成）
ConfigSystem.Instance.Register(new SkillConfigs());
```

### 初始化

```csharp
// 同步初始化
ConfigSystem.Instance.Initialize();

// 异步初始化（await）
await ConfigSystem.Instance.InitializeAsync();

// 异步初始化（回调）
ConfigSystem.Instance.InitializeAsync((success, errors) => { });

// 并行异步初始化
await ConfigSystem.Instance.InitializeAsyncParallel();
```

### 状态检查

```csharp
// 是否已初始化
bool isInitialized = ConfigSystem.Instance.isInitialized;

// 是否正在初始化
bool isInitializing = ConfigSystem.Instance.isInitializing;

// 是否已注册指定类型
bool isRegistered = ConfigSystem.Instance.IsRegistered<SkillConfigs>();

// 获取已注册配置数量
int count = ConfigSystem.Instance.RegisteredCount;
```

### 清理

```csharp
// 注销指定类型的配置
ConfigSystem.Instance.Unregister<SkillConfigs>();

// 释放所有资源
ConfigSystem.Instance.Dispose();
```

## 🔐 配置加密

运行时自动识别加密方式并解密，无需额外配置。

### 加密方式

| 加密方式 | 说明 | 版本要求 |
|----------|------|----------|
| **None** | 不加密，明文存储 | Free/Pro |
| **XOR** | 简单异或加密 | Free/Pro |
| **MD5** | MD5 哈希密钥加密 | Pro 专属 |
| **Custom** | 自定义加密器（内置 AES 示例） | Pro 专属 |

### 配置加密器

在 `RuntimeConfig.asset` 中设置加密配置：

1. 设置 `Encryption Mode`（加密模式）
2. 设置 `Encryption Key`（加密密钥）
3. 自定义模式需创建 `RuntimeEncryptorBase` 派生的 ScriptableObject
4. 编辑器端需创建 `CustomEncryptorBase` 派生的加密器

## 📁 配置加载路径

配置文件默认放在 `Resources/Configs/` 目录下，运行时通过 `Resources.Load` 加载。

### 自定义加载路径

```csharp
// 在生成的配置管理类中设置（通常不需要手动修改）
SkillConfigs.Instance.Path = "Configs/Skill";
```

### IConfigLoader 接口

通过实现 `IConfigLoader` 接口可以自定义配置加载方式：

```csharp
public interface IConfigLoader
{
    string LoadText(string path);
    byte[] LoadBytes(string path);
}
```

## ⚠️ 注意事项

1. **线程安全**：配置访问不是线程安全的，建议在主线程访问
2. **内存管理**：配置数据会常驻内存，适合游戏运行时使用
3. **热更新**：不支持热更新，修改配置需要重新导出和打包
4. **ID 唯一性**：每张表的 ID 必须唯一，否则会覆盖
5. **空值检查**：`Get(id)` 可能返回 null，建议进行空值检查
6. **异步加载**：异步加载完成前访问会自动转为同步加载

## 🎯 最佳实践

### 启动时预加载

```csharp
public class GameBootstrapper : MonoBehaviour
{
    private async void Start()
    {
        // 显示加载界面
        ShowLoadingScreen();
        
        // 并行异步加载所有配置
        await ConfigSystem.Instance.InitializeAsyncParallel();
        
        // 隐藏加载界面，进入游戏
        HideLoadingScreen();
        EnterGame();
    }
}
```

### 按需加载

```csharp
public class SkillManager : MonoBehaviour
{
    private void Awake()
    {
        // 预加载技能配置
        SkillConfigs.LoadInstanceAsync().ContinueWith(task =>
        {
            if (task.Result)
            {
                Debug.Log("技能配置加载完成");
            }
        });
    }
    
    public SkillConfig GetSkill(int id)
    {
        // 访问时如果未加载会自动同步加载
        return SkillConfigs.Instance.Get(id);
    }
}
```

### 缓存常用配置

```csharp
public class BattleSystem
{
    private Dictionary<int, SkillConfig> m_skillCache;
    
    private void InitializeCache()
    {
        m_skillCache = new Dictionary<int, SkillConfig>();
        foreach (var skill in SkillConfigs.Instance.Items)
        {
            m_skillCache[skill.id] = skill;
        }
    }
    
    public SkillConfig GetSkill(int id)
    {
        if (m_skillCache.TryGetValue(id, out var skill))
        {
            return skill;
        }
        return null;
    }
}
```