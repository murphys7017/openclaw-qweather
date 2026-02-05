# 天气查询技能 (Weather Query Skill)

## 描述
一个全面的天气查询系统，支持实时天气、天气预报、降水预报、气象预警和生活指数查询。系统优先使用和风天气企业API，失败时自动回退到Open-Meteo。

## 功能特性

### 1. 实时天气查询
- 获取当前温度、湿度、风力等信息
- 支持多种位置输入格式（城市名、经纬度、位置ID）

### 2. 天气预报
- **每日预报**：支持3天、7天、15天天气预报
- **逐小时预报**：支持24小时、72小时、168小时预报
- **分钟级降水预报**：2小时内每5分钟降水预测

### 3. 气象预警
- 气象灾害预警信息
- 支持多种预警类型（大风、暴雨、高温等）

### 4. 生活指数
- 提供多种生活指导（穿衣、洗车、运动、紫外线、感冒等）
- 支持1天和3天预报

### 5. 自动化服务
- **每日天气报告**：每天早上8点发送综合天气报告
- **天气预警检查**：每天早上8点、下午2点、晚上6点检查并推送预警

## API支持

### 已实现的API端点
- `/v7/weather/now` - 实时天气
- `/v7/weather/{day}d` - 每日天气预报（3d/7d/15d）
- `/v7/weather/{hour}h` - 逐小时天气预报（24h/72h/168h）
- `/v7/minutely/5m` - 分钟级降水预报
- `/v7/warning/now` - 气象灾害预警
- `/v7/indices/{day}` - 生活指数（1d/3d）

## 配置

### 必需配置
- 位于 `skills/weather-query/lib/qweather_config.js`
- 配置项包括API主机、项目ID、凭证ID、私钥路径等

### 认证方式
- JWT认证（和风天气企业版）
- 私钥格式：Ed25519

## 使用方法

### 直接调用API函数
```javascript
const { weather_now, weather_forecast } = require('./skills/weather-query/lib/qweather_weather_tool.js');
const { weather_hourly } = require('./skills/weather-query/lib/qweather_hourly_tool.js');
const { weather_minutely_precipitation } = require('./skills/weather-query/lib/qweather_precipitation_tool.js');
const { weather_warning } = require('./skills/weather-query/lib/qweather_warning_tool.js');
const { weather_indices } = require('./skills/weather-query/lib/qweather_indices_tool.js');
```

### 主要函数
- `weather_now({ location })` - 实时天气
- `weather_forecast({ location, days })` - 天气预报
- `weather_hourly({ location, hours })` - 逐小时预报
- `weather_minutely_precipitation({ location })` - 分钟级降水
- `weather_warning({ location })` - 气象预警
- `weather_indices({ location, days, type })` - 生活指数

## 回退机制
- 优先使用和风天气企业API
- 企业API失败时自动回退到Open-Meteo
- 确保服务的高可用性

## 依赖
- axios
- jose (JWT处理)
- 和风天气企业API认证
- Open-Meteo API (回退)

## 任务调度
- 每日天气报告任务
- 定时预警检查任务