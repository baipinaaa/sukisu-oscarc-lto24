# 旧版补丁备份：适配 BakaSU v4.2.0-rc3 (`fa8311f6`)

这里的 3 个补丁是 **`patches/` 下同名补丁的旧版本**，只适配
`BakaSU`（原 ReSukiSU）**`v4.2.0-rc3`**
（commit `fa8311f632a215b5381ec644627c6198d1e8a13e`，`kernel/feature/selinux_hide.c` = 2145 行）。

2026-11-14 起 `patches/` 下的同名补丁已改为适配 **BakaSU `main`** 的快照
`548d1dda060ca198823d8808804d01f5209ef4a9`（`selinux_hide.c` = 1837 行，结构被上游重写）。

## 为什么两套补丁不能混用

两套补丁插入的代码逻辑完全相同，区别只在**行号锚点**：

| 补丁 | rc3 锚点 | main 锚点 |
|---|---|---|
| `55_` | `my_write_access()` 内 `sscanf(buf, "%s %s %hu", ...)` 之后（约 `:243`） | 同位置，`:336` |
| `57_` hunk1 | `my_write_context()` 里 `return orig_context_write(...)` 之后（约 `:141`） | `:246`，且 main 多了一层 `#endif`（`KSU_COMPAT_HAS_SUSFS_FEATURE_SELINUX_HIDE`） |
| `57_` hunk2 + `58_` | setprocattr 的换行剥离块 + 转换条件（约 `:342`/`:347`） | `:487-488` / `:452` |

main 的 `my_write_context()` / `my_write_access()` 把 uid 过滤包在
`#ifndef KSU_COMPAT_HAS_SUSFS_FEATURE_SELINUX_HIDE ... #endif` 里，
rc3 没有这层包裹 —— 所以 `57_` hunk1 的上下文在两边不同，**互相打不上**。

## 回退方法

```bash
cd sukisu-oscarc-lto24/patches
cp bakasu-rc3/55_add_selinux_hide_access_filter.patch      ./
cp bakasu-rc3/57_add_selinux_hide_context_guards.patch     ./
cp bakasu-rc3/58_add_selinux_hide_setprocattr_guard.patch  ./
```

同时把 workflow 的 `resukisu_ref` 默认值改回
`fa8311f632a215b5381ec644627c6198d1e8a13e`，三处必须一起改，否则构建会在
patch 步骤直接 FAIL。
