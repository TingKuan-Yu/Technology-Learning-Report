# OsVendorDetector.py



## Overview

- Purpose: **Detect** the Android OS **version** for the pipeline, preferring cached/override data, otherwise deriving from the codebase, then export it as a pipeline variable. See [OsVerDetector.py](vscode-file://vscode-app/usr/share/code/resources/app/out/vs/code/electron-browser/workbench/workbench.html).
- Inputs: [--working_path](vscode-file://vscode-app/usr/share/code/resources/app/out/vs/code/electron-browser/workbench/workbench.html), [--aosp_build_repo_path](vscode-file://vscode-app/usr/share/code/resources/app/out/vs/code/electron-browser/workbench/workbench.html), [--output_variable_name](vscode-file://vscode-app/usr/share/code/resources/app/out/vs/code/electron-browser/workbench/workbench.html), [--override_os_ver](vscode-file://vscode-app/usr/share/code/resources/app/out/vs/code/electron-browser/workbench/workbench.html) (parsed in [OsVerDetector.py:27-35](vscode-file://vscode-app/usr/share/code/resources/app/out/vs/code/electron-browser/workbench/workbench.html)).
- Detection order:
  - Use existing [os_ver](vscode-file://vscode-app/usr/share/code/resources/app/out/vs/code/electron-browser/workbench/workbench.html) from [META_INFO_JSON](vscode-file://vscode-app/usr/share/code/resources/app/out/vs/code/electron-browser/workbench/workbench.html) if present ([load and check](vscode-file://vscode-app/usr/share/code/resources/app/out/vs/code/electron-browser/workbench/workbench.html)).
  - If not set and [--override_os_ver](vscode-file://vscode-app/usr/share/code/resources/app/out/vs/code/electron-browser/workbench/workbench.html) provided, use that ([override handling](vscode-file://vscode-app/usr/share/code/resources/app/out/vs/code/electron-browser/workbench/workbench.html)).
  - Else derive from codebase: read merged manifest, locate AOSP build repo, get default SDK version, map SDK→OS (`29→10`, `30→11`, `31→12`, `32→12L`, `33→13`) ([mapping and ](vscode-file://vscode-app/usr/share/code/resources/app/out/vs/code/electron-browser/workbench/workbench.html)[get_os_ver()](vscode-file://vscode-app/usr/share/code/resources/app/out/vs/code/electron-browser/workbench/workbench.html)).
- **Output**: Exports the OS version to Azure Pipelines as `##vso[task.setvariable variable=<name>;]<value>` and persists it back to [META_INFO_JSON](vscode-file://vscode-app/usr/share/code/resources/app/out/vs/code/electron-browser/workbench/workbench.html) ([variable export and dump](vscode-file://vscode-app/usr/share/code/resources/app/out/vs/code/electron-browser/workbench/workbench.html)).
- Errors: Throws [ValueError](vscode-file://vscode-app/usr/share/code/resources/app/out/vs/code/electron-browser/workbench/workbench.html) if required args are empty or if OS version cannot be determined ([OsVerDetector.py:25-26](vscode-file://vscode-app/usr/share/code/resources/app/out/vs/code/electron-browser/workbench/workbench.html), [OsVerDetector.py:62-65](vscode-file://vscode-app/usr/share/code/resources/app/out/vs/code/electron-browser/workbench/workbench.html)).



## USAGE

- scripts/security_patch/yaml/ingest_framework_security_patch.yaml

  ```yaml
            - script: |
                bash $(repoRoot)/$(securityPatchPipelineRootDir)/security_patch_script_launcher.sh "./util/OsVerDetector.py" \
                    --working_path "$(repoRoot)/$(artifactOutputPath)" \
                    --aosp_build_repo_path "$(repoRoot)/build" \
                    --output_variable_name 'androidVer' \
                    --override_os_ver '${{ replace(parameters.androidVersion, 'Auto-Detect', '') }}'
              displayName: "Detect the OS Version"
  ```



## Code Study - def output_ado_variable(var_name, var_value):

- code

  ```python
  def output_ado_variable(var_name, var_value):
      print(f'##vso[task.setvariable variable={var_name};]{var_value}')
      return
  ```

  - Purpose: [output_ado_variable()](vscode-file://vscode-app/usr/share/code/resources/app/out/vs/code/electron-browser/workbench/workbench.html) sets an Azure Pipelines variable at runtime **by emitting a special logging command to stdout.**
    - Also refer to --output_variable_name '**androidVer**' 
  - How: It prints `##vso[task.setvariable variable=<name>;]<value>`, which the Azure DevOps agent interprets to create/update the variable.
  - Inputs: [var_name](vscode-file://vscode-app/usr/share/code/resources/app/out/vs/code/electron-browser/workbench/workbench.html) (variable key) and [var_value](vscode-file://vscode-app/usr/share/code/resources/app/out/vs/code/electron-browser/workbench/workbench.html) (stringified value).
  - Scope: **Makes the variable available** to subsequent tasks in the **same job** (usable as `$(<name>)`). It doesn’t mark it secret or output-scoped.
  - Notes: If you need secrets or output variables, Azure supports flags like `issecret=true` or `isOutput=true` in the command, which this helper doesn’t include.
  - Example: [output_ado_variable('androidVer', '13')](vscode-file://vscode-app/usr/share/code/resources/app/out/vs/code/electron-browser/workbench/workbench.html) sets **`$(androidVer)`** to `13` for later steps.



## Log - Detect the OS Version

- log

  ```
  2025-12-02T09:16:02.7630739Z ##[section]Starting: Detect the OS Version
  2025-12-02T09:16:02.7639169Z ==============================================================================
  2025-12-02T09:16:02.7639381Z Task         : Command line
  2025-12-02T09:16:02.7639523Z Description  : Run a command line script using Bash on Linux and macOS and cmd.exe on Windows
  2025-12-02T09:16:02.7639716Z Version      : 2.250.1
  2025-12-02T09:16:02.7639853Z Author       : Microsoft Corporation
  2025-12-02T09:16:02.7639993Z Help         : https://docs.microsoft.com/azure/devops/pipelines/tasks/utility/command-line
  2025-12-02T09:16:02.7640294Z ==============================================================================
  2025-12-02T09:16:02.9072668Z Generating script.
  2025-12-02T09:16:02.9081369Z ========================== Starting Command Output ===========================
  2025-12-02T09:16:02.9094721Z [command]/usr/bin/bash --noprofile --norc /mnt/vss/_work/_temp/8130a973-bbf4-4b1d-8793-70f7032217d5.sh
  2025-12-02T09:16:03.4162233Z [2025-12-02 09:16:03][INFO] Start to detect the OS version
  2025-12-02T09:16:03.4165187Z [2025-12-02 09:16:03][INFO] Use the override OS version: 13
  2025-12-02T09:16:03.4165604Z [2025-12-02 09:16:03][INFO] The final OS version 13
  2025-12-02T09:16:03.4332925Z 
  2025-12-02T09:16:03.4400607Z ##[section]Finishing: Detect the OS Version
  ```

  
