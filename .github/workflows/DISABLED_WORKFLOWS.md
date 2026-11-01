# 本 fork 禁用的上游 workflow

从 `waditu/czsc` 同步时，以下 workflow 被**有意移除**：

| 文件 | 上游用途 | 为什么在 fork 里移除 |
|---|---|---|
| `code-quality.yml` | Rust 逐 crate 测试 + 六平台 Python 矩阵 + 安全扫描 | 需要完整 Rust 工具链，单次 25–30 分钟，fork 里只烧额度 |
| `python-publish.yml` | Trusted Publishing 到 PyPI | 绑定的是 `waditu/czsc` 的 OIDC 身份，fork 里无权限，必失败 |
| `rust-publish.yml` | 发布到 crates.io | 同上，需要 `waditu` 的 token |
| `claude.yml` | issue 里 `@claude` 触发 AI 助手 | 需要 Anthropic API key |

## 只保留的 workflow

- `sync-upstream.yml` —— 每周一自动跟上 `waditu/czsc`

## 如果上游改动了这些文件

`sync-upstream.yml` 的 `git merge` 会遇到 **modify/delete 冲突**（上游改了，本地删了）。

两种情况：

**A. 想继续删除**（推荐）
```bash
git rm .github/workflows/code-quality.yml
git commit -m "keep workflow disabled"
git push
```

**B. 想恢复上游版本**
```bash
git checkout --theirs .github/workflows/
git add .github/workflows/
git commit -m "restore upstream workflows"
```

恢复后记得在 Settings → Actions 里手动禁用，否则每次 push 都会跑。

## 想在 fork 里跑测试怎么办

不要启用上游的完整矩阵，按需加一个轻量 workflow：

```yaml
name: quick-check
on: [push, pull_request]
jobs:
  py:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with: { python-version: '3.11' }
      - run: pip install numpy pandas loguru deprecated
      - run: python -c "import ast,pathlib; [ast.parse(p.read_text()) for p in pathlib.Path('czsc').rglob('*.py')]"
```

只做语法检查，几秒钟，不碰 Rust。
