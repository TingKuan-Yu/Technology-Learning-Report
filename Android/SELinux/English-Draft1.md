

# A Comprehensive Introduction to Android SELinux



## Part 1: Core SELinux Concepts and Why Android Adopted It

### 1. What Is SELinux?

- **Core Objective:** Mandatory Access Control (MAC).
- DAC vs. MAC:
  - **DAC (Traditional Linux):** Based on users/groups (UID/GID); permissions are overly permissive (root can access everything).
  - **MAC (SELinux):** Based on policy rules. Even root cannot access certain resources unless explicitly allowed by policy.
- Why Does Android Need SELinux?
  - **Privilege Containment:** Limits blast radius after a compromise (e.g., a compromised process cannot access other partitions).
  - **Process Isolation:** Strict separation between apps and system processes.

### 2. Three Operating Modes (Essential for Daily Debugging)

- **Enforcing:**
   Enforces policy. Unauthorized actions are blocked and logged.
   - **CTS Requirement:** Google mandates that production devices must run in **Enforcing** mode.
   
- **Permissive:**
   Actions are **not blocked**, but `avc: denied` messages are logged in `dmesg`. This is the preferred mode for debugging.
- **Disabled:**
   Completely disabled. On Android, this is difficult to achieve and usually requires kernel boot parameter changes.

------

## Part 2: Labeling and the File System

### 1. Security Context Structure

- **Format:** `u:object_r:type:s0` (user : role : type : security level)
- **File Context (Object):** Labels for files, directories, and device nodes.
- **Process Context (Subject):** The security label assigned to a running service; while a child process typically inherits its parent’s context, a **type_transition** rule allows it to switch to a dedicated domain upon execution to ensure isolation.
- **Access Rule Matching:**
   A process (subject) must have explicit permissions to perform specific operations on a target (object).

### 2. File System Support Requirements

- **xattr (Extended Attributes):**
   Supported by filesystems such as ext4 and f2fs. Labels are stored directly on disk.
- **In-Memory Labeling:** 
   For filesystems that do not support extended attributes (`xattr`)—such as `sysfs`, `proc`, and `debugfs`—labels are assigned using the `genfscon` statement. This mechanism allows the kernel to map specific paths or prefixes to a security context at mount time. Unlike `file_contexts`, which labels files on a persistent disk, `genfscon` is essential for labeling in-memory and virtual filesystems where labels cannot be stored physically.

------

## Part 3: Android Sepolicy Configuration and Syntax

### 1. Basic Rule Format

- **Syntax:** allow <subject_type> <object_type>: { permissions };
- **Example:** Allow `system_app` to access V4L2 devices. allow system_app video_device:chr_file { read write open ioctl };

### 2. Common Macros and Special Constructs

- Common Macros:
  - `unix_socket_connect()` – simplifies socket connection rules.
  - `typeattribute` – associates a type with an attribute (e.g., `domain`).
- **Neverallow (Critical):**
   Defines actions that are **absolutely forbidden**. Violations are caught at compile time.
   If hit, the correct solution is to redesign permissions—not to add more `allow` rules.
- **genfscon:**
   Used to label filesystems that do not support extended attributes (e.g., `/proc` nodes).

------

## Part 4: HAL Layer and V4L2/UVC Practical Examples

### 1. HAL Sepolicy Example

- **Scenario:** A HAL process needs access to `/dev/videoX`.
- File Context Definition:
  - In `device/qcom/sepolicy/.../file_contexts`: /dev/video[0-9]+  u:object_r:video_device:s0
- **Policy Rules (`hal_camera_default.te`):** allow hal_camera_default video_device:chr_file rw_file_perms; allowxperm hal_camera_default video_device:chr_file ioctl { VIDIOC_STREAMON VIDIOC_ENUM_FMT };

### 2. HAL–Kernel Interaction Considerations

- Access to nodes such as `/sys/class/video4linux` requires additional permissions.
- **Key Principle:** Least privilege.
   HAL processes should never have `sysadmin` privileges.

------

## Part 5: Core Mechanisms — SELinux and the Kernel

### 1. Kernel Components

- **CONFIG_SECURITY_SELINUX=y:** Kernel build option.
- **LSM (Linux Security Modules):** Hook points used by SELinux in the kernel.
- **selinuxfs (`/sys/fs/selinux`):** Interface between kernel space and user space.

### 2. UVC Video Driver Interaction

- When the kernel executes `v4l2_open`, the LSM hook checks whether the calling process has `open` permission on the `video_device` label.

------

## Part 6: Policy Configuration and Merging (System vs. Vendor)

### 1. Partition Separation (Android 10+)

- **System Policy:** `/system/etc/selinux`
   Defines core platform components.
- **Vendor Policy:** `/vendor/etc/selinux`
   Defines SoC- and vendor-specific drivers.
- **Merge Mechanism:**
   Both are merged at build time into `precompiled_sepolicy`.

### 2. Common Pitfalls During Merging

- **Avoid Global Changes:**
   Do not modify system label attributes from the vendor side.
- **Attribute Conflicts:**
   Vendor policy cannot violate system `neverallow` rules; otherwise, the build fails.
- **Centralization vs. Distribution:**
   Place feature-specific rules in the corresponding `.te` files—avoid dumping everything into `device.te`.

------

## Part 7: Debugging and Practical Techniques

### 1. Essential Debug Commands

- `getenforce` / `setenforce 0` – Temporarily switch modes.
- `ls -Z` – View file labels.
- `ps -AZ` – View process labels.

### 2. How to Analyze AVC Denials

- **Log Collection:** dmesg | grep avc logcat | grep avc
- **Example:** avc: denied { ioctl } for comm="camerahalserver" path="/dev/video0" scontext=u:r:hal_camera_default:s0 tcontext=u:object_r:video_device:s0
- Tools:
  - **audit2allow:** Converts logs into policy rules: dmesg | audit2allow -p out/target/.../sepolicy ⚠️ **Warning:** Never copy rules blindly—always review for security impact.

### 3. V4L2-Specific Case Study

- If 

  ```
  VIDIOC_STREAMON
  ```

   fails with 

  ```
  EACCES
  ```

   or 

  ```
  EPERM
  ```

  :

  1. Temporarily run `setenforce 0`.
  2. If it works, SELinux is blocking a specific `ioctl`.
  3. Use `allowxperm` to precisely grant the required ioctl codes.

------

## Conclusion: SELinux Is Not an Obstacle, but a Shield

- Use **Permissive** mode early in development to collect complete logs.
- Before release, ensure all policies work correctly in **Enforcing** mode and pass CTS.

------

If you want, I can also:

- Tighten this into **executive-level slides**
- Convert it into **speaker notes**
- Create a **one-page SELinux debugging cheat sheet**
- Align terminology with **AOSP / Google documentation wording**

Just tell me the target audience.