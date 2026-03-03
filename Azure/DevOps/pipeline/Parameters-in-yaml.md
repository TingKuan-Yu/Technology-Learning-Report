[TOC]



# Azure pipeline裏parameters的功能?

在 **Azure DevOps Pipeline** 中，`parameters` 的主要功能是讓 **YAML 管線**在執行前具備高度的**可配置性與重用性**，透過參數化設計，你可以在不修改 YAML 檔案的情況下，動態控制管線的行為。以下是重點說明：

------

### ✅ **Parameters 的核心功能**

1. **定義可變值**

   - 在 YAML 檔案頂層使用 `parameters:` 區塊定義。
   - 每個參數必須包含 `name` 和 `type`，可選擇 `default` 預設值與 `values` 限定值範圍。
   - 支援的資料型態：
     - `string`：字串
     - `number`：數字
     - `boolean`：布林值
     - `object`：複雜 YAML 結構
     - `step` / `stepList`：單步驟或多步驟
     - `job` / `jobList`：單作業或多作業
     - `stage` / `stageList`：單階段或多階段 [[learn.microsoft.com\]](https://learn.microsoft.com/en-us/azure/devops/pipelines/yaml-schema/parameters-parameter?view=azure-pipelines)

2. **執行時選擇值**

   - 在 **Run Pipeline UI** 中，使用者可以手動選擇參數值，或保留預設值。
   - 適合用於：
     - 選擇不同的 **Agent Pool** 或 **VM Image**。
     - 控制是否執行測試、部署哪個環境等。 [[learn.microsoft.com\]](https://learn.microsoft.com/zh-cn/azure/devops/pipelines/process/runtime-parameters?view=azure-devops), [[ithelp.ithome.com.tw\]](https://ithelp.ithome.com.tw/articles/10301889)

3. **模板與重用**

   - 可將參數傳遞給 **template**，實現跨管線的重用。

   - 例如：

     parameters:

      \- name: environment

       type: string

       default: production

     jobs:

      \- job: Deploy

       steps:

        \- script: echo "Deploying to ${{ parameters.environment }}"

     \```[4](https://bing.com/search?q=Azure+pipeline+YAML+parameters+%e7%94%a8%e6%b3%95)

4. **條件控制與動態行為**

   - 搭配 

     ```
     if
     ```

      條件式與模板表達式，根據參數值決定執行哪些步驟或作業：

     \- ${{ if eq(parameters.runTests, true) }}:

      \- script: echo "Running tests"

     \```[5](https://blog.csdn.net/2501_90323865/article/details/149762109)

------

### ✅ **Parameters 與 Variables 的差異**

| 特性         | Parameters               | Variables                                                    |
| ------------ | ------------------------ | ------------------------------------------------------------ |
| **評估時間** | 模板剖析階段（Queue 時） | 執行階段（Runtime）                                          |
| **可變性**   | Queue 後不可變           | 可在執行過程中動態更新                                       |
| **用途**     | 控制 YAML 結構與流程     | 控制任務腳本與條件 [[learn.microsoft.com\]](https://learn.microsoft.com/zh-cn/azure/devops/pipelines/process/runtime-parameters?view=azure-devops) |

------

### ✅ **最佳實踐**

- **必須提供預設值**，否則執行時必須手動輸入。
- 使用 `values` 限定範圍，避免錯誤輸入。
- 適合搭配 **模板化設計**，提升管線重用性。
- 不支援秘密值（Secret），若需敏感資訊請用 `variables`。 [[learn.microsoft.com\]](https://learn.microsoft.com/zh-cn/azure/devops/pipelines/process/runtime-parameters?view=azure-devops)

------

要不要我幫你 **整理一個完整的 YAML 範例**，展示如何定義多個 `parameters`（含不同型態）、條件控制，以及模板傳遞？還是你想看 **Parameters vs Variables 的圖解比較**？





#　parameters和variables在yaml pipeline的差別

在 **Azure DevOps YAML Pipeline** 中，`parameters` 和 `variables` 都能讓管線更具彈性，但它們的用途與行為有明顯差異。以下是核心比較：

------

### ✅ **1. 評估時間 (Evaluation Time)**

- **Parameters**
  - 在 **模板剖析階段 (Queue 時)** 評估，屬於**編譯時期**概念。
  - 使用 `${{ parameters.xxx }}` 語法，值在管線開始前就已展開。
  - 適合控制 YAML 結構，例如決定是否包含某個 stage 或 job。 [[learn.microsoft.com\]](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/runtime-parameters?view=azure-devops)
- **Variables**
  - 在 **執行階段 (Runtime)** 評估，值可在管線執行過程中改變。
  - 使用 `$(variableName)` 或 `$[variables.var]` 語法，適合在 script 或 task 中使用。 [[learn.microsoft.com\]](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/variables?view=azure-devops)

------

### ✅ **2. 可變性 (Mutability)**

- **Parameters**
  - **不可變**：Queue 後無法修改，執行過程中不能被任務更新。
  - 適合定義固定選項，例如 agent 類型、部署環境。 [[learn.microsoft.com\]](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/runtime-parameters?view=azure-devops)
- **Variables**
  - **可變**：可透過 `##vso[task.setvariable]` 在執行過程中更新，並傳遞給後續步驟或作業。
  - 適合儲存動態值，例如 build 編號、路徑、運算結果。 [[compoundent.com\]](https://compoundent.com/the-glass/azure-pipelines-parameters-vs-variables)

------

### ✅ **3. 資料型態與安全性**

- **Parameters**
  - 支援多種型態：`string`、`number`、`boolean`、`object`、`step`、`job`、`stage` 等。
  - 可限制值範圍 (`values`)，確保輸入合法。
  - **不支援 Secret**，敏感資訊不能用 parameters。 [[ithelp.ithome.com.tw\]](https://ithelp.ithome.com.tw/articles/10301889)
- **Variables**
  - 所有值皆為 **字串**，但可標記為 **Secret** 並加密。
  - 適合儲存 API Key、密碼等敏感資訊。 [[learn.microsoft.com\]](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/variables?view=azure-devops)

------

### ✅ **4. 使用場景**

- Parameters

  - 控制管線結構：決定是否執行某個 stage/job、選擇 agent pool。

  - 適合模板化設計，讓 YAML 可重用。

  - 例如：

    parameters:

     \- name: vmImageOS

      type: string

      default: ubuntu-latest

      values:

       \- ubuntu-latest

       \- windows-latest

    pool:

     vmImage: ${{ parameters.vmImageOS }}

    \```[4](https://ithelp.ithome.com.tw/articles/10301889)

- Variables

  - 控制任務行為：傳遞路徑、版本號、條件判斷。

  - 可在 pipeline UI 或 variable group 設定，並在 YAML 中使用：

    variables:

     configuration: debug

    steps:

     \- script: echo "Config is $(configuration)"

    \```[2](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/variables?view=azure-devops)

------

### ✅ **總結對照表**

| 特性        | Parameters                 | Variables            |
| ----------- | -------------------------- | -------------------- |
| 評估時間    | Queue 時 (模板剖析)        | Runtime (執行階段)   |
| 可變性      | 不可變                     | 可變，支援動態更新   |
| 型態        | 多種型態 (string, number…) | 僅字串               |
| Secret 支援 | 不支援                     | 支援                 |
| 典型用途    | 控制 YAML 結構、模板重用   | 傳遞動態值、任務條件 |

------

要不要我幫你 **畫一張圖表 (Parameters vs Variables)**，或提供 **完整 YAML 範例**，示範兩者如何搭配條件判斷與模板？