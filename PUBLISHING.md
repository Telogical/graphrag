# Publishing Your GraphRAG Fork to PyPI

This guide shows how to publish your GPT-5-enabled GraphRAG fork to PyPI so anyone can install it with:
```bash
pip install graphrag-gpt5  # or whatever name you choose
```

## Prerequisites

1. PyPI account: https://pypi.org/account/register/
2. TestPyPI account (for testing): https://test.pypi.org/account/register/
3. Install build tools:
   ```bash
   pip install build twine
   ```

## Step 1: Update Package Name (Important!)

Since "graphrag" is already taken on PyPI by Microsoft, you need to rename your fork:

**Edit `pyproject.toml`:**
```toml
[project]
name = "graphrag-gpt5"  # or "graphrag-thirdcreed" or "graphrag-telogical"
version = "2.6.0-gpt5.1"  # Add your own version suffix
description = "GraphRAG with GPT-5 support - A graph-based RAG system"
```

Also update the GitHub URL:
```toml
[project.urls]
Source = "https://github.com/thirdcreed/graphrag"
```

## Step 2: Build the Distribution

```bash
# Clean any old builds
rm -rf dist/ build/ *.egg-info

# Build wheel and source distribution
python -m build
```

This creates:
- `dist/graphrag_gpt5-2.6.0.gpt5.1-py3-none-any.whl`
- `dist/graphrag-gpt5-2.6.0-gpt5.1.tar.gz`

## Step 3: Test on TestPyPI First

```bash
# Upload to TestPyPI
python -m twine upload --repository testpypi dist/*

# Test installation from TestPyPI
pip install --index-url https://test.pypi.org/simple/ graphrag-gpt5
```

## Step 4: Publish to PyPI

```bash
# Upload to real PyPI
python -m twine upload dist/*
```

## Step 5: Install Your Published Package

Now anyone can install it:
```bash
pip install graphrag-gpt5
```

## Alternative: Using GitHub Releases

Instead of PyPI, you can use GitHub Releases:

1. Create a release on GitHub with your wheel file
2. Users install via:
   ```bash
   pip install https://github.com/thirdcreed/graphrag/releases/download/v2.6.0-gpt5/graphrag_gpt5-2.6.0-py3-none-any.whl
   ```

## Recommended Package Names

Since "graphrag" is taken, consider:
- `graphrag-gpt5` - Clear GPT-5 focus
- `graphrag-telogical` - Your org name
- `graphrag-enhanced` - Generic enhancement
- `graphrag-fork` - Simple fork indicator

## Maintaining Your Fork

After publishing:

1. **Version bumping:** Update version in `pyproject.toml` for each release
2. **Changelog:** Document changes in `CHANGELOG.md`
3. **Tagging:** Create git tags for releases:
   ```bash
   git tag -a v2.6.0-gpt5.1 -m "GPT-5 support release"
   git push origin v2.6.0-gpt5.1
   ```

## Example Complete Workflow

```bash
# 1. Update version in pyproject.toml
# 2. Build
python -m build

# 3. Test locally
pip install dist/graphrag_gpt5-2.6.0.gpt5.1-py3-none-any.whl

# 4. Upload to PyPI
python -m twine upload dist/*

# 5. Tag release
git tag -a v2.6.0-gpt5.1 -m "Add GPT-5 support"
git push origin v2.6.0-gpt5.1

# 6. Create GitHub release with the wheel file
```

## Users Install Your Package

```bash
# From PyPI (if you publish)
pip install graphrag-gpt5

# From GitHub (always works)
pip install git+https://github.com/thirdcreed/graphrag.git

# With specific version
pip install git+https://github.com/thirdcreed/graphrag.git@v2.6.0-gpt5.1
```

## Notes

- **PyPI requires unique name** - You cannot use "graphrag" (Microsoft owns it)
- **GitHub install always works** - No need to publish to PyPI
- **Consider trademark** - Be respectful of Microsoft's GraphRAG name
- **Update documentation** - Make it clear this is a fork with GPT-5 support
