# openclaw-qweather 天气查询技能

## 项目简介

本项目为 OpenClaw 平台开发的天气查询 Skill，集成了和风天气企业 API，支持实时天气、天气预报、分钟级降水、气象预警、生活指数等多种查询能力，并具备自动回退到 Open-Meteo 的高可用机制。

## 主要特性

- **实时天气**：支持多种位置格式（城市名、经纬度、ID），获取当前温度、湿度、风力等。
- **天气预报**：支持3/7/15天日预报，24/72/168小时逐小时预报。
- **分钟级降水**：2小时内每5分钟降水预测。
- **气象预警**：多类型气象灾害预警。
- **生活指数**：穿衣、洗车、运动、紫外线等多种生活建议。
- **自动化服务**：定时推送每日天气报告和气象预警。
- **高可用回退**：企业API异常时自动切换到 Open-Meteo。

## 目录结构

- `lib/qweather_config.js`：API 配置（主机、项目ID、凭证ID、私钥路径等）
- `lib/qweather_jwt_session.js`：JWT 生成与缓存
- `lib/qweather_geo_tool.js`：位置解析
- `lib/qweather_weather_tool.js`：实时天气/日预报
- `lib/qweather_hourly_tool.js`：逐小时预报
- `lib/qweather_precipitation_tool.js`：分钟级降水
- `lib/qweather_warning_tool.js`：气象预警
- `lib/qweather_indices_tool.js`：生活指数
- `lib/weather_now_openmeteo.js`：Open-Meteo 备选天气查询
- `lib/ed25519-private.txt`：Ed25519 私钥（需自行生成）

## 快速开始

1. **配置和风天气企业API参数**  
   编辑 `lib/qweather_config.js`，填写 API Host、项目ID、凭证ID、私钥路径等。

2. **生成 Ed25519 私钥**  
   ```
   openssl genpkey -algorithm ED25519 -out ed25519-private.pem
   openssl pkey -pubout -in ed25519-private.pem > ed25519-public.pem
   ```
   将私钥内容保存为 `lib/ed25519-private.txt`。

3. **安装依赖**
   ```bash
   npm install axios jose
   ```

4. **调用示例**
   ```js
   const { weather_now } = require('./lib/qweather_weather_tool');
   weather_now({ location: "北京" }).then(console.log);
   ```

## 主要API函数

- `weather_now({ location })` - 实时天气
- `weather_forecast({ location, days })` - 天气预报
- `weather_hourly({ location, hours })` - 逐小时预报
- `weather_minutely_precipitation({ location })` - 分钟级降水
- `weather_warning({ location })` - 气象预警
- `weather_indices({ location, days, type })` - 生活指数

## 依赖

- axios
- jose (仅部分 JWT 相关脚本)
- Node.js 18+ (需支持 Ed25519)

## 任务调度建议

- 每日天气报告（如每天8点）
- 定时气象预警检查（如8点、14点、18点）

## 许可证

仅供学习与个人开发使用，商用请遵守和风天气及 Open-Meteo API 协议。
