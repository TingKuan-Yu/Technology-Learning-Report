

## artifactCopyNames

- example in [boot-partner-build.yaml](https://dev.azure.com/ampx/device/_git/manifest.qcom?version=GBqcm8550/13/release/0400&path=/yaml/boot-partner-build.yaml)

  -  parameters of /templates/build/main-build-pipeline.yaml@build_core

    - buildJob: **Build_Release**
    
      ```yaml
          - template: /templates/build/main-build-pipeline.yaml@build_core
            parameters:
              packageFeed: '$(packageFeedComponent)'
              repoManifestXMLArtifact: '$(repoManifestXMLArtifact)'
              repoManifestArtifactFeed: '$(runtimeRepoManifestArtifactFeed)'
              repoManifestArtifactName: '$(runtimeRepoManifestArtifactName)'
              repoManifestArtifactVersion: '$(runtimeRepoManifestPackageVersion)'
              repoBuildBranch: $(Build.SourceBranch)
              buildPool: $(buildPool) 
              buildImage: $(buildImage) 
              buildJobs:
              - buildJob: Build_Release
                dockerMapping:
                - "/d/mirror;/d/mirror"
                buildCmds:
                - buildCmd:
                    cmd: 'bash ./tooling/codebase_util/src/vendor_feature/script/setup_environment.sh && cd ./tooling/codebase_util && bash ./run.sh --setup && ./run.sh --run src/vendor_feature/script/apply_vendor_feature.py --device ms8550 --on_docker  --android_version v1 --clone_path new_repo --output py_output --vendor_patchs_repo mdep/mdep.qcm8550.docs --vendor_patchs_repo_branch $(vendorDocsBranch) --vendor_patchs_repo_patch_root sample-patches --vendor_patchs_config src/vendor_feature/config_files/qcm8550_patches.json --apply_merge_feature'
                    azImageName: $(azImageName)
                    azImageVersion: $(azImageVersion)
                    name: 'Apply merge patches'
                - buildCmd:
                    cmd: 'bash ./tooling/codebase_util/src/vendor_feature/script/setup_environment.sh && cd ./tooling/codebase_util && bash ./run.sh --setup && bash ./run.sh --run src/vendor_feature/script/download_vendor_feature_files.py --on_docker --build_type boot --device ms8550 --android_version v1 --clone_path . --output output_log --release_json /s/mdep/mdep-vendor-release/vendor-release-ms8550.jsonc'
                    azImageName: $(azImageName)
                    azImageVersion: $(azImageVersion)
                    name: 'Download Vendor Feature repo and libraries on Vendor'
                - buildCmd:
                    cmd: 'bash ./tooling/codebase_util/src/vendor_feature/script/setup_environment.sh && cd ./tooling/codebase_util && bash ./run.sh --setup && ./run.sh --run src/vendor_feature/script/apply_vendor_feature.py --device ms8550 --on_docker  --android_version v1 --clone_path new_repo --output py_output --vendor_patchs_repo mdep/mdep.qcm8550.docs --vendor_patchs_repo_branch $(vendorDocsBranch) --vendor_patchs_repo_patch_root sample-patches --vendor_patchs_config src/vendor_feature/config_files/qcm8550_patches.json --apply_vendor_feature'
                    azImageName: $(azImageName)
                    azImageVersion: $(azImageVersion)
                    name: 'Apply vendor feature patches'
                - buildCmd:
                    cmd: 'make build_boot'
                    azImageName: $(azImageName)
                    azImageVersion: $(azImageVersion)
                artifactCopyName: 'boot-release'
                packageName: 'boot-release'
                sign: false
      ```

      -  buildCmd:
            cmd: 'bash ./**tooling/codebase_util**/src/vendor_feature/script/setup_environment.sh && cd ./tooling/codebase_util && bash ./run.sh --setup && ./run.sh --run src/vendor_feature/script/apply_vendor_feature.py --device ms8550 --on_docker  --android_version v1 --clone_path new_repo --output py_output --vendor_patchs_repo mdep/mdep.qcm8550.docs --vendor_patchs_repo_branch $(vendorDocsBranch) --vendor_patchs_repo_patch_root sample-patches --vendor_patchs_config src/vendor_feature/config_files/qcm8550_patches.json --apply_merge_feature'
        -  <project remote="codebase_util" groups="build" name="tooling/_git/codebase_util" path="tooling/codebase_util"/>
    
    - log
    
      ```
      2025-10-16T05:31:10.9136768Z Generating script.
      2025-10-16T05:31:10.9142693Z Script contents:
      2025-10-16T05:31:10.9143161Z /mnt/vss/_work/1/s/build_core/public/scripts/create-version-json
      2025-10-16T05:31:10.9146246Z ========================== Starting Command Output ===========================
      2025-10-16T05:31:10.9153292Z [command]/usr/bin/bash /mnt/vss/_work/_temp/1667d834-91c5-4838-bc32-d00322db7d07.sh
      2025-10-16T05:31:11.1472409Z {
      2025-10-16T05:31:11.1472601Z   "Repository name": "manifest.qcom",
      2025-10-16T05:31:11.1472783Z   "Branch name": "refs/heads/qcm8550/13/release/0400",
      2025-10-16T05:31:11.1472969Z   "Commit SHA": "0d6b66dbfd12e8171b15789922d97aa7c0f1fe7e",
      2025-10-16T05:31:11.1473140Z   "Commit Message": "Add configurations for 0400 partner build",
      2025-10-16T05:31:11.1473306Z   "Build Version": "2025.1016.1746299",
      2025-10-16T05:31:11.1473495Z   "Date": "2025-10-16-05:31",
      2025-10-16T05:31:11.1473672Z   "Build Name": "qcm8550-13-partner-boot-build",
      2025-10-16T05:31:11.1473834Z   "Package Name": "boot-release",
      2025-10-16T05:31:11.1474013Z   "Package feed": "qcm8550-13-release-0400-component"
      2025-10-16T05:31:11.1474145Z }
      2025-10-16T05:31:11.1486919Z '/mnt/vss/_work/1/s/boot-release-version.json' -> '/mnt/vss/_work/1/a/boot-release-version.json'
      :
      2025-10-16T05:32:48.8094003Z Publishing package: boot-release, version: 2025.1016.1746299 using feed id: b2c77d2b-e708-7ef7-9d9b-603c4f9ff857, project: null
      2025-10-16T05:32:48.8098525Z /mnt/vss/_work/_tool/artifacttool/0.2.475/x64/artifacttool universal publish --feed b2c77d2b-e708-7ef7-9d9b-603c4f9ff857 --service https://dev.azure.com/ampx/ --package-name boot-release --package-version 2025.1016.1746299 --path /mnt/vss/_work/1/a --patvar UNIVERSAL_PUBLISH_PAT --verbosity None --description $(publish)
      ```
    
      
