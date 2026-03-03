## build.core - main - ado-cloud-setup.yaml



## Download build.core

- git clone

  ```
  git clone https://ampx@dev.azure.com/ampx/tooling/_git/build.core -b main main
  ```

  

## private/templates/ado-cloud-setup/ado-cloud-setup.yaml

- Checkout

  ```yaml
  steps:
  - checkout: self
    clean: true
    displayName: 'Checkout manifest (ado-cloud-setup.yaml)'
  - checkout: build_core
    path: s/build_core
    displayName: 'Checkout build_core for pipeline dependencies (ado-cloud-setup.yaml)'  
  ```
  
  - | `checkout` | ADO 内置任务（Task Type: `Checkout`），无需额外引用，核心功能是：1. 连接流水线关联的 Git 仓库（Azure Repos Git / GitHub / Bitbucket）；2. 将代码拉取到 Agent 的工作目录（默认路径：`$(Agent.BuildDirectory)/s`）；3. 切换到流水线触发的提交 / 分支 / 标签版本。 |
    | ---------- | ------------------------------------------------------------ |
    | `self`     | ADO 专属参数值，特指 “**当前流水线所归属的代码仓库**”（即流水线定义所在的仓库）：- 若流水线关联多个仓库，可替换为仓库名称 / ID（如 `checkout: my-other-repo`）；- 省略时默认等价于 `self`（ADO 流水线默认会自动添加 `checkout: self` 步骤，除非显式禁用）。 |
  
  - log
  
    ```
    2025-12-02T09:14:34.7079069Z ##[section]Starting: Checkout manifest (ado-cloud-setup.yaml)
    2025-12-02T09:14:34.7083233Z ==============================================================================
    2025-12-02T09:14:34.7083383Z Task         : Get sources
    2025-12-02T09:14:34.7083460Z Description  : Get sources from a repository. Supports Git, TfsVC, and SVN repositories.
    2025-12-02T09:14:34.7083588Z Version      : 1.0.0
    2025-12-02T09:14:34.7083661Z Author       : Microsoft
    2025-12-02T09:14:34.7083729Z Help         : [More Information](https://go.microsoft.com/fwlink/?LinkId=798199)
    2025-12-02T09:14:34.7083816Z ==============================================================================
    2025-12-02T09:14:34.9875211Z Syncing repository: codebase_util (Git)
    2025-12-02T09:14:34.9914455Z ##[command]git version
    ```
  
    - 這個template 被codebase_util使用,　所以self是codebase_util
  
  - log
  
    ```
    2025-12-02T09:14:38.3277496Z ##[section]Starting: Checkout build_core for pipeline dependencies (ado-cloud-setup.yaml)
    2025-12-02T09:14:38.3281177Z ==============================================================================
    2025-12-02T09:14:38.3281288Z Task         : Get sources
    2025-12-02T09:14:38.3281352Z Description  : Get sources from a repository. Supports Git, TfsVC, and SVN repositories.
    2025-12-02T09:14:38.3281451Z Version      : 1.0.0
    2025-12-02T09:14:38.3281512Z Author       : Microsoft
    2025-12-02T09:14:38.3281578Z Help         : [More Information](https://go.microsoft.com/fwlink/?LinkId=798199)
    2025-12-02T09:14:38.3281663Z ==============================================================================
    2025-12-02T09:14:38.5986606Z Repository is current at '/mnt/vss/_work/1/s/build.core', move to '/mnt/vss/_work/1/s/build_core'.
    2025-12-02T09:14:38.5987065Z Repository will be located at '/mnt/vss/_work/1/s/build_core'.
    2025-12-02T09:14:38.5987408Z Syncing repository: build.core (git)
    ```
  
    