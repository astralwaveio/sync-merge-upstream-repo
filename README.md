# Sync and merge upstream repository

同步并合并上游存储库和当前存储库; 这是一个用于合并远程更改的 Github Action。

## 用例

- 在保持最新状态的同时保留 repo（而不是 clone 它）。
- 有一个与上游同步的分支，并将更改拉入开发分支。

## 用法

> 注意： 需要在设置项目中设置环境变量: `UPSTREAM_REPO_VAR` ，要同步的仓库地址，以 `.git` 结尾。

- 示例：

```YAML
name: Sync and merge upstream repository

env:
  # [Required]: URL of upstream
  UPSTREAM_URL: ${{ vars.UPSTREAM_REPO_VAR }}
  # [Required]: ${{ secrets.GITHUB_TOKEN }} After the account is generated, no separate settings are required
  # Over here, we use a PAT instead to authenticate workflow file changes.
  WORKFLOW_TOKEN: ${{ secrets.GITHUB_TOKEN }}
  # [Optional]: defaults to main
  UPSTREAM_BRANCH: "main"
  # [Optional]: defaults to main
  DOWNSTREAM_BRANCH: "main"
  # [Optional]: fetch arguments
  FETCH_ARGS: ""
  # [Optional]: merge arguments
  MERGE_ARGS: ""
  # [Optional]: push arguments
  PUSH_ARGS: ""
  # [Optional]: toggle to spawn time logs (keeps action active); "true" or "false"; defaults to false
  SPAWN_LOGS: "true"

# Runs daily at 03:15
on:
  schedule:
    - cron: '15 3 * * *'
  # Allows manual workflow run (must in default branch to work)
  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: GitHub Sync to Upstream Repository
        uses: maximohub/sync-merge-upstream-repo@v0.8
        with: 
          upstream_repo: ${{ env.UPSTREAM_URL }}
          upstream_branch: ${{ env.UPSTREAM_BRANCH }}
          downstream_branch: ${{ env.DOWNSTREAM_BRANCH }}
          token: ${{ env.WORKFLOW_TOKEN }}
          fetch_args: ${{ env.FETCH_ARGS }}
          merge_args: ${{ env.MERGE_ARGS }}
          push_args: ${{ env.PUSH_ARGS }}
          spawn_logs: ${{ env.SPAWN_LOGS }}
```

此操作每天在 03:15 UTC 同步您在分支的仓库（合并来自上游的更改）。 请注意：GitHub Action 计划的工作流在被推送到队列时通常会面临延迟，延迟通常在 1 小时以内。

注意：如果 `SPAWN_LOGS` 设置为 `true` ，此操作将在根目录中创建一个 `sync-merge-upstream-repo` 文件，其中包含操作运行时的时间戳。这是为了减轻 `GitHub` 在检测到不活动时禁用回购操作的麻烦。

## 发展

在 [`action.yml`](https://github.com/maximohub/sync-merge-upstream-repo/blob/master/action.yml), 我们定义  `inputs`.  
然后我们将这些参数传递给 [`Dockerfile`](https://github.com/maximohub/sync-merge-upstream-repo/blob/master/Dockerfile), 然后传递给 [`entrypoint.sh`](https://github.com/maximohub/sync-merge-upstream-repo/blob/master/entrypoint.sh).

`entrypoint.sh` 做的工作如下：

- 设置变量。
- 设置 git 配置。
- 克隆下游存储库。
- 获取上游存储库。
