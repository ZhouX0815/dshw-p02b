 # 姓名：郑昕 学号：25210309
 # P02b：CSMAR 上市公司财务特征清洗与分析

## 数据来源

- 数据库：CSMAR（国泰安）
- 原始文件位置：`data/data_raw_zip/`
- 解压后文件位置：`data/raw/`
- 样本范围：A 股上市公司，2000 年至 2024 年
- 数据频率：年度
- 报表口径：合并报表（FS_Combas 系列为合并报表字段）

## 原始文件说明

| 文件名 | 主要内容 | 使用变量 |
|--------|---------|---------|
| CSMAR常用变量-2000-2024.zip | 股权结构、机构投资者、公司治理等常用变量 | Shrcr1(Top1), Shrhfd5(HHI5) |
| 资产负债表-2000-2010.zip | 2000-2010年合并资产负债表 | 货币资金、总资产、流动/非流动负债、短期/长期借款、权益 |
| 资产负债表-2011-2024.zip | 2011-2024年合并资产负债表 | 同上 |
| 利润表-现金流量表-2000-2010.zip | 2000-2010年利润表与现金流量表 | 营业收入、净利润、经营活动现金流 |
| 利润表-现金流量表-2011-2024.zip | 2011-2024年利润表与现金流量表 | 同上 |
| 上市公司基本信息年度表.zip | 年度公司基本信息：行业、上市日期、地区等 | 行业代码、首次上市日期 |
| 上市公司基本信息变更表2000-2024.zip | 公司名称、行业等变更记录 | 备查，本作业未直接使用 |

## 变量构造说明

| 变量 | 含义 | 计算方式 | 原始变量来源 |
|------|------|---------|------------|
| Lev | 总负债率 | total_liab / total_assets | 资产负债表 |
| SL | 流动负债率 | current_liab / total_assets | 资产负债表 |
| LL | 长期负债率 | noncurrent_liab / total_assets | 资产负债表 |
| SDR | 短债比率 | current_liab / total_liab | 资产负债表 |
| Cash | 现金比率 | cash / total_assets | 资产负债表 |
| ROA | 总资产收益率 | net_profit / total_assets | 利润表 + 资产负债表 |
| ROE | 净资产收益率 | net_profit / equity | 利润表 + 资产负债表 |
| SLoan | 短期银行借款率 | st_loan / total_assets | 资产负债表 |
| LLoan | 长期银行借款率 | lt_loan / total_assets | 资产负债表 |
| Top1 | 第一大股东持股比例 | Shrcr1 / 100 | CSMAR常用变量 |
| HHI5 | 前五大股东持股集中度 | Shrhfd5（原始已为小数） | CSMAR常用变量 |
| Size | 公司规模 | ln(total_assets) | 资产负债表 |
| Age | 上市年限 | year - list_year + 1 | 基本信息年度表 |

## 数据清洗说明

- **样本筛选规则**：保留 total_assets > 0 的观测；保留 2000 年及以后年度数据
- **重复值处理规则**：按 code-year 去重，保留 EndDate 最大的记录（年报优先）
- **缺失值处理规则**：保留缺失，不删除含缺失的行；在计算比率时若分母为0则设为 NaN
- **离群值处理规则**：对 Lev, SL, LL, SDR, Cash, ROA, ROE, SLoan, LLoan, HHI5 按年度 1%-99% 缩尾；Size、Top1、Age 不缩尾
- **合并规则**：以 code-year 为主键进行左连接（Left join），以资产负债表为基础

## 存储方式

- **基础格式**：CSV（`data/combined/csmar_firm_year_panel.csv`）
- **进阶格式**：Parquet（`data/combined/csmar_firm_year_panel.parquet`）
- **选择 Parquet 的理由**：Parquet 为列式存储，读取特定列时无需加载全表，适合大规模面板数据的分析场景；压缩比通常优于 CSV；支持 Schema 信息，避免类型推断错误

**CSV 优点**：人类可读，跨语言兼容性强，无需额外依赖库。
**CSV 不足**：大规模数据库（>1 GB）读取慢、无法按列读取、无内置类型信息。
**本作业仍用 CSV 的原因**：数据规模适中（约 7 万行），CSV 足够高效，且便于助教验证结果。

## GitHub 仓库

https://github.com/ZhouX0815/dshw-p02b

## 如何运行

1. 安装依赖：`pip install -r requirements.txt`
2. 将 7 个 CSMAR 原始压缩包放入 `data/data_raw_zip/`
3. 运行 `01_extract_raw_data.ipynb` — 解压并识别原始数据，生成变量字典
4. 运行 `02_clean_construct_variables.ipynb` — 清洗数据、构造变量、合并面板
5. 运行 `03_analysis.ipynb` — 生成表格、图形和分析结果
6. 生成报告：`jupyter nbconvert --to html 03_analysis.ipynb --output report.html`

## 最终数据说明

- `data/combined/csmar_firm_year_panel.csv`：71,787 行，5,820 家公司，2000-2024 年
- 所有比率变量均为小数形式（0.35 表示 35%）
- 缩尾变量已替换原始变量；原始变量以 `_raw` 后缀保留
