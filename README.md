# 2025年纽约夏季城市出行结构的多源时空分析



范围说明：选择完整的2025年6—8月窗口，是因为本项目已由TLC、MTA、ERA5和Census四个独立来源建成并完成规模、关联和复现核验。2026年夏季或中国城市也可以研究，但此项目没有相同口径、已采集核验的对应数据，因此不作跨年或跨城比较。

## 核心问题

1. TLC网约车与MTA轨道/缆车入站在时间上是否同步变化？
2. ERA5天气与两类客流及网约车平均基础车费有何条件关联？
3. 结合两类客流、基础车费和ERA5天气，日度特征可分为何种探索性类型，与星期有何对应？
4. 结合Census人口与边界，区域出行强度有何差异？

每个问题依赖两个以上独立来源。研究描述关联，不能识别因果。基础车费不含税费、小费、通行费和其他附加费，不是全城消费。MTA人次不是去重人数；换乘量是ridership子集，未重复相加。

## 已采集数据与目录

两个主要来源共95个完整原始文件、2,844,083,965字节（2.844GB / 2.649GiB），TLC约51.25%、MTA约48.75%。三个月TLC逐笔Parquet和92日MTA原始CSV均已真实下载并校验。全国Census辅助数据、缓存和派生结果不参与2GiB门槛计算。MTA CSV保存HTTP标准解码后的原始正文，不改变其格式。

- `data/raw/`：代表性六份代表性原始样本及小型辅助原始数据；本机完整TLC/MTA原始文件已按脚本下载，教学提交包未附全部大文件。
- `data/cleaned/`：规范化辅助表与六份代表性清洗样本；完整处理生成的分区Parquet未全部装入提交包。
- `data/integrated/`：完整的行政区日/小时融合表（460行与11,040行），已随包提交。
- `metadata/`：原始文件字节数、SHA-256、来源时间、字段与规模摘要。
- `logs/`：真实采集日志。重试历史自保存功能启用后保留；早期重试没有完整逐次归档，不声称全量历史。
- `quality/`：逐日、逐月和融合质量结果。
- `analysis/`：描述统计、模型、聚类和稳定性结果。
- `visualization/`：7张分析图和数据关联图，PNG与SVG。
- `docs/`：指标口径、官方来源证据与方法说明。
- 正式报告、PPT及独立关联图位于本附件的上一级目录，避免混入旧稿。
- 根目录Python脚本：采集、清洗、空间连接、分析、绘图与审计。

目录名称是组织约定，根目录脚本与分层数据对应；没有将同一来源的多个文件算作多个来源。

## 环境

Python 3.11+，建议16GB内存和至少15GB空闲磁盘。处理阶段DuckDB设3GB内存预算、4线程，临时文件放`temp/`。网络下载速度和磁盘空间会影响实际耗时；未声称固定耗时。安装：

```sh
python -m pip install -r requirements.txt
```

正式运行依赖DuckDB、pandas、NumPy、Matplotlib、Shapely、pyshp、ReportLab、Pillow。Windows绘图默认使用微软雅黑；其他系统请在`visualize.py`指定本机合法可用中文字体。投影转换库不是必需项。

## 轻量复核（无需重新下载数GB）

提交包已含完整融合表、结果和元数据：

```sh
python analysis.py
python enhance_analysis.py
python visualize.py
python verify_results.py
```

这重建聚类、回归与图表；它不证明原始采集量，原始数据须通过下面完整复现及audit.py核验。代表性原始样本不是随机抽样，不能替代全样本统计。轻量核验写入quality/lightweight_verification.json，保留完整原始切片核验记录quality/result_verification.json。

## 完整复现

在项目根目录按顺序运行；同一个DuckDB文件不要并发写入。

```sh
python acquire_auxiliary.py
python download_parallel.py
python crawler.py mta
python audit.py
python processing.py auxiliary
python processing.py mta
python spatial.py
python processing.py tlc
python processing.py integrate
python analysis.py
python enhance_analysis.py
python visualize.py
python verify_results.py
```

`download_parallel.py`使用公开文件的标准HTTP范围请求、8个并发下载任务；逐块验证Content-Range、完整长度及最终SHA-256。MTA按日请求，6个并发任务，检查行数与官方计数一致；失败文件不记入完整数据。若任一天失败，可重跑`crawler.py mta`补采，已经验证的文件会跳过；最后`audit.py`必须通过。

重新采集前保留原有manifest。上游可能修订历史记录；校验不一致时应建立新版本并重跑全链。处理脚本为节约资源会跳过已完成分区；原始版本变更时请在新的项目副本中重建，不能混用旧缓存。

报告和PPT以小组修改后的结构为基础完成事实修正。文稿不由统计脚本重新生成，轻量复核仅重建统计结果与分析图。PPT保留原生表格、图表及其内嵌数据工作簿。





## 文件定位与统计口径

`metadata/layer_index.csv`是生成样本时保留的本机完整三层数据索引，列出的95份完整原始与清洗大文件并未随提交包打包。请用`metadata/submission_manifest.csv`核对实际随包文件，用`metadata/sample_inventory.csv`查看六份原始样本、六份清洗样本、行数与用途；`metadata/raw_manifest.csv`单独记录本机实际下载的95份主要源文件校验值。

原始与清洗样本各6份，每份100行，选择各月首段作为代表性结构样本，不是概率抽样。融合表为完整460行行政区日和11,040行行政区小时数据。详细关系图见上一级`03_多源数据关联图.png`，分析图的原版PNG/SVG见`visualization/`。

聚类先对两类客流取自然对数、降水取log(1+x)，再对五项特征作Z-score标准化。K=2至5各40次初始化，按惯性选各K的最佳解，再按轮廓系数选K。类型1含27个周末和1个工作日，类型2含64个工作日。研究不推断因果。
# -
