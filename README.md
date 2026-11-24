# phonebox-config

PhoneBox 手机端 / TV 端配置同步仓库。配置文件固定为 [`config/phonebox.json`](config/phonebox.json)，分支 `main`。

## 谁写谁读

- **手机端**是唯一写入方（App 内「我的 → 播放设置 → 同步 → 配置同步（GitHub）」），需要 Fine-grained PAT：Repository access 只选本仓库，Permissions 只开 `Contents: Read and write`。
- **TV / 平板**只读拉取。公开仓库可不带 token（走 raw 地址）；带 token 也能读私有仓库。
- token 只存在设备本机设置里，不写进同步文件，不进日志。

## 同步规则

- 上传前先 GET 取当前文件 `sha`，PUT 时带上；另一台设备刚改过会返回 409，先「拉取」再上传。
- 上传必刷新 `updatedAt`（UTC，`yyyy-MM-ddTHH:mm:ssZ`），`appVersion` 写当前 versionName。
- 拉取端比对 `updatedAt` 与本地“上次应用”时间：远端更旧则不覆盖（App 内可用「强制覆盖」跳过）。
- `schema` 为格式版本，远端 `schema` 大于本地支持值时拒绝应用，提示升级 App。

## 字段

| 字段 | 类型 | 含义 |
| --- | --- | --- |
| `schema` | int | 格式版本，当前 `1` |
| `updatedAt` | string | 最后写入时间（UTC） |
| `appVersion` | string | 写入方 versionName |
| `writer` | string | 写入方标识，如 `phone-1.9.193` |
| `sources.vodRaw` | string | 点播/综合配置**原文**（原样保存，拉取时走 `ConfigStore.saveConfig`） |
| `sources.vodUrl` | string | 该配置的来源地址，可为空 |
| `sources.live` | array | 直播源列表，元素为 `{name, raw, url}`，`raw` 为 M3U/直播配置原文 |
| `sources.activeLiveName` | string | 当前激活的直播源名称（跨设备按名字重新激活） |
| `adBlock.on` | bool | 广告拦截总开关 |
| `adBlock.customRules` | array | 从播放页盾牌添加的自定义规则：域名，或含斜杠的路径前缀，或单分片完整地址（本集专用） |
| `player` | object | 播放偏好：`quality_mode` `auto_next` `external_enabled` `theme_mode` `accent` `seek_sensitivity` `jump_step_sec` `decoder` |

**不同步**：播放记录、收藏、单集片头片尾。这些按片名/本地进度存，跨设备会错位。

## 只读端用法

```
https://raw.githubusercontent.com/DSS201314/phonebox-config/main/config/phonebox.json
```

建议加 5–10 分钟缓存，避免每次启动都打 raw 域名。

## 手工检查

改完文件后用编辑器打开 `config/phonebox.json` 验证是合法 JSON；`updatedAt` 用 UTC 时间，格式必须能被 `yyyy-MM-dd'T'HH:mm:ss'Z'` 解析，否则拉取端会当成“0 时间”处理。
