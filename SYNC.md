# czsc fork 分支说明与自动同步

Fork 自 [waditu/czsc](https://github.com/waditu/czsc)。

## 分支模型

```
upstream/master  (waditu/czsc)
      │  每周一 02:00 UTC 自动 merge
      ▼
   master        ← 纯镜像，不在这里改代码
      │  自动 merge（workflow_run 触发）
      ▼
    dev          ← 你的开发分支，随便改
```

| 分支 | 用途 | 谁来更新 |
|---|---|---|
| `master` | 上游镜像，保持与 `waditu/czsc` 一致 | `sync-upstream.yml` 自动 |
| `dev` | 开发分支，写自己的东西 | `sync-dev.yml` 自动跟 master |
| `v0.9.69` (tag) | **纯 Python 版**，算法研究用 | 不变 |

## 三个基线的选择

### 1. `master` —— 最新 1.0.x（Rust）

- 分型/笔/中枢在 `crates/czsc-core` 的 Rust 里
- 改算法要重新编译（`maturin develop --release`）
- 适合：用最新功能、要性能

### 2. `v0.9.69` —— 纯 Python（推荐做算法研究）

- commit `67dda7b`，2025-08-02
- 分型/笔/中枢全在 Python：
  - `czsc/analyze.py` — 429 行（`remove_include` / `check_fx` / `check_fxs` / `check_bi` / `CZSC` 类）
  - `czsc/objects.py` — 1176 行（`FX` / `BI` / `ZS` / `NewBar` 全部类型）
- 改完直接跑，不用编译
- 适合：读懂算法、改判定规则、做实验

切过去：
```bash
git checkout v0.9.69
# 想在上面改，开分支
git checkout -b py-dev v0.9.69
```

### 3. `dev` —— 你的开发分支

从 master 拉出，自动跟随 master 更新。在这里加自己的代码。

## 在 dev 上工作

```bash
git clone https://github.com/gansx/czsc.git
cd czsc
git checkout dev

# 改东西
git add -A && git commit -m "feat: ..."
git push origin dev
```

### dev 怎么跟上游保持同步

自动的——`sync-dev.yml` 在每次 `sync-upstream` 成功跑完后，会把 master merge 进 dev。

如果撞车了（你在 dev 改的地方上游也改了），会：
1. 自动回滚，dev 不变
2. 开一个 issue（label `dev-sync`），正文里有三种处理方案

### 不想让 dev 自动跟

删掉 `.github/workflows/sync-dev.yml` 就行。

或者在 GitHub 上禁用这个 workflow（Settings → Actions）。

## 如果想从纯 Python 版开始改

```bash
git checkout -b py-dev v0.9.69
git push -u origin py-dev
```

这个分支不在自动同步链里——上游不会动 `v0.9.69`，所以它天然稳定。想拿上游的新修复时手动 cherry-pick。

## 上游 workflow 已被移除

见 [.github/workflows/DISABLED_WORKFLOWS.md](.github/workflows/DISABLED_WORKFLOWS.md)。

## 两个需要注意的点

**1. 60 天无 commit 活动，GitHub 会暂停 scheduled workflow**

上游停更时就会触发。恢复：Actions 页面点 "Enable workflow"。

**2. 别在 master 上改代码**

master 每周被上游覆盖，你的改动会丢。永远在 `dev` 或自己开的分支上改。
