# Installing GraphRAG with GPT-5 Support

This fork of Microsoft's GraphRAG includes GPT-5 support and fixes for issue #2023.

## Quick Install

### From GitHub (Recommended - No PyPI account needed)

```bash
pip install git+https://github.com/thirdcreed/graphrag.git
```

### With Specific Version/Tag

```bash
# Install from a specific branch
pip install git+https://github.com/thirdcreed/graphrag.git@main

# Install from a specific tag
pip install git+https://github.com/thirdcreed/graphrag.git@v2.6.0-gpt5

# Install from a specific commit
pip install git+https://github.com/thirdcreed/graphrag.git@abc123
```

### For Development (Editable Install)

```bash
git clone https://github.com/thirdcreed/graphrag.git
cd graphrag
pip install -e .
```

## Using in Other Projects

### Add to requirements.txt

```txt
# Basic install
git+https://github.com/thirdcreed/graphrag.git

# With specific branch
git+https://github.com/thirdcreed/graphrag.git@main

# With egg name (useful for dependency resolution)
git+https://github.com/thirdcreed/graphrag.git#egg=graphrag
```

### Add to pyproject.toml

```toml
[project]
dependencies = [
    "graphrag @ git+https://github.com/thirdcreed/graphrag.git",
]
```

### Add to setup.py

```python
install_requires=[
    "graphrag @ git+https://github.com/thirdcreed/graphrag.git",
]
```

### Using with Poetry

```bash
poetry add git+https://github.com/thirdcreed/graphrag.git
```

Or in `pyproject.toml`:
```toml
[tool.poetry.dependencies]
graphrag = { git = "https://github.com/thirdcreed/graphrag.git", branch = "main" }
```

### Using with pip-tools

In `requirements.in`:
```txt
graphrag @ git+https://github.com/thirdcreed/graphrag.git
```

Then:
```bash
pip-compile requirements.in
pip-sync
```

## Verifying Installation

After installation, verify GPT-5 support:

```python
import tiktoken
print(f"Tiktoken version: {tiktoken.__version__}")  # Should be >= 0.12.0

from graphrag.config.models.language_model_config import LanguageModelConfig
from graphrag.config.enums import ModelType, AuthType

# Test GPT-5 config
config = LanguageModelConfig(
    type=ModelType.Chat,
    model="gpt-5",
    model_provider="openai",
    api_key="test-key",
    auth_type=AuthType.APIKey,
)
print(f"✅ GPT-5 config created successfully")
```

## What's Included

This fork includes:

- ✅ **Tiktoken 0.12.0+** - Native GPT-5 recognition
- ✅ **Encoding fallback** - Handles unknown model names
- ✅ **Null parameter filtering** - Fixes GitHub issue #2023
- ✅ **Complete documentation** - GPT5_SETUP.md guide
- ✅ **Example configs** - examples/gpt5-settings.yaml

## Upgrading from Official GraphRAG

```bash
# Uninstall official version
pip uninstall graphrag

# Install this fork
pip install git+https://github.com/thirdcreed/graphrag.git
```

Your existing configurations will work - this is backward compatible!

## Troubleshooting

### Import Error

If you get import errors, make sure you've uninstalled the official package:
```bash
pip uninstall graphrag -y
pip install git+https://github.com/thirdcreed/graphrag.git
```

### Dependency Conflicts

If you have dependency conflicts:
```bash
pip install --upgrade git+https://github.com/thirdcreed/graphrag.git
```

### SSH vs HTTPS

If you have SSH keys set up with GitHub:
```bash
pip install git+ssh://git@github.com/thirdcreed/graphrag.git
```

## For Your Fabric Script

Update your `check_and_install_graphrag()` function:

```python
def check_and_install_graphrag():
    """Check and install GraphRAG with GPT-5 support"""
    packages_to_check = [
        ("graphrag", ["git+https://github.com/thirdcreed/graphrag.git"]),
        ("azure.storage.blob", ["azure-storage-blob"]),
        ("azure.identity", ["azure-identity"])
    ]
    # ... rest of your function
```

## Support

- **Documentation**: See `GPT5_SETUP.md` for detailed GPT-5 setup
- **Issues**: Open issues on GitHub
- **Original Project**: https://github.com/microsoft/graphrag
