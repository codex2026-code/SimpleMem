# BERTScore 离线运行指南

当测试里调用 `bert_score.score(...)` 时，如果本机没有缓存所需模型，`transformers` 会尝试从 Hugging Face 下载。网络不可达时会报错：

`We couldn't connect to 'https://huggingface.co' to load the files...`

## 原因说明

- 当前代码在未指定 `model_type` 时使用 `lang='en'`，`bert-score` 会自动解析并加载英文默认模型（通常来自 Hugging Face Hub）。
- 首次加载该模型需要联网下载权重、tokenizer、配置文件；如果没有网络或被防火墙拦截，就会失败。
- 失败后本项目会回退为 `bert_f1=0.0`，因此你会看到其它指标正常、BERTScore 为 0 的情况。

## 可靠离线修复方案（推荐）

核心思路：**在可联网机器提前下载模型，测试机只用本地路径加载**。

### 1) 在可联网机器准备模型目录

```bash
python -m pip install -U "huggingface_hub[cli]"
huggingface-cli download roberta-large --local-dir ./offline_models/roberta-large
```

### 2) 将目录拷贝到测试机

例如拷贝到：

`/opt/models/roberta-large`

### 3) 在测试机设置环境变量

```bash
export BERTSCORE_MODEL_TYPE=/opt/models/roberta-large
# 可选：固定层数（不同模型可按 bert-score 推荐值配置）
# export BERTSCORE_NUM_LAYERS=17
export TRANSFORMERS_OFFLINE=1
export HF_HUB_OFFLINE=1
```

### 4) 运行测试

```bash
python test_locomo10.py
```

## 本仓库已支持的离线参数

`test_locomo10.py` 与 `test_ref/utils.py` 中的 `calculate_bert_scores` 现在支持：

- `BERTSCORE_MODEL_TYPE`：本地模型目录（推荐）。
- `BERTSCORE_NUM_LAYERS`：可选，覆盖默认层数。

若未设置 `BERTSCORE_MODEL_TYPE`，则保持原行为（`lang='en'`，可能触发在线下载）。
