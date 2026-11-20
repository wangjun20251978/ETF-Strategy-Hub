# ETF 基金策略看板库

每天自动生成/更新的基金策略看板。**单文件 HTML、黑白红配色、离线可用、数据不出本机**，配套闲鱼文案。思路来自抖音/小红书/闲鱼/头条高赞内容与经典量化方法，全部由公开日 K 线机械复算。

> ⚠️ 仅供框架演示与信息整理，**不构成投资建议**。基金有风险，投资需谨慎。

---

## 一、当前 5 套策略

| # | 策略 | 思路来源 | 综合分模型 |
|---|---|---|---|
| 1 | 估值定投打分 | 小红书晒回本笔记同款 | V估值50% + F资金30% + M动量20% |
| 2 | 网格交易参数表 | 抖音『三步搭网格』 | 价格位置40% + 波动35% + RSI25% |
| 3 | 动量轮动排名 | 头条/叩富网轮动策略 | M动量45% + F资金30% + 趋势25% |
| 4 | 核心+卫星配置 | 头条『三套傻瓜策略』 | M35% + F35% + 趋势30%（挑卫星） |
| 5 | 均线趋势择时 | 叩富网散户量化首选 | 均线MACD40% + M30% + RSI30% |

打开 `index.html` 看导航页，点进去看每套看板和对应闲鱼文案。

---

## 二、目录结构

```
ETF每日策略库/
├── index.html              # 首页导航（自动生成）
├── 看板/                    # 每套策略一个单文件 HTML（自动生成）
├── 闲鱼文案/                # 每套策略一份闲鱼文案（自动生成）
├── strategies/              # ★ 策略配置 JSON —— 每天新造策略只需动这里
│   └── 0X_策略名.json
├── src/
│   ├── data.py              # 数据源 + 因子计算（一般不用改）
│   ├── render.py            # 看板/文案渲染器（一般不用改）
│   └── build.py             # 主构建脚本
└── .github/workflows/daily.yml  # 每天定时重算 + 自动推送
```

---

## 三、每天怎么用

### A. 定时更新数据（GitHub Actions 自动）

把仓库 push 到 GitHub 后，开启 GitHub Pages（Settings → Pages → Source 选 main / root），
`.github/workflows/daily.yml` 会在每个交易日收盘后自动重算全部看板并推送。
Powered by `GITHUB_TOKEN`，**不需要你配任何密钥**。首次 push 后可在 Actions 页手动点 "Run workflow" 验证。

### B. 每天 AI 新造 5 个策略

这套的架构是「**配置驱动**」——新增策略不用写一行代码，只要让 AI 产出符合 schema 的 JSON 放进 `strategies/`，再跑一次 `python src/build.py`，看板和文案就自动生成。

**轮换规则（关键，防止策略无限膨胀）**：
- `01_~05_` 这 5 个是**核心策略，永久保留，不要删**。
- 每天新造的 5 个策略用 **`d` + 日期前缀**命名，如 `d20261121_超跌反弹.json`。
- 每次生成新的一批前，**先删掉上一次的 `strategies/d*.json`**，只保留当天这 5 个（主页始终 = 5 核心 + 5 当日精选 = 10 个）。

**给 AI 的指令模板**（已配成本机定时任务，每天自动跑；也可手动复制执行）：

```
1. 删除 strategies/ 下所有 d 开头的 json（旧的一日精选）。
2. 联网搜索今天抖音/小红书/闲鱼/头条上高赞的基金/ETF策略思路，提炼 5 个新的、
   可机械复算、且与现有 5 个核心策略不重复的策略。
3. 按 README「策略配置 schema」写成 5 个 JSON，文件名 dYYYYMMDD_策略名.json，放进 strategies/。
   每个策略 3~5 个因子加权打分，因子只能用 src/data.py compute_metrics 返回的字段
   （s_m/s_v/s_f/s_rsi/s_trend/s_vol/r20/r60/pct250/drawdown/f_ratio/rsi14 等）。
4. 运行 python src/build.py，确认 10 个看板（5核心+5当日）都生成成功、无报错。
5. git add -A && git commit -m "每日精选 YYYY-MM-DD"（本地提交，不 push）。
```

---

## 四、策略配置 schema

一个 `strategies/XX_名字.json` 的字段：

| 字段 | 说明 |
|---|---|
| `id` | 英文小写+连字符，作文件名，如 `valuation-dca` |
| `name` | 看板标题 |
| `subtitle` | 首页简介 |
| `source` | 思路来源说明 |
| `model_line` | 模型一句话（"综合分 = A 40% + B 60%"） |
| `pool` | `"hot20"`（同花顺热榜前20）或 `[[代码,名称,板块],...]` 固定池 |
| `factors` | `[{"key":"字段名","label":"显示名","w":权重}]`，key 用 compute_metrics 返回的因子分字段（s_ 开头的标准分 -100~100） |
| `bands` | 信号分档，按 `min` 从高到低排：`[{"min":45,"label":"加码定投","cls":"b-strong","pos":"每期2倍"}]`。cls 可选 b-strong/b-ok/b-mid/b-bad |
| `columns` | 明细表列：`[{"key":"字段名","label":"列名","fmt":"格式"}]`。fmt 可选 pn2/pn0/num2/num0/pct1/price/x/signal/rsi/band/raw |
| `table_note` | 表格下方注释 |
| `verdict` | 结论先行文字，可用占位符 `{top} {topcode} {topscore} {strong} {n}` |
| `method_rows` | 口径说明：`[["因子名","计算方式"],...]` |
| `risks` | 风险提示字符串数组 |
| `xianyu` | 闲鱼文案：`titles`（3个标题）、`body`（正文，占位符 `{top3} {date} {n}`）、`keywords`（关键词数组）、`price`（版本定价 markdown 表） |

**可用因子字段**（见 `src/data.py` 的 `compute_metrics`）：
- 标准分（-100~100，推荐直接用）：`s_m`(动量) `s_v`(估值低位) `s_f`(资金) `s_rsi`(RSI择时) `s_trend`(均线趋势) `s_vol`(波动率)
- 原始指标：`r5` `r20` `r60` `r120` `pct250`(价格分位) `drawdown` `f_ratio`(量能比) `rsi14` `vol_ann`(年化波动) `box_low` `box_high` `grid_step` `ma5` `ma20` `ma60` `close`

---

## 五、本地运行

```bash
cd D:/ETF每日策略库
python src/build.py          # 全部策略
python src/build.py 3        # 只调试第3个
```

依赖：Python 3 + 标准库（无需 pip install）。数据源：同花顺热榜接口 + 新浪财经日 K 线；热榜挂了会自动回退到内置备用 ETF 池，单只标的 K 线不足会自动跳过。

---

## 六、发布到 GitHub（仓库已建好，只需双击一键脚本）

**仓库已建好**：`https://github.com/wangjun20251978/ETF-Strategy-Hub`（公开）

### 一键发布

直接**双击项目目录下的 `一键发布到GitHub.bat`**。它会：
1. 检查 gh 登录 → 提交本地改动 → git push；
2. 万一 git push 因网络/代理失败，自动改用 GitHub API 上传（脚本 `src/push_via_api.py`）；
3. 完成后打印 Pages 开启命令和在线地址。

> 若脚本提示 gh 未登录，先在终端运行一次 `gh auth login`。

### 开启 Pages（上传成功后，只需一次）

双击脚本后，在终端运行下面这行开启 Pages（Source = main / docs）：

```bash
gh api -X POST repos/wangjun20251978/ETF-Strategy-Hub/pages -f "source={""branch"":""main"",""path"":""/docs""}"
```

**在线地址**（开好后 1~2 分钟生效）：

```
https://wangjun20251978.github.io/ETF-Strategy-Hub/
```

这个链接就是发给客户的——打开是 10 个策略的导航页，点进去看每个看板，**不含闲鱼文案**（文案只在仓库外层，不发布）。

### 发给客户的东西

- **在线版**：上面的 Pages 链接（每天自动更新）。
- **离线版**：把 `docs/看板/` 里的单个 HTML 文件直接发给客户，双击就能开、断网可用。

### 以后每天

本机定时任务（每天 7:30）会自动造 5 个新策略并本地 commit；GitHub Actions 会自动重算数据。你只需**定期双击一键脚本**把本地提交推上去即可（Pages 会跟着更新）。

> 提醒：GitHub Pages 在境外，国内访问偶尔慢，离线版一定要给客户。文案里的金融管控词已在渲染出口统一脱敏。

> 提醒：GitHub Pages 在境外，国内访问偶尔慢。闲鱼卖的时候**离线版必须给**，避免买家打不开在线版要退款。金融类商品管控词（稳赚/保本/荐股/必涨/内幕等）千万别用，文案里已规避。
