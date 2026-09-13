# android_gki_kernel_5.15_common · Bonger34 fork

本 fork 存放 **hfdem 5.15 通用内核（android13-5.15）更新到 5.15.216** 的成果与编译测试。

## 分支

| 分支 | 内容 |
|---|---|
| `android13-5.15-2026-09` | **更新后的内核分支**（推荐使用） |
| `main`（默认） | 编译测试工作流 + 本文档 |
| 其余分支 | hfdem 原分支镜像 |

## 更新内容（相对 hfdem 的 2025-07 分支）

- 基线：ACK 2025-07（5.15.185）→ **5.15.216 + 2026-09 修复**
- 上游：`android13-5.15-lts`（2026-09-11）+ `android13-5.15` 主线（2026-09-09）
- 保留全部 187 个自定义提交（boeffla wakelock blocker、LZ4 1.10、zstd 1.5.7、调度/功耗调优等）
- 新增编译修复：`lib/lz4/lz4hc.c` MIN/MAX 宏重定义
- 变更规模：4819 个文件（+77080 / -68166）

## 使用

```bash
git fetch https://github.com/Bonger34/android_gki_kernel_5.15_common android13-5.15-2026-09
git checkout -b android13-5.15-2026-09 FETCH_HEAD
```

## 编译测试

- 本仓库内置 GitHub Actions 工作流：push 到 `main` 自动运行，或 Actions 页手动触发（可选 quick / full）
- 工具链：AOSP clang r563880（与发版一致）
- 产物：Image.lz4 / Image.gz（在 Actions 运行页面下载）

## 上游

- https://github.com/hfdem/android_gki_kernel_5.15_common
