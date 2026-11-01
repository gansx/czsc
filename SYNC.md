# czsc fork 自动同步说明

Fork 自 [waditu/czsc](https://github.com/waditu/czsc)，每周自动跟上上游。

## 同步配置

| 项 | 值 |
|---|---|
| 上游 | `waditu/czsc` (master) |
| 本仓库 | `gansx/czsc` (master) |
| 频率 | 每周一 **02:00 UTC** = 北京时间 **10:00** |
| 方式 | `git merge upstream/master`（保留双方历史） |
| 冲突处理 | 自动回滚 + 开 issue（label `upstream-sync`） |
| 手动触发 | Actions → "Sync upstream (waditu/czsc)" → Run workflow |

## 手动同步

```bash
# 方式 1：GitHub CLI
gh workflow run sync-upstream.yml --repo gansx/czsc

# 方式 2：本地 merge
git fetch upstream && git merge upstream/master && git push
```

## 上游 workflow 已被移除

见 [.github/workflows/DISABLED_WORKFLOWS.md](.github/workflows/DISABLED_WORKFLOWS.md)。
`code-quality` / `python-publish` / `rust-publish` / `claude` 四个在 fork 里缺 token 和 Rust 工具链，跑必失败。

## 两个需要注意的点

**1. 60 天无活动会暂停定时任务**

GitHub 的政策：仓库连续 60 天没有 commit / PR 活动，scheduled workflow 会被自动暂停。
上游停更时就会触发。恢复方式：Actions 页面点 "Enable workflow"，或手动跑一次。

**2. 想改代码就开分支**

`master` 保持纯粹跟随上游。自己的修改放功能分支：

```bash
git checkout -b my-feature
# 改代码
git push -u origin my-feature
```

这样 `master` 永远不会和上游冲突。如果在 `master` 上改，每周同步可能撞车。

## 本地开发

这个 fork 只做代码镜像，不改上游逻辑。要用它做研究：

```bash
git clone https://github.com/gansx/czsc.git
cd czsc
pip install -e .          # 需要 Rust 工具链
# 或只用纯 Python 版分析：
git checkout v0.9.69
```

`v0.9.69` 是 1.0 迁 Rust 之前的最后纯 Python 版本，分型/笔/中枢全在 `czsc/analyze.py` + `objects.py`。
