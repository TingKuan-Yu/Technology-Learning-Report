

[TOC]



# Outline

- SELinux簡介

- SELinux與Linux file system關係.

- Sepolicy設置的概念及Android的簡單範例

- Android Sepolicy的marco及特別用法

- Android HAL Sepolicy的範例

- SELinux與Kernel

- Android SELinux的設定

- Android System與vendor間SELinux的合併

- Android SELinux設定檔放置位址的注意事項

- Android SELinux debug方法

  

# SELinux 简介

- SELinux 核心目的（强制访问控制 MAC，区别于 Linux 传统 DAC）
- Android 引入 SELinux 的原因（解决系统权限漏洞、隔离进程 / 应用、符合谷歌 CTS 认证要求）
- Enforcing（强制模式）、Permissive（宽容模式）、Disabled（关闭模式）的区别（同事日常调试最常用）



## SELinux 与 Linux file system 关系

- SELinux 对文件的 “标签管控”（file context，如 `u:object_r:system_file:s0`）
- 文件标签与进程标签（process context）的匹配规则（核心：进程标签需拥有文件标签的访问权限）
- Android 中文件标签的默认规则（如 `/system`、`/vendor`、`/data` 目录的默认标签差异）
- SELinux 对文件系统的支持取决于两个核心条件：文件系统是否具备扩展属性（xattr） 功能，以及内核是否为该文件系统提供了 SELinux 适配层



## Sepolicy 设置的概念及 Android 的简单范例

- Sepolicy 核心概念（主体 subject、客体 object、规则 rule、类型 type）
- Android 简单范例优化（如允许 `system_app` 访问 `/dev/video0`，贴合你之前关注的 V4L2/UV 场景）
- 基础规则格式（`allow 主体类型 客体类型:权限类别 权限;`），避免同事理解抽象



# Android Sepolicy 的 marco 及特别用法

- 最常用的 marco（宏）（如 `allow_domain`、`typeattribute`、`neverallow`，尤其是 `neverallow` 的避坑点）
- 宏的作用（简化规则编写、统一权限管理，避免重复写规则）
- 特别用法（如 `genfscon` 用于挂载文件系统的标签设置、`mlsconstrain` 用于多级安全管控）



## Android HAL Sepolicy 的范例

- 贴合你之前的 V4L2/UV 场景（如 HAL 层访问 UVC 设备 `/dev/videoX` 的 sepolicy 范例）
- HAL 层常见权限（如 `hal_camera_default` 与 `video_device` 的权限配置）
- HAL 与 kernel 交互的 sepolicy 注意点（如 HAL 进程访问 kernel 节点的权限）



## SELinux 与 Kernel

- Android 内核中 SELinux 的启用配置（`CONFIG_SECURITY_SELINUX=y`）
- Kernel 层面的 SELinux 核心组件（LSM 安全模块、selinuxfs 文件系统）
- SELinux 规则如何作用于 Kernel（如系统调用的权限检查、设备节点的访问控制）
- 你之前关注的 V4L2/UV 相关：Kernel 中 `uvcvideo` 驱动的 SELinux 标签配置



## Android SELinux 的設定

- 两种配置方式（编译时配置、运行时临时调整）
- 运行时调试命令（如 `setenforce 0` 切换宽容模式、`getenforce` 查看当前模式）
- Android 10+ 分区隔离后的 SELinux 配置差异（`system` 与 `vendor` 分区的规则隔离）



## Android System 与 vendor 间 SELinux 的合併

- 合併核心问题（规则冲突、标签不一致、权限遗漏）
- 合併方法（`vendor` 规则继承 `system` 规则、使用 `hwservice` 权限桥接）
- 合併避坑点（避免 `neverallow` 规则冲突、确保 vendor 标签与 system 标签兼容）



## Android SELinux 設定檔放置位址的注意事項

- 不同 Android 版本的配置文件路径差异（如 Android 11+ 与 Android 9 的路径区别）
- 核心路径细节（`/system/etc/selinux/`、`/vendor/etc/selinux/`，区分 `system_sepolicy` 与 `vendor_sepolicy`）
- 放置注意事项（文件权限、标签正确、避免路径错误导致 SELinux 启动失败）
- 编译时的路径配置（Android.mk/Blueprint 中 sepolicy 文件的指定方式）
- 是否應集中SEPolicy的設定?



## Android SELinux debug 方法

- 核心 debug 工具（`dmesg` 过滤 `avc: denied`、`audit2allow` 生成修复规则、`sepolicy-inject` 临时注入规则）
- 日常调试场景（如 APP 闪退、HAL 启动失败，如何通过 SELinux 日志定位权限问题）
- 结合你之前的 V4L2 场景（如 `VIDIOC_STREAMON` 失败，排查 SELinux 是否禁止进程访问 `/dev/videoX`）
- 宽容模式下的日志收集方法（确保能捕获所有被拒绝的权限请求）



