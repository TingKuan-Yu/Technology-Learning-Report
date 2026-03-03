<table style="color:black; background-color:grey; font-size:bigger; margin:0px; padding:0px">
<tr>
<td><strong> Quality: </strong> draft </td>
<td><strong> Last Edit: </strong> 2024-09-06 </td>
<td><strong> Page Owner: </strong> rapope </td>
</tr></table>
# [Update-docker-images.md](https://dev.azure.com/ampx/dps/_git/wiki.old?path=%2FHow-Things-Work%2FDocker%2FUpdate-docker-images.md&_a=contents&version=GBwikiMain)



## Summary

This page describes the procedure (manual steps) of building and testing current docker images.

- Requirements
  Updating Docker images constantly is very important for maintaining the images up to date with the latest packages and be compliant with 1ES requirements.

- Currently we manually update the docker images once every 1-2 months but this should be automated in the future.

  

## Usage
There are 2 steps to take into consideration when updating a docker image:

1. Building the docker image.
2. Testing the docker image

----------------------------------------------------------------------------------------
1) **Building the docker image:**

Before building the docker images, please consult this [wiki](https://dev.azure.com/ampx/dps/_wiki/wikis/wiki/7822/Base-images-of-current-docker-images) to see the current docker images used and the base images of each one.

- Go to [docker images](https://dev.azure.com/ampx/dps/_git/docker.images)
- Create a personal branch from main branch.

   - Update base.u-2004 -> Run this [pipeline](https://dev.azure.com/ampx/dps/_build?definitionId=3573) and <span style="color: red;">**DON'T**</span> approve the LKG stage.
   - Create new branch from main (https://dev.azure.com/ampx/dps/_git/docker.images)
   - Update the Dockerfile of all images that use base.u-2004:lkg to base.u-2004:latest (hlos.eos-11, hlos.rockchip-13, nhlos.qcm6490, nhlos.qcm8550, nhlos.qcs8250)
   - Update images that are based on base.u-2004 (run the above 4 pipelines on your personal branch above) -> https://dev.azure.com/ampx/dps/_build?definitionScope=%5Cdocker

------------------------------------------------------------------------------
2. **Testing the docker image:**

- Go to each manifest
- Create new branch from main 
- Update the yaml files of the builds to use the latest docker tag instead of lkg (make sure you update only $(azImageVersion) from $(azImageName) and NOT from $(imageNameAzCli) because this is using azcli image to get packages)
- Run below manifest pipelines on the personal branch created above that will trigger the build pipelines
------------------------------------------------------------------------------
3. If all build pipelines are successful, approve the lkg stage from [base.u-2004](https://dev.azure.com/ampx/dps/_build?definitionId=3573)
   - Build base.u-2004 based images on main branch and approve the lkg stage:
     
     - [hlos.eos-11](https://dev.azure.com/ampx/dps/_build?definitionId=309)
     - [hlos.rockchip-13](https://dev.azure.com/ampx/dps/_build?definitionId=3765)
     - [nhlos.qcm6490](https://dev.azure.com/ampx/dps/_build?definitionId=4718)
     - [nhlos.qcm8550](https://dev.azure.com/ampx/dps/_build?definitionId=6351)
     - [nhlos.qcs8250](https://dev.azure.com/ampx/dps/_build?definitionId=326)

**base.u-2004 based images:**


1. ***hlos.eos-11*** 

- [mdep/manifest.mdep](https://dev.azure.com/ampx/mdep/_git/manifest.mdep)

    - baseline-manifest.yaml (https://dev.azure.com/ampx/mdep/_build?definitionId=6151)
    - partner-manifest.yaml (https://dev.azure.com/ampx/mdep/_build?definitionId=2700)
    - sign-manifest.yaml (https://dev.azure.com/ampx/mdep/_build?definitionId=2698)
    - system-manifest.yaml (https://dev.azure.com/ampx/mdep/_build?definitionId=3327&_a=summary)
    
- [mdep/manifest.ra](https://dev.azure.com/ampx/mdep/_git/manifest.ra) 

    - system-manifest.yaml (https://dev.azure.com/ampx/mdep/_build?definitionId=2702&_a=summary)

- [mdep/manifest.wcc](https://dev.azure.com/ampx/mdep/_git/manifest.wcc) 
    - system-manifest.yaml (https://dev.azure.com/ampx/mdep/_build?definitionId=2704)
    
- [manifest.ewt14](https://dev.azure.com/ampx/mdep/_git/manifest.ewt14) 
    - system-manifest.yaml (https://dev.azure.com/ampx/mdep/_build?definitionId=3216)
    
- [device/manifest.imx8m.merge](https://dev.azure.com/ampx/device/_git/manifest.imx8m.merge) 
    - merge-manifest.yaml (https://dev.azure.com/ampx/device/_build?definitionId=2680)
- [device/manifest.dart-mx8m.merge](https://dev.azure.com/ampx/device/_git/manifest.dart-mx8m.merge) 
    - merge-manifest.yaml (https://dev.azure.com/ampx/device/_build?definitionId=2692)
- [device/manifest.qcm8550.merge](https://dev.azure.com/ampx/device/_git/manifest.qcm8550.merge)
    - merge-manifest.yaml (https://dev.azure.com/ampx/device/_build?definitionId=6434)
- [device/manifest.mt8195.merge](https://dev.azure.com/ampx/device/_git/manifest.mt8195.merge) 
    - merge-manifest.yaml (https://dev.azure.com/ampx/device/_build?definitionId=3827)
- [device/manifest.rk3562t.merge](https://dev.azure.com/ampx/device/_git/manifest.rk3562t.merge) 
    - merge-manifest.yaml (https://dev.azure.com/ampx/device/_build?definitionId=3791)

2. ***hlos.rockchip-13:*** 

- [device/manifest.rockchip](https://dev.azure.com/ampx/device/_git/manifest.rockchip) (ms-rk3562t/13/0100/main)
     - optee-manifest.yaml (https://dev.azure.com/ampx/device/_build?definitionId=4227&_a=summary). 
       
     - vendor-manifest.yaml (https://dev.azure.com/ampx/device/_build?definitionId=3787)
     
     - vendor-component-manifest.yaml ( https://dev.azure.com/ampx/device/_build?definitionId=4212)

3. ***hlos.rockchip-15***:

- [device/manifest.rockchip](https://dev.azure.com/ampx/device/_git/manifest.rockchip) (ms-px30/15/0100/main)
     - optee-manifest.yaml (https://dev.azure.com/ampx/device/_build?definitionId=7344&_a=summary). 
       
     - vendor-manifest.yaml (https://dev.azure.com/ampx/device/_build?definitionId=7340)
     
     - vendor-component-manifest.yaml ( https://dev.azure.com/ampx/device/_build?definitionId=7342)
                                             


4. ***nhlos.qcm6490*** 

- [device/manifest.qcom](https://dev.azure.com/ampx/device/_git/manifest.qcom)  (ms6490/13/0200/main) 
    - adsp-manifest.yaml (https://dev.azure.com/ampx/device/_build?definitionId=4834)
    - aop-manifest.yaml(https://dev.azure.com/ampx/device/_build?definitionId=4835)                                                  
    - boot-manifest.yaml (https://dev.azure.com/ampx/device/_build?definitionId=4836)                                                    
    - btfm-manifest.yaml (https://dev.azure.com/ampx/device/_build?definitionId=4837)                                                        
    - btfm-hsp-manifest.yaml (https://dev.azure.com/ampx/device/_build?definitionId=4838&_a=summary)                                                             
    - btfw-manifest.yaml (https://dev.azure.com/ampx/device/_build?definitionId=4839)                                                            
    - cdsp-manifest.yaml (https://dev.azure.com/ampx/device/_build?definitionId=4840)                                                        
    - cpucp-manifest.yaml (https://dev.azure.com/ampx/device/_build?definitionId=4841)                                                           
    - meta-manifest.yaml (https://dev.azure.com/ampx/device/_build?definitionId=4843)                                                            
    - modem-manifest.yaml (https://dev.azure.com/ampx/device/_build?definitionId=4844)                                                       
    - tz-apps-manifest.yaml (https://dev.azure.com/ampx/device/_build?definitionId=4845)                                                         
    - tz-manifest.yaml (https://dev.azure.com/ampx/device/_build?definitionId=4846)                                                      
    - vendor-manifest.yaml (https://dev.azure.com/ampx/device/_build?definitionId=4832)                                                      
    - vendor-component-manifest.yaml (https://dev.azure.com/ampx/device/_build?definitionId=4871)                                                        
    - wlan-manifest.yaml (https://dev.azure.com/ampx/device/_build?definitionId=4847)                                                        
    - wlan-hsp-manifest.yaml (https://dev.azure.com/ampx/device/_build?definitionId=4848)
    
- [device/manifest.qcm6490.merge](https://dev.azure.com/ampx/device/_git/manifest.qcm6490.merge) 
    - merge-manifest.yaml (https://dev.azure.com/ampx/device/_build?definitionId=4833)

5. ***nhlos.qcm8550***

- [device/manifest.qcom](https://dev.azure.com/ampx/device/_git/manifest.qcom) (ms8550/13/0200/main)
    - adsp-manifest.yaml (https://dev.azure.com/ampx/device/_build?definitionId=6408)
    - aop-manifest.yaml (https://dev.azure.com/ampx/device/_build?definitionId=6410)
    - boot-manifest.yaml (https://dev.azure.com/ampx/device/_build?definitionId=6412)
    - btfw-manifest.yaml (https://dev.azure.com/ampx/device/_build?definitionId=6414)
    - cdsp-manifest.yaml (https://dev.azure.com/ampx/device/_build?definitionId=6416)
    - cpucp-manifest.yaml (https://dev.azure.com/ampx/device/_build?definitionId=6418)
    - cpuss-manifest.yaml (https://dev.azure.com/ampx/device/_build?definitionId=6420)
    - le-um-manifest.yaml (https://dev.azure.com/ampx/device/_build?definitionId=6422)
    - meta-manifest.yaml (https://dev.azure.com/ampx/device/_build?definitionId=6426)
    - spss-manifest.yaml (https://dev.azure.com/ampx/device/_build?definitionId=6424)
    - tz-apps-manifest.yaml (https://dev.azure.com/ampx/device/_build?definitionId=6428)
    - tz-manifest.yaml (https://dev.azure.com/ampx/device/_build?definitionId=6430)
    - wlan-manifest.yaml (https://dev.azure.com/ampx/device/_build?definitionId=6432)
              

6. ***nhlos.qcs8250*** 

- [device/manifest.qcom](https://dev.azure.com/ampx/device/_git/manifest.qcom)  (ms8250/13/0700/main) 
    - adsp-manifest.yaml (https://dev.azure.com/ampx/device/_build?definitionId=2648)
    - aop-manifest.yaml (https://dev.azure.com/ampx/device/_build?definitionId=2649)                                                         
    - boot-manifest.yaml (https://dev.azure.com/ampx/device/_build?definitionId=2650)                                                        
    - cdsp-manifest.yaml (https://dev.azure.com/ampx/device/_build?definitionId=2651)                                                        
    - kernel-manifest.yaml (https://dev.azure.com/ampx/device/_build?definitionId=2652)                                                      
    - meta-manifest.yaml (https://dev.azure.com/ampx/device/_build?definitionId=2653)                                                        
    - slpi-manifest.yaml (https://dev.azure.com/ampx/device/_build?definitionId=2654)                                                        
    - spss-manifest.yaml (https://dev.azure.com/ampx/device/_build?definitionId=2655)                                                        
    - tz-apps-manifest.yaml (https://dev.azure.com/ampx/device/_build?definitionId=2656)                                                         
    - tz-manifest.yaml (https://dev.azure.com/ampx/device/_build?definitionId=2657)                                                      
    - vendor-manifest.yaml (https://dev.azure.com/ampx/device/_build?definitionId=2658)                                                      
    - vendor-component-manifest.yaml (https://dev.azure.com/ampx/device/_build?definitionId=3304)
    
- [device/manifest.ms8250.merge](https://dev.azure.com/ampx/device/_git/manifest.ms8250.merge)  (ms8250/mdep/main/13/0700/main) 
    - merge-manifest.yaml (https://dev.azure.com/ampx/device/_build?definitionId=2659)
              
------------------------------------------------------------------------------------------------

***azcli.eos-11*** 

<span style="color: red;">The base OS is changed to Azure Linux in 2.64.0

<span style="color: red;">Please run tdnf install shadow-utils before using addgroup</span>
- Check latest azure-cli image -> https://mcr.microsoft.com/en-us/product/azure-cli/tags
- Check latest xsignextension version 
    - https://github.com/microsoft/Sign.Client.AzCli/releases 
    - download the .whl file locally and publish it (az artifacts universal publish --organization "https://dev.azure.com/ampx/" --feed "azcli-extensions" --name "xsignextension" --version "0.49.0" --path .) to https://dev.azure.com/ampx/dps/_artifacts/feed/azcli-extensions
- Create new branch from main (https://dev.azure.com/ampx/dps/_git/docker.images)
- Use latest version from both packages (update the [Dockerfile](https://dev.azure.com/ampx/dps/_git/docker.images?path=/images/azcli.eos-11/Dockerfile&version=GBmain&_a=contents) and [prebuild-steps.yaml](https://dev.azure.com/ampx/dps/_git/docker.images?path=/images/azcli.eos-11/prebuild-steps.yaml&version=GBmain&_a=contents) on your personal branch and test to see if they are compatible (the docker image is building successfully)
- Build image on your personal branch-> https://dev.azure.com/ampx/dps/_build?definitionId=316
- Test against builds 
    - any build that uses azcli for fetch packages (ex: [manifest.nxp](https://dev.azure.com/ampx/device/_git/manifest.nxp))
         - create new branch from ms-imx8m/13/0100/main
         - update vendor-build.yaml -> replace $(azImageVersion) for $(imageNameAzCli) image name with "latest"
         - run manifest build on your personal branch https://dev.azure.com/ampx/device/_build?definitionId=2678 that will trigger the build pipelines
    - CloudConnect signing: 
        - Go to [tooling/build.app](https://dev.azure.com/ampx/tooling/_git/build.app)
        - Create a new branch from main
        - Go to [templates/apps/vars.yaml](https://dev.azure.com/ampx/tooling/_git/build.app?path=/templates/apps/vars.yaml)
        - Change imageVersion value to "latest" instead of "lkg"
        - Go to [CloudConnect](https://dev.azure.com/ampx/app/_git/CloudConnect)
        - Create new branch from apps/main
        - Update [azure-pipeline.yaml](https://dev.azure.com/ampx/app/_git/CloudConnect?path=/azure-pipeline.yaml) to use your personal branch created above from tooling/build.app instead of main.
        - Run CloudConnect for signing (https://dev.azure.com/ampx/app/_build?definitionId=1813&_a=summary)
- If all build pipelines are successful, build docker [image](https://dev.azure.com/ampx/dps/_build?definitionId=316) on main branch and approve the lkg stage.
------------------------------------------------------------------------------------------------
***hlos.nxp-13*** 
- Build docker image -> https://dev.azure.com/ampx/dps/_build?definitionId=325 and <span style="color: red;">**DON'T**</span> approve the LKG stage
- Test the new image
     - create new branch on the manifests below from main
     - replace $(azImageVersion) with "latest" in the yaml files below
     - run below manifest pipeline on your personal branch.
- [device/manifest.nxp](https://dev.azure.com/ampx/device/_git/manifest.nxp) (ms-imx8m/13/0100/main) 
    - imxtrusty-manifest.yaml (https://dev.azure.com/ampx/device/_build?definitionId=2679)
    - vendor-manifest.yaml (https://dev.azure.com/ampx/device/_build?definitionId=2678)
    
- [device/manifest.nxp](https://dev.azure.com/ampx/device/_git/manifest.nxp) (ms-dart-mx8m/13/0100/main) 
    - imxtrusty-manifest.yaml (https://dev.azure.com/ampx/device/_build?definitionId=2691)                                                             
    - vendor-manifest.yaml (https://dev.azure.com/ampx/device/_build?definitionId=2690)

- If all builds are successful, approve the lkg stage.