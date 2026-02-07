---
name: openclaw-qweather
description: Query weather via QWeather Enterprise API with automatic JWT auth + geo resolution, and Open-Meteo fallback for current conditions.
user-invocable: true
metadata: { "openclaw": { "emoji": "☀️" } }
---

# openclaw-qweather 天气查询 Skill

## 你能用它做什么

本 Skill 面向「对话/代理」提供结构化天气查询能力：

- 实时天气（当前温度、体感、湿度、风等）
- 天气预报（日预报 3/7/15 天）
- 逐小时预报（24/72/168 小时）
- 分钟级降水（未来 2 小时，每 5 分钟）
- 气象预警
- 生活指数（穿衣、洗车、运动、紫外线等）

### 重要说明：调用方不需要关心 JWT 和位置解析

对外调用只需要传 `location`（城市名 / 经纬度 / locationId）。
Skill 内部会自动完成：

1) 获取并缓存企业 API JWT（Ed25519 / JWT）  
2) 将输入位置解析为 QWeather 可用的 `locationId`（必要时提取经纬度）  
3) 请求对应天气接口  
4) 对 `weather_now`：当企业 API 异常时自动回退到 Open-Meteo（保证实时天气可用）

> 这符合 OpenClaw Skill 的设计：对外暴露的是“能力”，不是“实现步骤”。  
> 并且 Skill 文档建议在正文中使用 `{baseDir}` 引用技能目录路径。 :contentReference[oaicite:1]{index=1}

---

## 输入约定（Input）

所有查询 action 通用输入：

- `location`（可选）
  - 城市名：`"北京"`
  - 经纬度：`"116.4,39.9"`（建议使用 `lon,lat`）
  - 位置 ID：`"101010100"`
  - 未传入时使用默认位置（由配置决定）

---

## 可用 Actions（对外能力）

> 说明：以下 actions 代表“你应该从用户意图映射到哪种查询”，而不是要求你显式调用 JWT/Geo。

### 1 `weather_now` 实时天气（含回退）

**适用**：用户问“现在XX天气怎样/多少度/风大吗/要不要带伞（当前）”。  
**特性**：企业 API 失败时自动回退 Open-Meteo。

**输入**
- `{ location? }`

**输出（成功）**
- `success: true`
- `source: "QWeather Enterprise API" | "Open-Meteo (free tier)"`
- `now: { temp, feelsLike, text, humidity?, windDir?, windScale?, obsTime? }`
- `fxLink?`

---

### 2 `weather_forecast` 日预报（3/7/15 天）

**适用**：用户问“未来几天/本周天气趋势”。  
**输入**
- `{ location?, days? }`，days 推荐值：`3 | 7 | 15`

**输出（成功）**
- `success: true`
- `forecast: [{ date, tempMax, tempMin, textDay, textNight, precip? }]`

---

### 3 `weather_hourly` 逐小时预报（24/72/168 小时）

**适用**：用户需要“今天每小时温度/降水概率变化”。  
**输入**
- `{ location?, hours? }`，hours：`"24h" | "72h" | "168h"`

**输出（成功）**
- `success: true`
- `hourly: [{ time, temp, text, humidity?, precip?, pop?, windDir?, windScale?, windSpeed?, ... }]`

---

### 4 `weather_minutely_precipitation` 分钟级降水（2 小时）

**适用**：短时决策（是否马上会下雨、是否要带伞、临近出门提醒）。  
**输入**
- `{ location? }`（内部会自动保证获得经纬度）

**输出（成功）**
- `success: true`
- `precipitation: { summary, updateTime, fxLink?, minutely: [{ time, precip, type }] }`

---

### 5 `weather_warning` 气象预警

**适用**：安全风险（暴雨、大风、高温、寒潮等）。  
**输入**
- `{ location? }`

**输出（成功）**
- `success: true`
- `warning: { updateTime, fxLink?, alerts: [...] }`

---

### 6 `weather_indices` 生活指数（1d / 3d）

**适用**：穿衣/洗车/运动/紫外线/感冒等建议。  
**输入**
- `{ location?, days?, type? }`
- `days: "1d" | "3d"`
- `type: "all"` 或具体指数类型

**输出（成功）**
- `success: true`
- `indices: [{ date, type, name, level, category, text }]`

---

## 使用示例（Usage Examples）

> 注意：示例只演示“天气 action 的调用方式”。  
> JWT 和 Geo 解析会在这些 action 内部自动完成。

### 示例 1：实时天气（含回退）

```js
const { weather_now } = require("{baseDir}/lib/qweather_weather_tool");

(async () => {
  const r = await weather_now({ location: "北京" });
  console.log(r);
})();
```
### 示例 2：日预报（7 天）
```js
const { weather_forecast } = require("{baseDir}/lib/qweather_weather_tool");

(async () => {
  const r = await weather_forecast({ location: "上海", days: 7 });
  console.log(r);
})();
```
###  示例 3：逐小时预报（72h）
```js
const { weather_hourly } = require("{baseDir}/lib/qweather_hourly_tool");

(async () => {
  const r = await weather_hourly({ location: "广州", hours: "72h" });
  console.log(r);
})();
```
###  示例 4：分钟级降水（2 小时）
```js
const { weather_minutely_precipitation } = require("{baseDir}/lib/qweather_precipitation_tool");

(async () => {
  const r = await weather_minutely_precipitation({ location: "杭州" });
  console.log(r);
})();
```
###  示例 5：气象预警
```js
const { weather_warning } = require("{baseDir}/lib/qweather_warning_tool");

(async () => {
  const r = await weather_warning({ location: "成都" });
  console.log(r);
})();
```
###  示例 6：生活指数（全部）
```js
const { weather_indices } = require("{baseDir}/lib/qweather_indices_tool");

(async () => {
  const r = await weather_indices({ location: "南京", days: "1d", type: "all" });
  console.log(r);
})();
```
## 失败与可靠性（Reliability Notes）

- weather_now：具备 Open-Meteo 回退，增强可用性（企业 API 异常时仍返回基础实时天气）。

- 其他 action：以企业 API 为主；失败时返回 success:false 并附带错误信息（不回退）。

- 分钟级降水：底层依赖经纬度；若无法从位置解析获得坐标，将返回失败。



## 配置与安全（Configuration & Security）

### 配置来源（固定文件，不由 OpenClaw 注入）

本 Skill 的企业 API 配置 **不通过 OpenClaw 的 env/apiKey 机制注入**，也不期望 OpenClaw 读取或管理该配置。

- 配置文件：`{baseDir}/lib/qweather_config.js`
- 私钥文件路径：由 `qweather_config.js` 内的 `PRIVATE_KEY_PATH` 指定（技能内部读取本地文件）

也就是说：部署者需要在技能目录中自行准备好配置与私钥文件，确保运行时可读。

### 安全建议

- 不要把私钥内容直接写入仓库（建议仅保存路径并通过部署流程落盘）
- 确保私钥文件权限最小化（仅运行该 Skill 的进程用户可读）
- 避免在日志中输出完整 JWT 或私钥内容（调试时仅在本地环境开启）
