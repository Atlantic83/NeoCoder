# neoCoder Configuration Guide

This guide covers all aspects of configuring neoCoder, from initial setup to advanced customization.

## Quick Start

### Create New Configuration

```bash
# Create default configuration in ~/.neoCoder
neocoder \init-config

# Or use Python module
python -m neoCoder.config.init_config

# Create in custom location
python -m neoCoder.config.init_config /path/to/config
```

## Configuration Files

### Location

Configuration files are stored in `~/.neoCoder/` by default:

```
~/.neoCoder/
├── settings.json          # Main agent configuration
├── models.json            # Model definitions
├── tool_selection.json    # Tool selection settings
├── workflow_state.json    # Workflow state (auto-created)
└── notes/                 # Persistent notes
    └── README.md
```

You can override the location with the `NEOCODER_CONFIG` environment variable:

```bash
export NEOCODER_CONFIG=/custom/path/settings.json
```

### settings.json

Main configuration file with all agent settings:

```json
{
  "agent": {
    "name": "neocoder-agent",
    "description": "neoCoder AI Agent",
    "default_model": "anthropic/claude-sonnet-4.5",
    "max_iterations": 100,
    
    "anthropic": {
      "auth_token": null,
      "base_url": null,
      "streaming": true,
      "thinking_enabled": true,
      "thinking_budget": 10000
    },
    
    "zenmux": {
      "api_key": null,
      "base_url": "https://zenmux.ai/api/v1",
      "reasoning_enabled": true
    },
    
    "memory": {
      "max_context_length": 100000,
      "compression_threshold": 0.8,
      "rolling_window_size": 10
    },
    
    "specialized_agents": {
      "coder_model": "anthropic/claude-sonnet-4.5",
      "coder_temperature": 0.0,
      "coder_max_tokens": 8192,
      "architect_model": "anthropic/claude-sonnet-4.5",
      "architect_temperature": 0.0
    },
    
    "monitoring": {
      "enabled": true,
      "show_startup_info": true,
      "show_usage_updates": true,
      "display_format": "box"
    }
  }
}
```

### models.json

Model definitions with capabilities:

```json
{
  "models": {
    "anthropic/claude-opus-4": {
      "name": "Claude Opus 4",
      "provider": "anthropic",
      "max_context_tokens": 200000,
      "max_output_tokens": 16384,
      "reasoning_tokens": 100000,
      "reasoning_effort": "high",
      "supports_vision": true,
      "supports_tools": true,
      "supports_streaming": true
    },
    "anthropic/claude-sonnet-4.5": {
      "name": "Claude Sonnet 4.5",
      "provider": "anthropic",
      "max_context_tokens": 200000,
      "max_output_tokens": 16384,
      "reasoning_tokens": 100000,
      "reasoning_effort": "medium"
    }
  }
}
```

## API Configuration

### Using Anthropic API

Set your API key via environment variable:

```bash
export ANTHROPIC_API_KEY=your-key
# or
export ANTHROPIC_AUTH_TOKEN=your-key
```

Or in `settings.json`:

```json
{
  "agent": {
    "anthropic": {
      "auth_token": "your-key"
    }
  }
}
```

### Using ZenMux API

ZenMux provides unified access to multiple LLM providers:

```bash
export ZENMUX_API_KEY=your-key
```

Or in `settings.json`:

```json
{
  "agent": {
    "zenmux": {
      "api_key": "your-key",
      "base_url": "https://zenmux.ai/api/v1"
    }
  }
}
```

### Custom API Endpoint

Use a custom API endpoint:

```bash
export NEOCODER_BASE_URL=https://custom-api.com/v1
```

Or via CLI:

```bash
neocoder --base-url https://custom-api.com/v1 "your task"
```

## Configuration via CLI

### Presets

Use predefined configuration presets:

```bash
# Fast preset - quick responses
neocoder --preset fast "your task"

# Thorough preset - detailed analysis
neocoder --preset thorough "your task"

# Workflow preset - multi-stage workflow
neocoder --preset workflow "your task"

# ZenMux preset - optimized for ZenMux API
neocoder --preset zenmux "your task"
```

### Model Selection

Override the default model:

```bash
neocoder --model anthropic/claude-opus-4 "your task"
```

### Mode Selection

Choose agent mode:

```bash
# Coding mode (default)
neocoder --mode coding "create a new feature"

# Debug mode
neocoder --mode debug "fix the failing test"

# Refactor mode
neocoder --mode refactor "improve code structure"
```

## Advanced Configuration

### Memory Management

Control context window and compression:

```json
{
  "agent": {
    "memory": {
      "max_context_length": 100000,
      "compression_enabled": true,
      "compression_threshold": 0.8,
      "rolling_window_size": 10
    }
  }
}
```

### Specialized Agents

Configure different models for different tasks:

```json
{
  "agent": {
    "specialized_agents": {
      "coder_model": "anthropic/claude-sonnet-4.5",
      "coder_temperature": 0.0,
      "coder_max_tokens": 8192,
      "coder_thinking_budget": 10000,
      
      "architect_model": "anthropic/claude-opus-4",
      "architect_temperature": 0.7,
      "architect_max_tokens": 4096,
      "architect_thinking_budget": 30000
    }
  }
}
```

### Monitoring

Control monitoring and display:

```json
{
  "agent": {
    "monitoring": {
      "enabled": true,
      "show_startup_info": true,
      "show_usage_updates": true,
      "display_format": "box",
      "cache_ttl": 3600,
      "api_timeout": 5.0
    }
  }
}
```

## Configuration via Python

### Using ConfigInitializer

```python
from pathlib import Path
from neoCoder.config.init_config import ConfigInitializer

# Create example config
config_dir = ConfigInitializer.create_example_config()
print(f"Created config at: {config_dir}")

# Create in custom location
custom_dir = Path("/custom/path")
ConfigInitializer.create_example_config(custom_dir)

# Get example settings dict
settings = ConfigInitializer.get_example_settings()
models = ConfigInitializer.get_example_models()
```

### Loading Settings

```python
from neoCoder.config import load_settings, get_settings

# Load settings
settings = load_settings()

# Access configuration
api_key = settings.agent_config.anthropic.auth_token
model = settings.agent_config.default_model

# Get specialized agent config
model, temp, max_tokens, thinking_budget = settings.get_specialized_agent_config("coder")
```

## Troubleshooting

### Configuration Not Found

If neoCoder can't find your configuration:

1. Check the default location: `~/.neoCoder/settings.json`
2. Verify file permissions
3. Try creating a new config: `neocoder \init-config`
4. Set custom location: `export NEOCODER_CONFIG=/path/to/settings.json`

### API Key Issues

If API calls fail:

1. Verify API key is set:
   ```bash
   echo $ANTHROPIC_API_KEY
   echo $ZENMUX_API_KEY
   ```

2. Check settings.json has the key:
   ```bash
   cat ~/.neoCoder/settings.json | grep auth_token
   ```

3. Test with explicit key:
   ```bash
   neocoder --api-key your-key "test task"
   ```

## Best Practices

1. **Use environment variables for API keys** - Don't commit keys to version control
2. **Start with default config** - Use `neocoder init-config` for a working baseline
3. **Use presets for common scenarios** - Faster than manual configuration
4. **Enable monitoring** - Helps track token usage and costs
5. **Customize specialized agents** - Use different models for different tasks
6. **Keep notes directory** - Persistent learning across sessions

## See Also

- [Quick Start Guide](../README.md)
- [CLI Reference](CLI.md)
- [Workflow Guide](WORKFLOW.md)
- [API Documentation](../api/)
