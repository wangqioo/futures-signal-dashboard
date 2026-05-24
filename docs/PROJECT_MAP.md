# 项目地图

这份地图用于从头到尾理解当前项目，不替代 README。README 面向使用和运行，这里面向维护和整理。

## 一句话主线

行情数据进入 `server.py`，经过内置公式和插件公式计算，组装成前端可画的 `indicator_series`，再由 `dashboard/index.html` 画出主图、副图、信号和趋势。

## 运行主路径

1. 用户打开 `/`。
2. Flask 从 `dashboard/index.html` 返回单页前端。
3. 前端 `loadData(mode)` 请求 `/api/data?symbol=...&period=...`。
4. `server.py:get_data()` 根据周期选择分钟线、日线或周线。
5. `indicators.py` 计算主图信号和波段王副图。
6. `data_fetcher.py` 补充支撑压力位和资金流向。
7. 后端返回 `indicator_series`、`signals`、`meta`、`levels`、`history`。
8. 前端 `renderAll()` 或 `updateBars()` 绘制图表和侧边栏。

## 信号主路径

1. `server.py` 启动时创建后台扫描线程 `_scanner_loop()`。
2. 扫描线程按当前触发周期判断是否到 K 线收盘时刻。
3. `_do_scan()` 遍历默认品种，调用 `get_data()` 计算最新信号。
4. `_is_new_signal()` 写入 `data/signals.db` 做跨重启去重。
5. 新信号进入 `_pending_signals`。
6. 前端每 3 秒轮询 `/api/signals/pending?since=...`。
7. 前端用弹窗、桌面通知和未读队列展示信号。

`scheduler.py` 是可选 sidecar：它复用 `server.py:get_data()`，通过 HTTP POST `/api/signals/push` 把信号推回主服务。默认运行不需要它。

## 公式主路径

内置公式在 `indicators.py`：

- `calc_main_signals()`：计算 M1-M5、支撑线、QRG、破浪、空仓。
- `calc_bsd_wang()`：计算 K/D 和波段王多空标记。
- `get_latest_signals()`：把最新一根 K 线转成 `signals` 和 `meta`。

插件公式在 `indicators_pkg/`：

- 每个插件提供 `META` 和 `compute(df)`。
- `indicators_pkg/__init__.py` 负责热加载。
- `tdx_parser/` 可以把 TDX/慧赢公式解析成插件源码。

## 主流程文件

整理主流程时，优先改 `server.py`、`dashboard/index.html`、`indicators.py`、`indicators_pkg/` 和 `tdx_parser/`。早期棕榈油单品脚本已经删除，避免它们继续干扰多品种看板逻辑。

## 当前整理优先级

1. 抽出行情数据模块：`server.py` 的数据获取、缓存、周线聚合可以形成更深的模块。
2. 抽出信号扫描模块：`server.py` 和 `scheduler.py` 现在重复了交易时段、K 线收盘、去重逻辑。
3. 扩充公式回归测试：继续增加真实行情切片，验证 `破浪`、`空仓`、`K/D` 输出。
