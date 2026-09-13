# android_gki_kernel_5.15_common · Bonger34 fork

本 fork 存放 **hfdem 5.15 通用内核（android13-5.15）更新到 5.15.216** 的成果与编译测试。

## 下载（刷机包）

最新版：**[v26.09.13](https://github.com/Bonger34/android_gki_kernel_5.15_common/releases/tag/v26.09.13)**（内核 5.15.216）

- 刷机包：**`kernel-hfdem-v26.09.13.zip`**（AnyKernel3 卡刷包）
- 刷入方式：KernelSU / Magisk 应用内「从存储安装」，或 recovery 直接刷入
- 仅替换内核镜像，不影响数据
- SHA-256：`6013ac278852f84aeece050d94e47146602eee0e14234506181e493714b290e2`

## 分支

| 分支 | 内容 |
|---|---|
| `android13-5.15-2026-09` | **更新后的内核分支**（推荐使用；release v26.09.13 即基于此） |
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

## 编译测试状态

- ✅ **quick**（快速验证）：[run #34742068392](https://github.com/Bonger34/android_gki_kernel_5.15_common/actions/runs/34742068392)
- ✅ **full**（发布配置：LTO/CFI/KASAN/UBSAN/BTF）：[run #34742092026](https://github.com/Bonger34/android_gki_kernel_5.15_common/actions/runs/34742092026)
  - 产物：Image.lz4 27.4MB / Image.gz 23.3MB；内核版本串 `5.15.216-hfdem-g3b714aa31694`
  - 构建日志：0 警告 0 错误
- 触发方式：push 到 `main` 自动运行 quick；Actions 页手动 Run workflow 可选 quick / full
- 工具链：AOSP clang r563880（与发版一致）；产物在 Actions 运行页面下载

## 上游

- https://github.com/hfdem/android_gki_kernel_5.15_common
