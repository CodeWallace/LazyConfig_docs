# 使用示例

本文提供了 LazyConfig 的各种使用场景示例，帮助你快速上手。

## 🎯 基础用法

### 访问配置

```csharp
using Game.Config;
using UnityEngine;

public class SkillExample : MonoBehaviour
{
    private void Start()
    {
        // 访问单例（自动加载）
        var skill = SkillConfigs.Instance.Get(1);
        
        if (skill != null)
        {
            Debug.Log($"技能: {skill.name}");
            Debug.Log($"冷却时间: {skill.cooldown}s");
            Debug.Log($"魔法消耗: {skill.manaCost}");
        }
    }
}
```

### 遍历配置

```csharp
public class SkillListExample : MonoBehaviour
{
    private void Start()
    {
        foreach (var skill in SkillConfigs.Instance.Items)
        {
            Debug.Log($"ID: {skill.id}, 名称: {skill.name}");
        }
    }
}
```

## 🔗 跨表关联

### 基础关联

```csharp
public class SkillDamageExample : MonoBehaviour
{
    private void Start()
    {
        var skill = SkillConfigs.Instance.Get(1);
        
        // 通过自动生成的方法访问关联数据
        var damage = skill.GetDamage();
        var effect = skill.GetEffect();
        
        if (damage != null)
        {
            Debug.Log($"{skill.name} 的伤害类型: {damage.name}");
        }
        
        if (effect != null)
        {
            Debug.Log($"{skill.name} 的效果: {effect.name}");
        }
    }
}
```

### 多级关联

```csharp
public class MultiLevelLinkExample : MonoBehaviour
{
    private void Start()
    {
        var level = LevelConfigs.Instance.Get(1);
        
        // 一级关联
        var boss = level.GetBossCharacter();
        
        if (boss != null)
        {
            Debug.Log($"关卡 {level.name} 的 Boss: {boss.name}");
            
            // 二级关联（通过 Boss 角色关联到技能）
            foreach (int skillId in boss.skillIds)
            {
                var skill = SkillConfigs.Instance.Get(skillId);
                if (skill != null)
                {
                    Debug.Log($"  - Boss 技能: {skill.name}");
                }
            }
        }
    }
}
```

## ⚡ 加载模式

### 同步加载

```csharp
public class SyncLoadExample : MonoBehaviour
{
    private void Start()
    {
        // 同步初始化所有配置（会阻塞主线程）
        ConfigSystem.Instance.Initialize();
        
        Debug.Log("所有配置加载完成");
        Debug.Log($"已加载配置数量: {ConfigSystem.Instance.RegisteredCount}");
    }
}
```

### 异步加载（Await）

```csharp
public class AsyncLoadExample : MonoBehaviour
{
    private async void Start()
    {
        Debug.Log("开始异步加载...");
        
        // 异步初始化所有配置
        await ConfigSystem.Instance.InitializeAsync();
        
        Debug.Log("异步加载完成");
        
        // 访问配置
        var skill = SkillConfigs.Instance.Get(1);
        Debug.Log($"技能名称: {skill.name}");
    }
}
```

### 异步加载（回调）

```csharp
public class CallbackLoadExample : MonoBehaviour
{
    private void Start()
    {
        Debug.Log("开始异步加载（回调模式）...");
        
        ConfigSystem.Instance.InitializeAsync((success, errors) =>
        {
            if (success)
            {
                Debug.Log("加载成功");
                var skill = SkillConfigs.Instance.Get(1);
                Debug.Log($"技能名称: {skill.name}");
            }
            else
            {
                Debug.LogError("加载失败:");
                foreach (string error in errors)
                {
                    Debug.LogError($"  - {error}");
                }
            }
        });
    }
}
```

### 并行加载

```csharp
public class ParallelLoadExample : MonoBehaviour
{
    private async void Start()
    {
        Debug.Log("开始并行异步加载...");
        
        // 并行加载所有配置（最快方式）
        await ConfigSystem.Instance.InitializeAsyncParallel();
        
        Debug.Log("并行加载完成");
        
        // 访问所有配置
        Debug.Log($"角色数量: {CharacterConfigs.Instance.Count}");
        Debug.Log($"技能数量: {SkillConfigs.Instance.Count}");
        Debug.Log($"伤害类型数量: {DamageConfigs.Instance.Count}");
    }
}
```

## 📊 条件查询

### 简单条件

```csharp
public class ConditionQueryExample : MonoBehaviour
{
    private void Start()
    {
        // 查找魔法消耗大于 30 的技能
        var highManaSkills = SkillConfigs.Instance.FindAll(s => s.manaCost > 30);
        
        Debug.Log("高魔法消耗技能:");
        foreach (var skill in highManaSkills)
        {
            Debug.Log($"  - {skill.name}: {skill.manaCost} MP");
        }
    }
}
```

### 复杂条件

```csharp
public class ComplexQueryExample : MonoBehaviour
{
    private void Start()
    {
        // 查找冷却时间在 5-10 秒之间且魔法消耗小于 50 的技能
        var skills = SkillConfigs.Instance.FindAll(s => 
            s.cooldown >= 5 && 
            s.cooldown <= 10 && 
            s.manaCost < 50
        );
        
        Debug.Log("中等冷却且低消耗技能:");
        foreach (var skill in skills)
        {
            Debug.Log($"  - {skill.name}: {skill.cooldown}s, {skill.manaCost} MP");
        }
    }
}
```

### 使用枚举条件

```csharp
public class EnumQueryExample : MonoBehaviour
{
    private void Start()
    {
        // 查找战士职业的角色
        var warriors = CharacterConfigs.Instance.FindAll(c => 
            c.classType == Game.ClassType.Warrior
        );
        
        Debug.Log("战士角色:");
        foreach (var character in warriors)
        {
            Debug.Log($"  - {character.name}");
        }
    }
}
```

## 🎮 游戏实战示例

### 示例 1：技能系统

```csharp
public class SkillSystem : MonoBehaviour
{
    public void UseSkill(int skillId)
    {
        var skill = SkillConfigs.Instance.Get(skillId);
        
        if (skill == null)
        {
            Debug.LogError($"技能 ID {skillId} 不存在");
            return;
        }
        
        Debug.Log($"使用技能: {skill.name}");
        Debug.Log($"描述: {skill.description}");
        
        // 获取伤害类型
        var damage = skill.GetDamage();
        if (damage != null)
        {
            Debug.Log($"伤害类型: {damage.name}");
        }
        
        // 获取技能效果
        var effect = skill.GetEffect();
        if (effect != null)
        {
            Debug.Log($"效果: {effect.name}");
        }
    }
}
```

### 示例 2：角色系统

```csharp
public class CharacterSystem : MonoBehaviour
{
    public CharacterConfig GetCharacter(int id)
    {
        return CharacterConfigs.Instance.Get(id);
    }
    
    public void PrintCharacterInfo(int id)
    {
        var character = CharacterConfigs.Instance.Get(id);
        
        if (character == null)
        {
            Debug.LogError($"角色 ID {id} 不存在");
            return;
        }
        
        Debug.Log($"=== {character.name} ===");
        Debug.Log($"职业: {character.classType}");
        Debug.Log($"生命值: {character.maxHp}");
        Debug.Log($"魔法值: {character.maxMp}");
        Debug.Log($"攻击力: {character.attack}");
        Debug.Log($"防御力: {character.defense}");
        Debug.Log($"移动速度: {character.moveSpeed}");
        Debug.Log($"出生位置: {character.spawnPosition}");
        
        // 打印技能列表
        Debug.Log("技能:");
        foreach (int skillId in character.skillIds)
        {
            var skill = SkillConfigs.Instance.Get(skillId);
            if (skill != null)
            {
                Debug.Log($"  - {skill.name}");
            }
        }
    }
}
```

### 示例 3：关卡系统

```csharp
public class LevelSystem : MonoBehaviour
{
    public LevelConfig GetLevel(int id)
    {
        return LevelConfigs.Instance.Get(id);
    }
    
    public void LoadLevel(int levelId)
    {
        var level = LevelConfigs.Instance.Get(levelId);
        
        if (level == null)
        {
            Debug.LogError($"关卡 ID {levelId} 不存在");
            return;
        }
        
        Debug.Log($"加载关卡: {level.name}");
        Debug.Log($"推荐战力: {level.recommendedPower}");
        Debug.Log($"关卡阶段: {level.stageCount}");
        
        // 获取 Boss 信息
        var boss = level.GetBossCharacter();
        if (boss != null)
        {
            Debug.Log($"Boss: {boss.name} ({boss.classType})");
        }
        
        // 打印奖励
        Debug.Log("奖励:");
        foreach (string reward in level.rewards)
        {
            Debug.Log($"  - {reward}");
        }
    }
}
```

## 🔧 自定义类型示例

### 使用自定义 SkillEffect 类型

```csharp
public class CustomTypeExample : MonoBehaviour
{
    private void Start()
    {
        var skill = SkillConfigs.Instance.Get(1);
        
        // 直接访问自定义类型字段
        var effect = skill.effect;
        
        if (effect != null)
        {
            Debug.Log($"效果 ID: {effect.effectId}");
            Debug.Log($"效果名称: {effect.effectName}");
        }
    }
}
```

## ⚠️ 注意事项

1. **空值检查**：`Get(id)` 可能返回 null，建议进行空值检查
2. **线程安全**：配置访问不是线程安全的，建议在主线程访问
3. **性能优化**：频繁访问的配置可以缓存到本地变量
4. **加载状态**：在异步加载完成前访问会自动转为同步加载
5. **资源释放**：不需要时可以调用 `ConfigSystem.Instance.Dispose()` 释放资源

## 📚 更多示例

查看项目中的 Samples 目录获取完整的示例代码：

```
Assets/Plugins/LazyConfig/Samples/
├── Scripts/
│   ├── UseageExample.cs      # 完整使用示例
│   └── DemoUIBuilder.cs      # UI 演示
└── UsageExample.unity        # 演示场景
```