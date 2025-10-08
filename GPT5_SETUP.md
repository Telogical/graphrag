# GPT-5 Support in GraphRAG

This document explains how to use GPT-5 with your GraphRAG installation.

## What Was Fixed

As of this update, GraphRAG now supports GPT-5 and other newer language models. There were two main issues preventing GPT-5 from working:

### Issue #1: Tiktoken Model Recognition

Previously, when trying to use a model name that tiktoken didn't recognize (like "gpt-5"), the system would crash with a `KeyError`.

**Fix Applied:** Modified `/graphrag/config/models/language_model_config.py` in the `_validate_encoding_model()` method. Now, when tiktoken doesn't recognize a model name, the system:

1. Catches the `KeyError` exception
2. Logs a warning message
3. Falls back to the `cl100k_base` encoding (used by GPT-4 and newer OpenAI models)

### Issue #2: Null Parameter Rejection (GitHub Issue #2023)

GPT-5 rejects API calls that include `null` (None) values for parameters like `max_tokens`, while GPT-4 accepts them. This was causing the error:

```
Error code: 400 - {'error': {'message': "Invalid type for 'max_tokens': expected an unsupported value, but got null instead.", ...}}
```

**Fix Applied:** Modified both:
- `/graphrag/language_model/providers/litellm/chat_model.py`
- `/graphrag/language_model/providers/litellm/embedding_model.py`

Added filtering to remove `None` values from API parameters before sending requests. This ensures that only parameters with actual values are sent to the API, preventing GPT-5 from rejecting the requests.

These fixes allow GraphRAG to work with GPT-5 and any future OpenAI models without requiring code changes.

## How to Use GPT-5

### Option 1: Using LiteLLM (Recommended)

Create or update your `settings.yaml` file:

```yaml
models:
  default_chat_model:
    type: chat
    auth_type: api_key
    api_key: ${GRAPHRAG_API_KEY}
    model_provider: openai
    model: gpt-5  # or gpt-5-turbo, gpt-5-preview, etc.
    model_supports_json: true
    temperature: 0
    max_tokens: 4000
    requests_per_minute: 50
    tokens_per_minute: 100000

  default_embedding_model:
    type: embedding
    auth_type: api_key
    api_key: ${GRAPHRAG_API_KEY}
    model_provider: openai
    model: text-embedding-3-large
```

### Option 2: Using Azure OpenAI

If you're using Azure OpenAI with a GPT-5 deployment:

```yaml
models:
  default_chat_model:
    type: chat
    auth_type: api_key
    api_key: ${AZURE_OPENAI_API_KEY}
    model_provider: azure
    model: gpt-5
    deployment_name: your-gpt5-deployment-name
    api_base: https://your-resource.openai.azure.com/
    api_version: 2024-08-01-preview
    model_supports_json: true
```

### Option 3: Using Legacy fnllm Provider

While not recommended (deprecated in favor of LiteLLM), you can still use the older fnllm provider:

```yaml
models:
  default_chat_model:
    type: openai_chat
    auth_type: api_key
    api_key: ${GRAPHRAG_API_KEY}
    model: gpt-5
    model_supports_json: true
```

**Note:** You'll see a deprecation warning when using this approach.

## Environment Setup

1. Create a `.env` file in your project root:

```bash
GRAPHRAG_API_KEY=sk-your-openai-api-key-here
```

2. Or export the environment variable:

```bash
export GRAPHRAG_API_KEY=sk-your-openai-api-key-here
```

## Example Configuration File

See `examples/gpt5-settings.yaml` for a complete example configuration.

## Testing Your Configuration

You can test your GPT-5 configuration by running:

```bash
graphrag init --config settings.yaml
graphrag index --config settings.yaml
```

## Troubleshooting

### "Model 'gpt-5' not recognized by tiktoken"

This warning appears if you're using tiktoken 0.11.0. The system will automatically fall back to `cl100k_base` encoding, which works fine for GPT-5.

**Recommended solution:** Upgrade to tiktoken 0.12.0 or higher, which includes native GPT-5 support:
```bash
pip install --upgrade "tiktoken>=0.12.0"
```

With tiktoken 0.12.0+, GPT-5 is natively recognized and this warning won't appear.

### Error: "Invalid type for 'max_tokens': expected an unsupported value, but got null instead"

**This error has been fixed!** If you're still seeing it, make sure you have the latest version with the fix applied to:
- `graphrag/language_model/providers/litellm/chat_model.py`
- `graphrag/language_model/providers/litellm/embedding_model.py`

The fix filters out `None` values from API parameters. If you still see the error after updating, try explicitly setting `max_tokens` in your configuration:

```yaml
models:
  default_chat_model:
    max_tokens: 4000  # Or remove this line entirely to let it be filtered
```

### API Key Issues

Make sure your API key has access to GPT-5. You may need to:
- Check your OpenAI account's model access permissions
- Verify you're using the correct API key
- For Azure, ensure your deployment name matches your Azure resource

### Rate Limiting

GPT-5 may have different rate limits than GPT-4. Adjust these parameters in your config:

```yaml
requests_per_minute: 50  # Adjust based on your quota
tokens_per_minute: 100000  # Adjust based on your quota
concurrent_requests: 25  # Lower if you hit rate limits
```

### Still Having Issues?

If you continue to experience problems:

1. **Check your GraphRAG version:** Ensure you're running the version with GPT-5 fixes
2. **Verify API access:** Confirm your API key has GPT-5 access via OpenAI's API
3. **Enable debug logging:** Add `GRAPHRAG_LOG_LEVEL=DEBUG` to your `.env` file
4. **Check LiteLLM compatibility:** Make sure you're using `type: chat` (LiteLLM) not `type: openai_chat` (deprecated)

## Technical Details

### Fix #1: Encoding Model Fallback

When you use GPT-5, the system uses the following logic:

1. For LiteLLM configurations (`type: chat` or `type: embedding`), no tiktoken encoding is needed - LiteLLM handles tokenization automatically
2. For legacy fnllm configurations (`type: openai_chat`), the system tries to get the encoding from tiktoken
3. If tiktoken doesn't recognize the model, it falls back to `cl100k_base`

The `cl100k_base` encoding is used by:
- gpt-4
- gpt-4-turbo
- gpt-4o
- gpt-5 (natively supported in tiktoken 0.12.0+)

**Important:** This fix includes a fallback mechanism for backward compatibility, but we **recommend upgrading to tiktoken 0.12.0 or higher** for native GPT-5 support. As of [tiktoken 0.12.0](https://github.com/openai/tiktoken/commit/97e49cbadd500b5cc9dbb51a486f0b42e6701bee), GPT-5 is officially recognized and you won't see the fallback warning.

To upgrade tiktoken:
```bash
pip install --upgrade tiktoken
# or with uv:
uv pip install --upgrade tiktoken
```

### Fix #2: Null Parameter Filtering

GPT-5's API is stricter about parameter validation than GPT-4. It rejects requests that include parameters with `null` values, particularly for `max_tokens`.

**The Problem:**
- GraphRAG's default configuration may not set `max_tokens`, leaving it as `None`
- When passed to the API as `null`, GPT-5 returns a 400 error
- GPT-4 accepted `null` values, so this wasn't an issue before

**The Solution:**
Before sending requests to the API, the system now filters out all parameters with `None` values:

```python
# Remove None values - some models (like GPT-5) don't accept null parameters
base_args = {k: v for k, v in base_args.items() if v is not None}
```

This ensures only parameters with actual values are sent, making the system compatible with both strict APIs (GPT-5) and lenient ones (GPT-4).

**Parameters Affected:**
- `max_tokens`
- `max_completion_tokens`
- `reasoning_effort`
- `api_base`, `api_version`, `organization`, `proxy`, `audience` (when not set)
- And any other optional parameters that default to `None`

## Further Reading

- [GraphRAG Language Model Configuration](docs/config/models.md)
- [LiteLLM Documentation](https://docs.litellm.ai/)
- [OpenAI API Documentation](https://platform.openai.com/docs/api-reference)

