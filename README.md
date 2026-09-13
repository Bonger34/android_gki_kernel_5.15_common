# android_gki_kernel_5.15_common · Bonger34 fork

本 fork 存放 **hfdem 5.15 通用内核（android13-5.15）更新到 5.15.216** 的成果与编译测试。

## 下载（刷机包）

最新版：**[v26.09.13](https://github.com/Bonger34/android_gki_kernel_5.15_common/releases/tag/v26.09.13)**（内核 5.15.216，含 KMI 标记）

- 刷机包：**`kernel-hfdem-v26.09.13.zip`**（AnyKernel3 卡刷包）
- 配套模块：**`schedhorizon-20241107.zip`**（与上游一致，未改动）
- ⚠️ **刷入前请备份！！**
- 刷入方式（参考上游作者指引）：
  - 内核：使用 **Kernel Flasher** 刷入（作者推荐）；也可用 KernelSU / Magisk「从存储安装」，或 recovery 直接刷入
  - 模块：KernelSU 或 Magisk 应用内安装
- 仅替换内核镜像，不影响数据
- root 兼容性：只替换内核（boot 分区），不触碰 init_boot；Magisk / KernelSU（LKM）的 root 状态保留，无需重新修补
- 内核含 **KMI 标记**（`-android13-8`）：KernelSU 可自动识别内核 KMI（android13-5.15）
- SHA-256：`d7424b210c6364758d135492b72845f85ede35dfbd36fbbf1fa2cab5e165cc86`

## 模块使用（schedhorizon）

- 作用：设置 schedhorizon 调速器参数（balance / performance / powersave / fast 情景模式）
- 切换模式：支持 **scene** 或 **perapp-rs**；也可直接执行 `/data/powercfg.sh <模式>`
- 游戏请切换至 **fast** 系统调度
- balance 参数按骁龙 8 Gen 2 调校（作者平时只用 balance）；其他处理器可自行调参或不装模块
- 参考：上游作者酷安帖 https://www.coolapk.com/feed/58959644

## 文档

| 文档 | 内容 |
|---|---|
| [docs/update-2026-09.md](docs/update-2026-09.md) | **2026-09 更新完整记录**（合并结构、冲突处理、CI 记录、发布流程、验证） |
| [docs/cve-audit.md](docs/cve-audit.md) | **CVE 漏修审计报告**（54 个回退分析 + 零宽字符专项） |
| [docs/upgrade-guide.md](docs/upgrade-guide.md) | **下次升级指南**（步骤、冲突参考、检查清单） |

## 分支

| 分支 | 内容 |
|---|---|
| `android13-5.15-2026-09` | **更新后的内核分支**（推荐使用；release v26.09.13 即基于此） |
| `main`（默认） | 编译测试工作流 + 文档 |
| 其余分支 | hfdem 原分支镜像 |

## 更新内容（相对 hfdem 的 2025-07 分支）

- 基线：ACK 2025-07（5.15.185）→ **5.15.216 + 2026-09 修复**
- 上游：`android13-5.15-lts`（2026-09-11）+ `android13-5.15` 主线（2026-09-09）
- 保留全部 187 个自定义提交（boeffla wakelock blocker、LZ4 1.10、zstd 1.5.7、调度/功耗调优等）
- 新增编译修复：`lib/lz4/lz4hc.c` MIN/MAX 宏重定义
- 变更规模：4820 个文件（+77082 / -68166）

## 使用

```bash
git fetch https://github.com/Bonger34/android_gki_kernel_5.15_common android13-5.15-2026-09
git checkout -b android13-5.15-2026-09 FETCH_HEAD
```

## 编译测试状态

- ✅ **quick**（快速验证）：[run #34751431564](https://github.com/Bonger34/android_gki_kernel_5.15_common/actions/runs/34751431564)
- ✅ **full**（发布配置：LTO/CFI/KASAN/UBSAN/BTF，含 KMI 标记）：[run #34751450430](https://github.com/Bonger34/android_gki_kernel_5.15_common/actions/runs/34751450430)
  - 产物：Image.lz4 27.5MB / Image.gz 23.3MB；内核版本串 `5.15.216-hfdem-android13-8-g3b714aa31694`
  - 构建日志：28m50s，0 警告 0 错误
- 触发方式：push 到 `main` 自动运行 quick；Actions 页手动 Run workflow 可选 quick / full
- 工具链：AOSP clang r563880（与发版一致）；产物在 Actions 运行页面下载

## 上游

- https://github.com/hfdem/android_gki_kernel_5.15_common
