---
date: "2025-12-17"
title: "ty: An extremely fast Python type checker and LSP"
tags: ["python"]
summary: "ty: An extremely fast Python type checker and LSP"
slug: ty-python-beta
---

[ty: An extremely fast Python type checker and LSP](https://astral.sh/blog/ty): Charlie Marsh from the team at [Astral](https://astral.sh) recently announced the Beta release of `ty`. Included in the announcement is an [LSP](https://docs.astral.sh/ty/features/language-server/) and [VS Code Plugin](https://marketplace.visualstudio.com/items?itemName=astral-sh.ty) (which ships with `ty` included).

There are a variety of supported [installation methods](https://docs.astral.sh/ty/installation/), however if you're using [`uv`](https://docs.astral.sh/uv/) for managing your Python project, the easiest way to start is by adding `ty` as a development dependency with `uv add --dev ty`. Then run `ty` against your codebase with `uv run ty`.

In the past few months, I've found myself spending more time writing Python than Go, prompting me to search for new developer tools in the Python ecosystem. There are some real gems of well-designed developer tools that have been created since I was last actively developing in Python as my primary language. Several of these tools have been created by the folks at Astral, which makes sense since [according to their about page](https://astral.sh/about):

> "Our early team includes the authors of [ripgrep](https://github.com/BurntSushi/ripgrep), [bat](https://github.com/sharkdp/bat), [hyperfine](https://github.com/sharkdp/hyperfine), and [maturin](https://maturin.rs/); early, core contributors to [Biome](https://github.com/biomejs/biome) and [Prefect](https://github.com/PrefectHQ/prefect/); and multiple CPython core developers — all building at the intersection of Rust and Python".

I'll be updating my personal Python project template soon, and I expect that it will include many of the Astral team's tools such as [`uv`](https://docs.astral.sh/uv/), [`ruff`](https://docs.astral.sh/ruff/) and now [`ty`](https://docs.astral.sh/ty/).