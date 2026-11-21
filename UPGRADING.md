# 升级 SOP：`LGD-AI/lgd-ci` 的版本化与调用方升级

本仓是全生态的 CI 单一真源。它一旦变化，所有调用方行为跟着变，因此**不允许"静默升级"**。
下面这套流程是把 GitHub 原生机制（reusable workflow + tag + release）用成"可控升级通道"。

## 1. 三条硬规则

1. **调用方一律写完整 40 位 commit SHA**，不写分支、不写短 tag。跨仓引用是信任边界。
2. **本仓内不得移动任何已发布的 tag**。`v1.0.0` 指向哪个 commit，永远是那个 commit。
3. **tag 只是"人可读的坐标"，不是调用方引用的依据**。调用方升级 = 改自己那一行 SHA。

## 2. 发布流程（维护者）

1. 改 `lgd-reusable-ci.yml` / `selftest.yml`，在 PR 里写清"改了什么、影响谁、默认值有没有变"。
2. 默认值**只增不减**：新增 input 一律 `default: false`，避免"调用方什么都没做却行为变了"。
3. 合并后在本仓跑一次 `selftest.yml`（自测工作流），确认唯一真源自身可用。
4. 打 tag 并发布 release：

```
gh release create vX.Y.Z --repo LGD-AI/lgd-ci \
  --target <merge后main的SHA> \
  --title "vX.Y.Z" \
  --notes "- 变更点\n- 影响面\n- 是否改变默认行为：否"
```

5. 版本语义：
   - `X`：破坏性变更（input 改名、必需行为变化、权限收紧）
   - `Y`：新增能力（新增 opt-in input、新增步骤）
   - `Z`：修复（不改行为、不改默认值）

## 3. 升级流程（调用方）

1. 读 release notes，确认"是否改变默认行为"。
2. 本地先开一个分支，把 `uses: LGD-AI/lgd-ci/.github/workflows/lgd-reusable-ci.yml@<旧SHA>` 改成新 SHA。
3. 开 PR，看 CI 结论；绿了再合。
4. 合完在本仓 `main` 上再跑一遍（合并后守门），失败就立刻 revert。

## 4. 回滚

```bash
gh release delete vX.Y.Z --repo LGD-AI/lgd-ci --yes   # 只删 release，不删 tag
git push --delete origin refs/tags/vX.Y.Z             # 删 tag（需明确理由，记入 release notes）
```

调用方回滚 = 把 SHA 改回上一个值，一次 commit 解决，不需要动工作流逻辑。

## 5. 已知的"静默瘫痪"坑（写在这里，避免重复踩）

- `actions.lock` 记的是**精确版本**（`actions/checkout@v7.0.1`）。工作流若写浮动 major
  （`actions/checkout@v7`），GitHub 判定该 workflow **无法启动**（`startup_failure`），
  而 `git push` 不报错、页面上没有日志。
- lockfile 里的 `workflows:` 块必须逐条登记每个工作流；把条目追加到文件末尾会落到
  `dependencies:` 之后，平台报 `cannot unmarshal !!seq into lockfile.Action`，整仓工作流全灭。
- 因此本仓新增 `verify-lockfile-refs`（**默认 false**）。哪个仓想开，自己加：

```yaml
with:
  verify-lockfile: true
  verify-lockfile-refs: true   # 打开引用一致性预检
```

开了之后，任何"浮动 ref 与 lockfile 精确版本不一致"都会在 CI 里点名报错，而不是静默瘫痪。
