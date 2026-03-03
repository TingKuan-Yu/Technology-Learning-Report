# [(1-0) ingest-framework-security-patch](https://dev.azure.com/ampx/tooling/_build?definitionId=1647) Overview



## Overview

- 會由Manifest Branch(如mdep/13/1303.1/main)產生merge 的manifest xml file. 下載由ado上的security patch, 分析是否有無法apply的repo, 最後產生mdep-framework-patch artifact.

## input - Parameters

![image-20251217173156715](/home/tony/.config/Typora/typora-user-images/image-20251217173156715.png)

- [(1-0) ingest-framework-security-patch](https://dev.azure.com/ampx/415ef6b4-62b1-4640-87ac-20f7c497e53f/_build?definitionId=1647&_a=summary)/[2025.1201.1](https://dev.azure.com/ampx/415ef6b4-62b1-4640-87ac-20f7c497e53f/_build/results?buildId=1956295)
  
  - security_patch/scripts/security_patch/yaml/ingest_framework_security_patch.yaml
  
    ```yaml
    parameters:
    - name: deviceName
      displayName: "Device Name"
      type: string
      default: "mdep"
      values:
        - "mdep"
    
    - name: androidVersion
      displayName: "Android Version"
      type: string
      default: "Auto-Detect"
      values:
        - "Auto-Detect"
        - "11"
        - "12L"
        - "13"
        - "15"
    
    - name: manifestUrl
      displayName: "HTTP URL for Manifest (e.g. https://dev.azure.com/ampx/mdep/_git/manifest.mdep)"
      type: string
      default: ""
    
    - name: manifestBrnach
      displayName: "Manifest Branch (e.g. ssi/12/1202/main)"
      type: string
      default: ""
    
    - name: manifestName
      displayName: "Manifest Name (optional)"
      default: " "
      type: string
    
    - name: securityPatchVersion
      displayName: "Security Patch Artifact Version"
      default: ""
      type: string
    ```
  
    - Device Name
      - mdep
  
    - Android Version
      - 13
  
    - HTTP URL for Manifest (e.g. https://dev.azure.com/ampx/mdep/_git/manifest.mdep)
      - https://dev.azure.com/ampx/mdep/_git/manifest.mdep
  
    - Manifest Branch (e.g. ssi/12/1202/main)
      - mdep/13/1303.1/main
  
    - Manifest Name (optional)
      - empty
  
    - Security Patch Artifact Version
      - 2025.901.1
        - 9月的.
  
    - Target ADO Projects
      - mdep,device,ssi
        - 應用mdep即可. 因我們只作mdep
  
    - Update the SPL Version
      - 是否要產生spl version patch

## output

- mdep-framework-patch - [2025.1201.1](https://dev.azure.com/ampx/mdep/_artifacts/feed/security-patch/UPack/mdep-framework-patch/overview/2025.1201.1)

## Resource

- [Introduce all security patch automation pipelines - Overview](https://dev.azure.com/ampx/eng/_wiki/wikis/wiki/8555/Introduce-all-security-patch-automation-pipelines)
- [Pipelines - Pipelines](https://dev.azure.com/ampx/tooling/_build?definitionScope=\codebase_util\security-patch)
- artifact
  - [security_patch](https://dev.azure.com/ampx/mdep/_artifacts/feed/security-patch)
    - framework-patch

- repo
  - codebase_util
    - branch 
      - [security_patch](https://dev.azure.com/ampx/tooling/_git/codebase_util?version=GBsecurity_patch)




## Parameters

- code

  - scripts/security_patch/yaml/ingest_framework_security_patch.yaml

- deviceName

  ```
  parameters:
  - name: deviceName
    displayName: "Device Name"
    type: string
    default: "mdep"
    values:
      - "mdep"
  ```

  -  usage

    

  

  

- targetAdoProject

  在ADO內remote是為**mdep**還是其它 - https://dev.azure.com/ampx/**mdep**/_git

  ```
  - name: targetAdoProject
    displayName: "Target ADO Projects"
    default: "mdep,device,ssi"
    type: string
  ```

  - usage

    -   ingest_framework_security_patch.py

      ```yaml
                - script: |
                    bash $(repoRoot)/$(securityPatchPipelineRootDir)/c "./ingest_framework_security_patch.py" \
                        --os_ver "$(androidVer)" \
                        --security_patch_path "$(System.DefaultWorkingDirectory)" \
                        --target_ado_projects "${{ parameters.targetAdoProject }}" \
                        --output "$(repoRoot)/$(artifactOutputPath)" \
                        --ignore_error "${{ lower(parameters.ignoreError) }}"
      ```

      

- updateSplVer

  ```yaml
  - name: updateSplVer
    displayName: "Update the SPL Version"
    type: boolean
    default: true
  ```

  - usage

    - generate_spl_version_patch.py

      ```
                - ${{ if eq(parameters.updateSplVer, true) }}:
                  - script: |
                      bash $(repoRoot)/$(securityPatchPipelineRootDir)/security_patch_script_launcher.sh "generate_spl_version_patch.py" \
                          --security_patch_path "$(repoRoot)/$(artifactOutputPath)" \
                          --aosp_build_repo_path "$(repoRoot)/build"
                    displayName: "Generate Patch to Update the SPL Version"
      ```

      

- ignoreError

  ```yaml
  - name: ignoreError
    displayName: "Ignore Error"
    type: boolean
    default: false
  ```

  
