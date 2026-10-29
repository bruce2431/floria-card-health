# floria-card-health

Floria 工作区预览卡片「健康」——身体成分、跑步表现与血液生化指标的一体化看板，自带一套供 agent 调用的卡片 API。

> **仓库内全部是可公开的演示数据**（`data.example.json`，内含 14 次虚构跑步记录与 12 项虚构化验值）。真实体检数据只留在本地，`.gitignore` 已排除 `data.json`。

## 界面

- **四个视图**：全景看板 / 跑步表现 / 身高体重与体征 / 血液各项指标 / 历史明细
- **6 格 KPI**：身高体重 BMI、静息心率血压、累计里程、加权平均配速、步频步幅功率、血液达标率
- **四张图表**（原生 SVG，无图表库）：
  - 配速走势：等距训练序列轴 + 5 次滑动均值 + 步频双轨 + 目标参考线 + 十字准星悬浮
  - 步频 × 步幅散点：叠加等配速双曲线，看提速靠步频还是步幅
  - 月度跑量 + 距离档位配速箱线
  - BMI 四色标尺
- **血液指标靶向卡**：每项带参考区间色带、越界高亮、分类筛选
- **录入弹窗**：身高体重体征、单条血液指标，保存后落盘

## 卡片公开 API（供 agent 调用）

卡片被 iframe 嵌入时自动启用，命名空间 `health-pv`：

- 指令：宿主 → 卡 `{type:'health-pv-cmd', cmd, ...args}`
- 回执：卡 → 宿主 `{type:'health-pv-event', ev:'ack'|'ready'|'state', cmd, ok, result}`

| 命令 | 参数 | 用途 |
|---|---|---|
| `state` | — | 快照：次数、总里程、最佳配速、最新体重、血液异常项 |
| `query.runs` | `limit` `since` `until` | 查跑步记录 |
| `query.body` | `limit` | 查体征记录 |
| `query.blood` | `cat` `only_abnormal` | 查化验项 |
| `add.run` | `d t km dur c s w kcal load e` | 写入/覆盖跑步记录 |
| `add.body` | `date weightKg bodyFatPct restingHr bpSys bpDia…` | 按日期合并体征 |
| `add.blood` | `date source items[]` | 按代码 upsert 化验项 |
| `open` / `close` | `id` = all/run/body/blood/logs | 切换视图 |
| `scroll` / `locate` | `line` | 定位历史明细行 / 读取当前位置 |
| `reload` / `export` | — | 重拉远端数据 / 导出全量 JSON |

未注册命令与非法参数一律回 `ok:false`，不静默。

## 数据与持久化

- 页面内嵌一份默认数据（本仓库为演示数据），运行时优先后端 `data.json`
- 读取：`GET /gateway/file?label=<label>&path=.claude/preview/health/data.json`
- 写入：`POST /gateway/file/write`，带 `baseMtime` 做并发保护；写失败时 API 返回 `ok:false`
- `data.json` 结构：`{ profile, bodyRecords[], bloodReports[], runs[] }`

## 使用

放到 `<工作区>/.claude/preview/health/index.html`，并在同级 `preview.json` 注册：

```json
{
  "id": "health",
  "title": "健康",
  "icon": "dshGauge",
  "host": "view",
  "path": "health/index.html"
}
```

需要录入自己的数据时，删掉 `index.html` 里的默认 JSON 或用界面弹窗录入即可；网关会把记录写进 `data.json`。
