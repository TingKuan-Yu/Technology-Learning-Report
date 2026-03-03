# [(4-0) generate-security-patch-report](https://dev.azure.com/ampx/tooling/_build?definitionId=1650&_a=summary)



## Note

- Pipeline的script寫成一個功能, 獨立一個script, 減少依賴.



## Resources for pipeline

- Feed
  - out
    - [security-patch](https://dev.azure.com/ampx/tooling/_artifacts/feed/security-patch) 
      - [mdep-framework-report](https://dev.azure.com/ampx/tooling/_artifacts/feed/security-patch/UPack/mdep-framework-report/overview)
- pipeline
  - [security-patch](https://dev.azure.com/ampx/tooling/_build?definitionScope=%5Ccodebase_util%5Csecurity-patch)
- repo
  - codebase_util - [security_patch](https://dev.azure.com/ampx/tooling/_git/codebase_util?version=GBsecurity_patch)
    - security_patch/yaml/generate_security_patch_report.yaml



## Format of pull reques

- aosp-platform.packages.services.Telephony

  - [Pull Request 21997](https://dev.azure.com/ampx/mdep/_git/aosp-platform.packages.services.Telephony/pullrequest/21997): Apply security patch 2025-09

    - branch - spl/2509_new/mdep/13/1303.1/main/packages/services/Telephony

    