# Phigros LSPosed 模块

**最新正式版：[v0.8.4](https://github.com/yskzctx/phigros-lsposed/releases/tag/v0.8.4)**

> **未经过真机 LSPosed 验收。** 电脑端能验证编译、签名、原生库齐全、dex、manifest 声明和回归测试；不能替代真实 LSPosed 环境下的注入与整局游玩。

运行时附加，**不修改游戏 APK、不改签名、不重打包**。保留宿主登录、联网和原版上传流程，不绕过服务器、账号、购买或反作弊校验。

| 产物 | 目标 |
|---|---|
| `Phigros-Lab-LSPosed-157-161-arm64-v0.8.4-TEST.apk` | 4.0.1(157) **或** 4.1.0(161) arm64 |

覆盖安装后强停目标游戏再打开。

---

## 支持 4.1.0

按**函数序言匹配**自动选钩子表，不按版本号猜：

- 遍历候选表（157 用 `native/core/expected-hooks.h`，4.1.0 用 `native/core/expected-hooks-161.h`）
- 选第一张 9 个函数序言全部匹配当前 `libil2cpp.so` 的表
- 157 表不匹配时自动落到 4.1.0 表
- `bad_call` 随表走（157 `0x1d875c4` / 4.1.0 `0x1d89dcc`）

`GameProfile` 接受 157 和 161，各自校验 libil2cpp 的 sha256。

---

## 功能

### 判定保护（不改输入）

面板「这局怎么打」有两个保护选项：

| 选项 | 行为 |
|---|---|
| **判定保护** | **完全不碰输入**。你自己打，判定产生后 Good/Bad/Miss 强制改成 Perfect |
| 自动补全 | 替你按键，全 Perfect（旧行为） |

保护类型（判定保护下生效）：

| 类型 | 效果 |
|---|---|
| 保护 AP | 任何非 Perfect 都改成 Perfect |
| 保护 FC | 只有 Bad / Miss 改成 Perfect，Good 保留 |
| 保护分数 | 只在当前分数低于保底时改判 |

**不需要开自动游玩。**

### 结算页强制显示

- 8 个独立字段：分数、准确率、P/G/B/M、最大连击
- 结算符号：不干预 / Φ（全 Perfect）/ FC（无 Bad、Miss）
- Good 早/晚：可选开关
- 独立开关，接受真实一局不可能出现的组合

### 全解锁 / 全部 AP / 目标 RKS

- 曲库 **1072 谱面 / 329 首**（4.1.0 数据）
- 「全解锁 ＋ 全部 AP」：解锁全部曲目、收藏品、曲绘、头像并设置全 AP
- 补全逻辑：随包数据是 157 快照（1037 谱面记录、646 个歌曲解锁标志），缺的部分自动补齐
- 目标历史 RKS：四位小数，上限 17.32（= 全 AP）

### 导入 / 导出存档

- 导入游戏原始 `playerprefs.xml` 或面板导出的 ZIP
- 先检查（来源、条目数、大小）→ 确认 → 自动备份 → 写入
- 备份列表一键回退；恢复限制在备份目录内
- 不解密不重加密，账号身份完整保留
- 导入后需完全退出重启

### 我的自制章节 / 音符皮肤 / 初始 Data

- 自制章节：导入带音乐和插画的 `.pez` / ZIP 谱包
- 音符皮肤：标准 Phira 音符图包（图片上限 8192 边长 / 3200 万像素）
- 初始 Data：准备至少 10000 MB 本地 Data

---

## 边界：绝不隐藏或阻止登录

登录隐藏只属于离线修改版（它本来不联网）。模块运行在原版游戏里，登录、账号、云同步必须正常。

`tools/check_module_no_login_block.py` 强制检查：源码无隐私符号、产物无 `libphigros_privacy.so`、原生库不导出相关符号。

---

## 构建

```bash
python build_module.py \
    --sdk <Android SDK> --ndk <NDK> --xposed-api <api-82.jar> \
    --keystore <jks> --alias <alias> --password-file <txt> \
    --resources-from-apk <离线修改版 APK> \
    --chapter-so <libphigros_chapter.so> \
    --out <输出目录>
```

`--resources-from-apk` 按**内容**校验（包名、`assets/offline/`、无 INTERNET），不是哈希白名单。

### 原生库

模块必须自带它加载的每个库，缺任一启动即崩：

- `libphigros_lsp.so` — 判定/结算 hook
- `libphigros_skins.so` — 皮肤渲染
- `libphigros_chapter.so` — 自制章节引擎（Rust 构建，见 `chapter-player/`）

---

## 已知限制

- **4.1.0 皮肤渲染不可用**：12 个 Unity 辅助 RVA 已恢复 7 个，5 个未解。已做安全降级——库能加载、不装错 hook、不崩，但功能不可用。判定保护、结算、成绩、曲库不受影响。
- 未真机验收。

---

## 相关仓库

- [Phira 模块](https://github.com/yskzctx/phira-lsposed)
- [Phigros 离线修改版](https://github.com/yskzctx/phigros-offline-lab)

## 免责声明

见 [DISCLAIMER.md](DISCLAIMER.md)。
