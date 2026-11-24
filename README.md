# phonebox-config

PhoneBox 手机端/TV 端配置同步仓库。配置文件固定为 [`config/phonebox.json`](config/phonebox.json)，分支 `main`。

## 写入约定

- 手机端是唯一写入方，持 Fine-grained PAT（仅 Contents: Read and write，仅限本仓库）。
- 上传前先 GET 取当前文件 `sha`，把 `sha` 随 PATCH 提交，防止覆盖别人（或另一台手机）的更新。
- 每次上传必须刷新 `updatedAt`（UTC，ISO8601），`appVersion` 写当前 versionName。
- 拉取端比对本地 `updatedAt`，远端更旧则不应用，避免旧配置回滚。
- `schema` 用于格式版本，升级时只兼容向前迁移，不静默丢字段。

## 字段说明

| 字段 | 含义 |
| --- | --- |
| `schema` | 文件格式版本，当前 1 |
| `updatedAt` | 最后写入时间（UTC） |
| `appVersion` | 写入方的 versionName |
| `writer` | 写入方标识（如 `phone-1.9.192`） |
| `sources.vod` | 点播源列表（名称/地址/启用状态） |
| `sources.live` | 直播源列表 |
| `sources.liveGroups` | 直播分组与频道 |
| `adBlock.on` | 广告拦截总开关 |
| `adBlock.customRules` | 从播放页盾牌添加的自定义规则，元素为域名或含斜杠的路径前缀 |
| `adBlock.stats` | 最近一次拦截统计，可选、仅用于展示 |
| `player.decoder` | 解码方式：`auto` / `hw` / `sw` |
| `player.defaultQuality` | 默认清晰度 |
| `player.speed` | 默认倍速 |
| `player.skipIntroDefault` | 片头默认跳过秒数 |
| `player.skipOutroDefault` | 片尾默认跳过秒数 |

> 单集片头片尾记录按片名存本地，不进同步文件（换设备后片名对不上会导致误跳过）。

## 读取方式

无需 token：

```
https://raw.githubusercontent.com/DSS201314/phonebox-config/main/config/phonebox.json
```

只读端请加短缓存（建议 5–10 分钟），避免每次启动都打 raw 域名。
