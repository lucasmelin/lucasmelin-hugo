---
date: "2025-12-22"
title: "Configuring the pytest import mechanism via pyproject"
tags: ["python", "til"]
summary: "While working on refreshing my cookiecutter-pyproject repository, I ran into a ModuleNotFoundError when running my tests with pytest. Here's how I fixed it."
slug: pytest-pyproject
---

While working on refreshing my [cookiecutter-pyproject](https://github.com/lucasmelin/cookiecutter-pyproject) repository, I ran into a `ModuleNotFoundError` when running my tests with `uv run pytest`. My project directory looks like this:

```text
my-project/
├─ docs/
├─ src/
│  └── my_project/
│     ├─ __init__.py
│     └─ main.py
├─ tests/
│  ├─ __init__.py
│  └─ test_my_project.py
├─ pyproject.toml
└─ README.md
```

and the contents of `test_my_project.py` look like this:

```python
"""Tests for my_project package."""
import my_project

def test_version():
    """Verify the package version."""
    assert my_project.__version__ == "0.1.0"
```

My suspicion was that the `src` directory wasn't being searched and imported by [pytest](https://docs.pytest.org/en/stable/index.html), even though both my module and my test directory had `__init__.py` files.

After looking at the [pytest docs](https://docs.pytest.org/en/stable/explanation/pythonpath.html) explanation as to how `pytest` imports files, the fix I went with was to set the `pythonpath` value in my `pyproject.toml` file like so:

```toml
[tool.pytest.ini_options]
pythonpath = [
  "src"
]
```

After that, re-running `uv run pytest` from my project's root directory was successful!