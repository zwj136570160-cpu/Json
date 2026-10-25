# JsonMgr 脚本说明

## 简介

`JsonMgr` 是一个 JSON 数据管理类（单例模式），用于 JSON 数据的序列化存储与反序列化读取：

- **序列化**：把内存中的对象转为 JSON 字符串并保存到硬盘；
- **反序列化**：从硬盘读取 JSON 文件，还原为内存对象。

类内部集成了两种 JSON 处理方案，通过枚举 `JsonType` 选择：

| 枚举值 | 方案 |
| --- | --- |
| `JsonType.JsonUtility` | Unity 自带的 JsonUtility |
| `JsonType.LitJson` | 第三方库 LitJson（默认方案） |

## 访问方式

构造函数私有，外部不能 `new`，通过 `Instance` 访问唯一实例：

```csharp
JsonMgr.Instance.SaveData(...);
T data = JsonMgr.Instance.LoadData<T>("文件名");
```

## API

### 保存数据

```csharp
public void SaveData(object data, string fileName, JsonType type = JsonType.LitJson)
```

- `data`：要保存的对象；
- `fileName`：文件名，无需带 `.json` 后缀，脚本会自动拼接；
- `type`：序列化方案，可选，默认 `LitJson`。

文件固定写入 `Application.persistentDataPath/fileName.json`，同名文件会被直接覆盖。

### 读取数据

```csharp
public T LoadData<T>(string fileName, JsonType type = JsonType.LitJson) where T : new()
```

- `fileName`：文件名，无需带 `.json` 后缀；
- `type`：反序列化方案，可选，默认 `LitJson`；
- 泛型约束 `where T : new()`：T 必须具有无参构造函数。

文件查找顺序：

1. 先找 `Application.streamingAssetsPath/fileName.json`；
2. 找不到再找 `Application.persistentDataPath/fileName.json`；
3. 两处都不存在，返回 `new T()`（空白对象，不会返回 null）。

## 快速示例

```csharp
// 存档
PlayerData data = new PlayerData();
data.playerName = "玩家1";
data.level = 10;
JsonMgr.Instance.SaveData(data, "PlayerData");

// 读档
PlayerData loadData = JsonMgr.Instance.LoadData<PlayerData>("PlayerData");
Debug.Log(loadData.playerName);
```

## 注意事项

- 文件名不要带 `.json` 后缀；
- 同一份数据存取时建议使用相同的 `JsonType`；
- 读取优先级为 `streamingAssetsPath` > `persistentDataPath` > `new T()`；
- 文件不存在时返回的是空白对象，字段均为默认值，无需判空。
