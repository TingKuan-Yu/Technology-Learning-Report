# parameters in download_spl_from_android_manifest.yam



## Global Parameters

- code snippet

  ```yaml
  parameters:
  - name: splPackageVersion
    displayName: "Publish the downloaded SPL package version (only if you want to publish the SPL package which was downloaded from Google Drive)"
    type: string
    default: "Skip"
  
  - name: securityBulletinPkgName
    displayName: "The package name for security bulletin package"
    type: string
    default: "spl-from-android-security-bulletins"
    values:
    - "spl-from-android-security-bulletins"
    - "spl-from-android-security-tags"
  
  - name: securityBulletinPackageVersion
    displayName: "Security Bulletin Package Version"
    type: string
    default: "2025.1028.1"
  
  - name: securityTags
    displayName: "Security Tags, e.g., android-security-13.0.0_r10;android-security-13.0.0_r9"
    type: string
    default: "android-security-13.0.0_r10;android-security-13.0.0_r9"
  ```

- splPackageVersion

  - \- ${{ if ne(parameters.splPackageVersion, 'Skip') }}:
    - 若不是'Skip', 表有版號,則由security-patch artifact 下載後轉發.
  - \- ${{ if eq(parameters.splPackageVersion, 'Skip') }}:
    - 若是'Skip', 表無版號,則不由security-patch artifact 下載後轉發.

- securityBulletinPkgName

  - 



