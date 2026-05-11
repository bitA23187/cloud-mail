# Upstream History Rewrite — Runbook

When `🔄 Sync upstream into deploy` fails with `fatal: refusing to merge unrelated histories`, the upstream (`maillab/cloud-mail`) has force-pushed / rewritten its `main`. This runbook captures the manual reconcile flow used on 2026-05-11.

## 1. 现象 (Symptom)

GitHub 发邮件: `[<fork>/cloud-mail] Run failed: 🔄 Sync upstream into deploy - main (<sha>)`.

Failing step `🔀 合并上游 / Merge upstream` 报:

```
fatal: refusing to merge unrelated histories
fatal: There is no merge to abort (MERGE_HEAD missing).
```

## 2. 定位根因 (Diagnose)

```bash
# 注意 default repo 可能解析成 upstream,显式指定 fork
gh run list --repo <user>/cloud-mail --workflow=sync-upstream.yml --limit 10
gh run view <run-id> --repo <user>/cloud-mail --log-failed

# 验证 upstream 是否 force-push 了 (注意输出里的 'forced update' 标记)
git fetch upstream main
# 期望看到: + <old>...<new>  main  -> upstream/main  (forced update)

# 验证 deploy 和 upstream 已无共同祖先
git fetch origin deploy
git merge-base origin/deploy upstream/main; echo "exit=$?"
# exit=1 表示真的无共同祖先
```

## 3. 手动 Reconcile (一次性)

在 worktree 里做,避免污染主仓库工作区:

```bash
git worktree add /tmp/cloud-mail-deploy-merge origin/deploy
cd /tmp/cloud-mail-deploy-merge
git checkout -B deploy-test
git config user.name "<you>"
git config user.email "<you>@local"

git merge --no-commit --no-ff --allow-unrelated-histories upstream/main
# 预期会出现一堆 AA (add/add) 冲突
```

冲突解决策略:

| 文件类型 | 策略 | 说明 |
|---------|------|------|
| 应用代码 (`mail-vue/**`, `mail-worker/**`) | `--theirs` | 上游 rewrite 后内容是我们要的新版本 |
| `.github/workflows/deploy-cloudflare.yml` | `--theirs` + 手动改 `branches: [ deploy ]` | 保留上游新功能(PROJECT_LINK / pnpm v9 等),只回填 fork 的 branch 配置 |
| `doc/github-action.md` | `--ours` | fork 已写好 deploy 工作流文档,上游版本仍是 eoao 时代的旧文档 |
| `.github/workflows/sync-upstream.yml` / `AGENTS.md` | (fork-only) | 不会出现冲突,自动保留 |

批量解冲突:

```bash
# 1) 全部 take theirs
git diff --name-only --diff-filter=U | xargs -I{} git checkout --theirs {}
git diff --name-only --diff-filter=U | xargs git add

# 2) 回滚 doc/github-action.md 到 ours
git checkout HEAD -- doc/github-action.md
git add doc/github-action.md

# 3) 手改 deploy-cloudflare.yml: branches: [ main ] -> branches: [ deploy ]
#    然后 git add

# 4) sanity check
grep -rln '<<<<<<<\|>>>>>>>\|=======' .  # 应为空
git diff --stat upstream/main HEAD       # 应该只有 4 个 fork-owned 文件不同
```

提交并强推:

```bash
git commit --no-edit
git push --force-with-lease origin deploy-test:deploy
```

清理:

```bash
cd - && git worktree remove /tmp/cloud-mail-deploy-merge --force
git branch -D deploy-test
```

## 4. 已加入的防御

`.github/workflows/sync-upstream.yml` 的 merge 步骤现在会:

1. 用 `git merge-base "upstream/$UPSTREAM_BRANCH" HEAD` 检测无共同祖先。
2. 若检测到则改用 `git merge --allow-unrelated-histories -X theirs ...`,并在 step summary 里高亮 `⚠️ Upstream history rewrite detected`,提示人工 review。
3. 合并完成后从 `BEFORE_SHA` 恢复 fork-owned 路径:`.github/workflows`、`AGENTS.md`、`doc/github-action.md`。

下次再遇上 force-push,workflow 应能自动跑通;但仍建议看 step summary 里的告警并人工 diff `git diff upstream/main origin/deploy` 确认 fork-owned 文件以外没有意外。

## 5. 历史参考

- 2026-05-11: 上游从 `6ce918e` force-push 到 `c96e349`(同时 squash + 追加 blacklist / i18n / permission-fix 等若干新提交)。Reconcile 后 `deploy` HEAD = `41f73d9`。
