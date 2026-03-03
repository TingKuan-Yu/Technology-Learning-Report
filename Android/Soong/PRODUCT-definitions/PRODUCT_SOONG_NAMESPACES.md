[TOC]



# `PRODUCT_SOONG_NAMESPACES`

`PRODUCT_SOONG_NAMESPACES` is a critical configuration in Android's build system (Soong) that **defines the build namespaces for product-specific modules** (e.g., device-specific libraries, apps, or configs). It ensures Soong can correctly locate, resolve dependencies for, and avoid naming conflicts with modules defined in device/vendor/product directories. Below is a detailed guide on its usage, purpose, syntax, and best practices.


## 1. Core Purpose of `PRODUCT_SOONG_NAMESPACES`
Before diving into usage, understand why it’s needed:
- Soong (Android.bp’s build system) uses **namespaces** to isolate modules (e.g., two modules named `libutils` in different directories won’t conflict if in separate namespaces).
- `PRODUCT_SOONG_NAMESPACES` tells Soong: *"These directories contain product-specific modules—treat each as a distinct namespace."*
- Without it, Soong may fail to find your custom modules (e.g., device-specific HALs) or throw "module not found" errors when resolving dependencies.


## 2. Where to Define `PRODUCT_SOONG_NAMESPACES`
Define it in **product configuration files** (not Android.bp) that are part of your product’s build flow:
- Common files: `device/<vendor>/<device>/device.mk`, `product/<vendor>/<product>/product.mk`, or `vendor/<vendor>/<device>/vendor.mk`.
- Syntax: It’s a build variable (similar to `PRODUCT_NAME`), so use `:=` to assign values (space-separated directory paths).


## 3. Basic Syntax & Usage
### 3.1 Minimal Example
Suppose you have a device-specific module in `device/myvendor/mydroid/custom_modules/` (with Android.bp files). To register this directory as a Soong namespace:

In `device/myvendor/mydroid/device.mk`:
```makefile
# Define Soong namespaces for product-specific modules
PRODUCT_SOONG_NAMESPACES += \
    device/myvendor/mydroid/custom_modules \
    vendor/myvendor/mydroid/hal_modules  # Add multiple namespaces (space/newline-separated)
```

### 3.2 Key Rules for Paths
- Paths are **relative to the Android source root** (e.g., `device/myvendor/mydroid` is correct, not `./custom_modules`).
- Use forward slashes (`/`) even on Windows.
- Add one namespace per directory that contains Android.bp files (don’t nest namespaces unless necessary).


## 4. Critical Use Cases (Why You Need It)
### 4.1 Case 1: Expose Custom Modules to the Product
If you write a device-specific library (e.g., `libmydevicehal`) in `vendor/myvendor/mydroid/hal_modules/Android.bp`, adding this directory to `PRODUCT_SOONG_NAMESPACES` lets other modules (e.g., system apps, framework) depend on it via `shared_libs: ["libmydevicehal"]`.

Example Android.bp in `vendor/myvendor/mydroid/hal_modules/`:
```bp
cc_library_shared {
    name: "libmydevicehal",  # Module name
    srcs: ["hal_impl.cpp"],
    shared_libs: ["liblog", "libutils"],
}
```

Without `PRODUCT_SOONG_NAMESPACES += vendor/myvendor/mydroid/hal_modules`, Soong will throw:
```
error: module "libmydevicehal" not found (required by ...)
```

### 4.2 Case 2: Avoid Module Naming Conflicts
If two directories define a module with the same name (e.g., `libconfig`), namespaces isolate them. For example:
- Namespace `device/myvendor/mydroid/config_v1` has `libconfig` (v1).
- Namespace `device/myvendor/mydroid/config_v2` has `libconfig` (v2).

To reference a specific version, use the namespace prefix in dependencies:
```bp
cc_binary {
    name: "myapp",
    srcs: ["main.cpp"],
    shared_libs: [
        "device/myvendor/mydroid/config_v2:libconfig",  # Explicitly use v2 from its namespace
    ],
}
```

### 4.3 Case 3: Integrate Vendor/Device-Specific HALs/Apps
Most device-specific components (e.g., camera HAL, custom system apps) live in `vendor/` or `device/` directories. `PRODUCT_SOONG_NAMESPACES` is mandatory to make these modules visible to the core Android build (e.g., framework, system server).


## 5. Advanced Usage: Namespace Configuration (soong_namespace.json)
For complex namespaces (e.g., setting default visibility, excluding paths), add a `soong_namespace.json` file in the namespace directory. This file lets you fine-tune how Soong processes the namespace.

### Example `soong_namespace.json`
In `device/myvendor/mydroid/custom_modules/soong_namespace.json`:
```json
{
    "name": "mydevice_custom",  # Optional: Explicit namespace name (defaults to directory path)
    "visibility": [
        "//device/myvendor/mydroid/...",  # Allow modules in this path to use my namespace
        "//system/core/..."               # Allow core system modules to use my namespace
    ],
    "exclude_dirs": [
        "test"  # Exclude the "test/" subdirectory from the namespace
    ]
}
```

#### Key Fields:
- `visibility`: Restricts which modules can depend on modules in this namespace (security/best practice to limit exposure).
- `exclude_dirs`: Skips subdirectories that don’t contain buildable modules (speeds up build).
- `name`: Overrides the default namespace name (default is the directory path relative to source root).


## 6. Common Pitfalls & Fixes
### Pitfall 1: "Module Not Found" Error
- Cause: The directory containing the module is not in `PRODUCT_SOONG_NAMESPACES`.
- Fix: Add the module’s parent directory to `PRODUCT_SOONG_NAMESPACES` (ensure the path is relative to source root).

### Pitfall 2: Naming Conflicts
- Cause: Two modules in different namespaces have the same name, and no explicit namespace prefix is used.
- Fix: Reference the module with its full namespace path (e.g., `vendor/myvendor/mydroid/hal:libmydhal`).

### Pitfall 3: Visibility Issues ("Module X is not visible to Y")
- Cause: The namespace’s `visibility` (in `soong_namespace.json`) blocks the dependent module.
- Fix: Update `visibility` to include the path of the dependent module (e.g., `"//packages/apps/MyApp/..."`).

### Pitfall 4: Relative Paths in `PRODUCT_SOONG_NAMESPACES`
- Cause: Using paths like `./custom_modules` (relative to the .mk file) instead of source-root-relative paths.
- Fix: Use full relative paths (e.g., `device/myvendor/mydroid/custom_modules`).


## 7. Verification: Check if Namespaces Are Applied
To confirm your namespace is registered correctly:
1. Build your product (e.g., `m myproduct`).
2. Check the Soong build log for lines like:
   ```
   Soong namespace registered: device/myvendor/mydroid/custom_modules
   ```
3. Use the `soong_ui` tool to list namespaces (run from source root):
   ```bash
   build/soong/soong_ui.bash --dump-namespaces
   ```
   This will print all registered namespaces and their modules.


## 8. Official References
- [Soong Namespaces (Android Official Docs)](https://source.android.com/docs/setup/build/soong/namespaces)
- [PRODUCT_SOONG_NAMESPACES (Android Build Variables)](https://source.android.com/docs/setup/build/variables#product_soong_namespaces)
- [soong_namespace.json Schema](https://source.android.com/docs/setup/build/soong/namespaces#namespace-configuration)


## Summary
- `PRODUCT_SOONG_NAMESPACES` registers directories as Soong build namespaces for product-specific modules.
- Define it in `device.mk`/`product.mk` with source-root-relative paths.
- Critical for exposing custom modules (HALs, apps) and avoiding naming conflicts.
- Use `soong_namespace.json` for advanced configuration (visibility, exclusions).

By following these rules, you’ll ensure Soong correctly resolves your device/vendor modules and integrates them into the Android build.