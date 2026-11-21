# LGD-AI / lgd-ci

单一真源的 CI 工作流仓库。生态仓库**不应**再各自复制粘贴工作流，而是引用本仓的 reusable workflow。

## 用法

```yaml
name: CI
on: [pull_request, push]
jobs:
  ci:
    uses: LGD-AI/lgd-ci/.github/workflows/lgd-reusable-ci.yml@main
    with:
      python-version: "3.11"
      run-tests: true
      require-skill-md: false
```

## 为什么

- 一处修改，全生态生效，不再有 20 份彼此漂移的模板
- 升级是一次可审查的提交，而不是 20 次复制粘贴
- 可被 org/仓库 ruleset 作为「合并前必须通过的 workflow」直接引用

## 约定

- 第三方 action 用完整 40 位 SHA 或官方 tag，升级前先在本仓跑通
- 工作流只授予 `contents: read`，不接触 secrets