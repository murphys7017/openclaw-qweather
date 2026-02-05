# openclaw-qweather Weather Query Skill

## Project Overview

This project is a weather query skill for the OpenClaw platform, integrating the QWeather (HeWeather) Enterprise API. It supports real-time weather, weather forecasts, minute-level precipitation, weather warnings, and lifestyle indices, with a high-availability fallback to Open-Meteo.

## Features

- **Real-time Weather**: Supports multiple location formats (city name, coordinates, ID), provides current temperature, humidity, wind, etc.
- **Weather Forecast**: 3/7/15-day daily forecasts, 24/72/168-hour hourly forecasts.
- **Minute-level Precipitation**: 5-minute interval precipitation prediction for the next 2 hours.
- **Weather Warnings**: Multiple types of weather disaster alerts.
- **Lifestyle Indices**: Clothing, car wash, sports, UV, cold risk, and more.
- **Automation**: Scheduled daily weather reports and warning notifications.
- **High Availability**: Automatic fallback to Open-Meteo if the enterprise API fails.

## Directory Structure

- `lib/qweather_config.js`: API configuration (host, project ID, credential ID, private key path, etc.)
- `lib/qweather_jwt_session.js`: JWT generation and caching
- `lib/qweather_geo_tool.js`: Location resolution
- `lib/qweather_weather_tool.js`: Real-time weather / daily forecast
- `lib/qweather_hourly_tool.js`: Hourly forecast
- `lib/qweather_precipitation_tool.js`: Minute-level precipitation
- `lib/qweather_warning_tool.js`: Weather warnings
- `lib/qweather_indices_tool.js`: Lifestyle indices
- `lib/weather_now_openmeteo.js`: Open-Meteo fallback weather query
- `lib/ed25519-private.txt`: Ed25519 private key (generate yourself)

## Quick Start

1. **Configure QWeather Enterprise API**  
   Edit `lib/qweather_config.js` and fill in API Host, Project ID, Credential ID, Private Key Path, etc.

2. **Generate Ed25519 Private Key**  
   ```
   openssl genpkey -algorithm ED25519 -out ed25519-private.pem
   openssl pkey -pubout -in ed25519-private.pem > ed25519-public.pem
   ```
   Save the private key content as `lib/ed25519-private.txt`.

3. **Install Dependencies**
   ```bash
   npm install axios jose
   ```

4. **Usage Example**
   ```js
   const { weather_now } = require('./lib/qweather_weather_tool');
   weather_now({ location: "Beijing" }).then(console.log);
   ```

## Main API Functions

- `weather_now({ location })` - Real-time weather
- `weather_forecast({ location, days })` - Weather forecast
- `weather_hourly({ location, hours })` - Hourly forecast
- `weather_minutely_precipitation({ location })` - Minute-level precipitation
- `weather_warning({ location })` - Weather warnings
- `weather_indices({ location, days, type })` - Lifestyle indices

## Dependencies

- axios
- jose (for JWT, partial)
- Node.js 18+ (Ed25519 support required)

## Task Scheduling Suggestions

- Daily weather report (e.g., 8:00 AM)
- Scheduled weather warning checks (e.g., 8:00, 14:00, 18:00)

## License

For learning and personal development only. For commercial use, please comply with QWeather and Open-Meteo API terms.
