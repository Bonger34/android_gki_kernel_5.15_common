# 下次升级指南（hfdem 5.15 内核更新流程）

> 本指南基于 **2026-09 更新**（5.15.185 → 5.15.216）的完整经验整理。
> 该次更新的完整记录见 [update-2026-09.md](update-2026-09.md)，安全审计见 [cve-audit.md](cve-audit.md)。

## 现状（起点）

| 项目 | 内容 |
|---|---|
| 更新分支 | `android13-5.15-2026-09`（基于 5.15.216） |
| 构建 | GitHub Actions 工作流（`main` 分支的 `.github/workflows/build.yml`） |
| 发布 | Release [v26.09.13](https://github.com/Bonger34/android_gki_kernel_5.15_common/releases/tag/v26.09.13) |
| 内核版本串 | `5.15.216-hfdem-android13-8-g3b714aa31694` |

## 升级流程（全程可在 GitHub/手机完成，无需本地编译）

### 1. 克隆并准备

```bash
git clone https://github.com/Bonger34/android_gki_kernel_5.15_common.git
cd android_gki_kernel_5.15_common
git checkout android13-5.15-2026-09
```

### 2. 拉取上游

```bash
# ACK（Android Common Kernel）
git remote add ack https://android.googlesource.com/kernel/common
git fetch ack 'refs/heads/android13-5.15-lts:refs/remotes/ack/lts'
git fetch ack 'refs/heads/android13-5.15:refs/remotes/ack/main'

# hfdem 原仓库（如果作者有新提交，建议一并拉取）
git remote add hfdem https://github.com/hfdem/android_gki_kernel_5.15_common.git
git fetch hfdem
```

### 3. 合并（顺序：先 lts，后主线）

```bash
git merge ack/lts     # 稳定版线（通常更新）
git merge ack/main    # 主线（补 lts 尚未包含的修复）
```

### 4. 冲突处理参考（2026-09 经验，很可能再次出现）

| 文件 | 处理方式 |
|---|---|
| `drivers/staging/android/ashmem.c` | 保留本地重写版（上游改动通常只涉及已移除的路径） |
| `fs/fs-writeback.c` | 组合两边：`if (dirtytime_expire_interval)` 判断 + `system_power_efficient_wq` |
| `mm/page_reporting.c` | 组合为 `system_freezable_power_efficient_wq` |
| `lib/zstd/zstd_internal.h` | 保持删除（本地 zstd v1.5.7 已取代） |
| `lib/lz4/lz4hc.c` | 若报 `MIN/MAX macro redefined` → 加 `#undef MIN` / `#undef MAX`（参照 lz4.c 的写法） |

> 编译时如遇其他宏重定义/新警告，优先对照 `lz4.c`、`zstd` 等"已有正确写法"的文件。

### 5. 构建（GitHub Actions）

1. 把合并后的分支推送到 fork：
   ```bash
   git push origin android13-5.15-2026-09
   ```
2. 在 Actions 页面手动触发工作流：先 **quick** 验证，通过后跑 **full**（发布配置）
3. 或直接 push 到 `main` 触发 quick

工作流已内置：
- AOSP clang r563880 下载（3 个源自动切换 + 缓存）
- 40GB swap + zswap（LTO 链接必需，不足会报 OOM）
- KMI 标记构建（`BRANCH=android13-5.15 KMI_GENERATION=8`）与版本串校验

### 6. 打包 Release

1. 从 full 构建的 Artifacts 下载 `Image.gz`
2. 复用上一版 release zip 的 AnyKernel3 结构（`anykernel.sh` + `META-INF/` + `tools/`），仅替换 `Image.gz`
   - 更新 `anykernel.sh` 里的 `kernel.string`（版本号）
   - 保持各文件权限（tools 755，其余 644）
3. 打包 zip → 创建 release（tag 格式 `vYY.MM.DD`）→ 上传内核 zip
4. 配套模块 `schedhorizon-20241107.zip` 原样附带（见下方兼容性说明）
5. 更新 README（版本、SHA-256、版本串、CI 记录）

## 发布前检查清单

- [ ] 版本串含 KMI 标记（`-android13-8`）—— KernelSU 识别需要
- [ ] full 构建 0 警告 0 错误
- [ ] Release 说明的链接用 GitHub 渲染 API 验证（防粘连）
- [ ] SHA-256 与本地文件一致
- [ ] root 兼容性说明（刷内核不碰 init_boot，无需重修补）

## 常见问题

| 问题 | 处理 |
|---|---|
| 构建 OOM（LTO 阶段） | 检查工作流 swap 步骤是否生效（需 ≥20GB；已有硬校验） |
| clang 下载失败 | 工作流有 3 个源自动切换，重跑即可 |
| `MIN/MAX macro redefined` | 见冲突表（lz4hc.c 加 #undef） |
| KernelSU 不识别 KMI | 检查版本串是否含 `-android13-8`（构建时需带 BRANCH/KMI_GENERATION） |
| 刷入后 root 丢失？ | 不会——刷内核只写 boot 分区；root 补丁在 init_boot（KSU LKM / Magisk） |

## 模块兼容性（schedhorizon）

- 模块为**纯用户空间调参脚本**，与内核版本解耦
- 检查内核侧这些接口是否存在即可（`schedhorizon` 调速器、`efficient_freq`、`up_delay`、`scaling_min_freq_limit`、`uclamp.min.multiplier`）
- 上游实践：连续 4 个内核版本沿用同一模块（20241107），仅当模块自身改动才更新

## 参考

- ACK 仓库：https://android.googlesource.com/kernel/common（分支 `android13-5.15-lts`、`android13-5.15`）
- 上游作者（hfdem）酷安帖：https://www.coolapk.com/feed/58959644
- 内核 CVE 数据库：https://git.kernel.org/pub/scm/linux/security/vulns.git
