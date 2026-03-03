# [(2-0) create-pr-branch](https://dev.azure.com/ampx/tooling/_build?definitionId=1648) Overview



## Overview

- 會依由 [(1-0) ingest-framework-security-patch](https://dev.azure.com/ampx/tooling/_build?definitionId=1647) 產生的security-patch/mdep-framework-patch內的資料來建立test branch. 其中可選
  - security-patch/mdep-framework-patch的版本.
  - test branch的prefix名字
  - 是否要移除之前的test branch
  - 是否要push 至ADO

## input - Parameters

- scripts/security_patch/yaml/create_pr_branch.yaml

  ```yaml
  parameters:
  - name: deviceName
    displayName: "Device Name"
    type: string
    default: "mdep"
    values:
      - "mdep"
  
  - name: securityPatchType
    displayName: "Security Patch Type"
    type: string
    default: "framework"
    values:
      - "framework"
      - "kernel"
  
  - name: ingestedSecurityPatchVersion
    displayName: "Ingested Security Patch Artifact Version"
    default: ""
    type: string
  
  - name: prBranchPrefix
    displayName: "PR Branch Prefix"
    default: ""
    type: string
  
  - name: deleteExistingBranch
    displayName: "Delete Existing Branch"
    type: boolean
    default: false
  
  - name: gitPush
    displayName: "Push PR Branch to Remote"
    type: boolean
    default: true
  ```

  - name: securityPatchType

    - 目前只打mdep, 因此選framework

  - name: ingestedSecurityPatchVersion

    - scripts/security_patch/yaml/create_pr_branch.yaml

      用來下載security patch. Downloading package: **mdep-framework-patch**, version: **2025.812.2** using feed id: security-patch

      ```yaml
                - task: UniversalPackages@0
                  displayName: "Download Ingested Security Patch"
                  retryCountOnTaskFailure: 5
                  inputs:
                    command: 'download'
                    downloadDirectory: $(repoRoot)/$(artifactOutputPath)
                    feedsToUse: 'internal'
                    vstsFeed: $(securityPatchFeedName)
                    vstsFeedPackage: $(ingestedSecurityPatchPackage)
                    vstsPackageVersion: ${{ parameters.ingestedSecurityPatchVersion }}
                    verbosity: 'debug'
      ```

    - log

      2025-08-12T13:06:33.3349396Z Downloading package: **mdep-framework-patch**, version: **2025.812.2** using feed id: security-patch, project: null.

      其中mdep-framework-patch是由 [(1-0) ingest-framework-security-patch](https://dev.azure.com/ampx/tooling/_build?definitionId=1647) 產生的.

      ```
      2025-08-12T13:06:26.9004988Z ##[section]Starting: Download Ingested Security Patch
      2025-08-12T13:06:26.9009417Z ==============================================================================
      2025-08-12T13:06:26.9009535Z Task         : Universal packages
      2025-08-12T13:06:26.9009589Z Description  : Download or publish Universal Packages
      2025-08-12T13:06:26.9009649Z Version      : 0.260.0
      2025-08-12T13:06:26.9009700Z Author       : Microsoft Corporation
      2025-08-12T13:06:26.9009751Z Help         : https://docs.microsoft.com/azure/devops/pipelines/tasks/package/universal-packages
      2025-08-12T13:06:26.9009829Z ==============================================================================
      2025-08-12T13:06:29.4496925Z Downloading: https://0psvsblobprodwus2145.vsblob.vsassets.io/artifacttool/artifacttool-linux-x64-Release_0.2.405.zip?sv=2019-07-07&sr=b&sig=-REDACTED-&skoid=8f4a31b6-ccf8-4d83-8ff6-444a0146cdb6&sktid=33e01921-4d64-4f8c-a055-5bdaffd5e33d&skt=2025-08-12T08%3A32%3A40Z&ske=2025-08-14T09%3A32%3A40Z&sks=b&skv=2019-07-07&se=2025-08-12T14%3A06%3A30Z&sp=r&P1=1755007290&P2=11&P3=2&P4=3O3y4%2bWmYlnbTUrD7S5GN%2fgmQLfiiF9AXOehgGJ3UeU%3d
      2025-08-12T13:06:32.1523982Z Caching tool: artifacttool 0.2.405 x64
      2025-08-12T13:06:33.3349396Z Downloading package: mdep-framework-patch, version: 2025.812.2 using feed id: security-patch, project: null
      2025-08-12T13:06:33.3349794Z /mnt/vss/_work/_tool/artifacttool/0.2.405/x64/artifacttool universal download --feed security-patch --servic
      ```

      
