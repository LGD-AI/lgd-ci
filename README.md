# LGD-AI / lgd-ci

单一真源的 CI 工作流仓库。生态仓库不应再各自复制粘贴工作流，而是引用本仓的 reusable workflow。

## 为什么必须 pin 到 SHA

跨仓引用是信任边界：调用方若写可变的 main 分支，本仓任意一次恶意或误合的提交会立刻改变所有调用方的行为。
因此调用方必须写完整 40 位 commit SHA，禁止指向分支或短 tag。

## 用法

    jobs:
      ci:
        uses: LGD-AI/lgd-ci/.github/workflows/lgd-reusable-ci.yml@
        with:
          python-version: "3.11"
          run-tests: false
          require-skill-md: true

调用方自己声明触发条件：pull_request 用于 PR 门禁，push 到 main 用于合并后守门。

## 合并后守门是必须的

PR 门禁采用松散检查（不强制分支最新），代价是合并后可能撞坏基分支。补偿手段就是在 main 上跑全量 CI，
失败立即 revert，而不是让基分支长期带病。

## 约定

- 工作流只授予 contents: read，不接触 secrets
- 第三方 action 用官方 tag 或完整 SHA；本仓内升级视为一次可审查提交
- 修改本仓后，调用方需同步更新 pin 的 SHA（这是设计意图，不是缺陷）