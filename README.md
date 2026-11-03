# MCP Weather Forecast

A Python [Model Context Protocol (MCP)](https://modelcontextprotocol.io/) server that provides US weather forecasts and active alerts using the [National Weather Service (NWS) API](https://www.weather.gov/documentation/services-web-api). Connect it to an MCP client to ask about the weather in natural language.

The server runs locally over standard input/output (`stdio`). No API key is required; an internet connection is needed to retrieve weather data.

## Requirements

- Python 3.14 or newer
- [uv](https://docs.astral.sh/uv/)
- An MCP client that supports local `stdio` servers

## Setup

From the project directory, install the locked dependencies:

```sh
uv sync --locked
```

Start the server directly with:

```sh
uv run python weather.py
```

This starts an MCP process that waits for protocol messages on standard input. It does not start a web server or an interactive command prompt.

## Connect an MCP client

For clients that use an `mcpServers` configuration object, add the following entry, replacing the paths with absolute paths on your machine:

```json
{
  "mcpServers": {
    "weather": {
      "command": "/absolute/path/to/uv",
      "args": [
        "--directory",
        "/absolute/path/to/mcp_weather_forecast",
        "run",
        "python",
        "weather.py"
      ]
    }
  }
}
```

Find the `uv` executable with `command -v uv` on macOS/Linux or `where uv` on Windows. Restart or reconnect your MCP client after updating its configuration. Configuration file locations and formats depend on the client.

Example prompts once connected:

- “Are there any active weather alerts in California?”
- “What's the forecast for latitude 37.7749, longitude -122.4194?”

## Available tools

### `get_alerts`

Retrieves active weather alerts for a US state.

| Parameter | Type | Description |
| --- | --- | --- |
| `state` | string | Two-letter US state code, such as `CA` or `NY`. |

Example arguments:

```json
{"state": "CA"}
```

Returns alert event, affected area, severity, description, and instructions. Multiple alerts are separated by `---`. If the state has no active alerts, the tool returns `No active alerts for this state.`

### `get_forecast`

Retrieves a forecast for coordinates supported by the NWS API.

| Parameter | Type | Description |
| --- | --- | --- |
| `latitude` | number | Latitude in decimal degrees. |
| `longitude` | number | Longitude in decimal degrees. |

Example arguments:

```json
{"latitude": 37.7749, "longitude": -122.4194}
```

Returns up to the next five forecast periods, each including its name, temperature and unit, wind speed and direction, and detailed forecast. These are NWS forecast periods, not necessarily five days.

## Behavior and limitations

- Weather coverage is limited to locations supported by the US National Weather Service; this is not a global weather service.
- Each tool call fetches fresh data from `https://api.weather.gov`; there is no local cache.
- HTTP requests use a 30-second timeout per request. Forecast retrieval makes two sequential requests: one to resolve the location's forecast endpoint and another to retrieve the forecast.
- Failed requests produce a readable error message. Check your internet connection, state code or coordinates, and NWS availability if a tool cannot fetch data.
- The request user agent is configured by `USER_AGENT` in `weather.py`.

## Project layout

```text
weather.py                           # MCP server and weather tools
pyproject.toml                       # Package metadata and dependencies
uv.lock                              # Locked dependency versions
.python-version                      # Project Python version
src/mcp_weather_forecast/__init__.py  # Placeholder package entry point
```

The implemented server entry point is `weather.py`. The `mcp-weather-forecast` console command currently runs the placeholder greeting in the package; use `uv run python weather.py` to launch the weather server.
