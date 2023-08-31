# 结合PAI-LLM、Hologres / AnalyticDB / Elasticsearch / FAISS、LangChain的解决方案

- 上传用户本地知识库文件，基于SGPT-125M模型生成embedding
- 生成embedding存储到向量数据库，并用于后续向量检索
- 输入用户问题，输出该问题的prompt，用于后续PAI-LLM部分生成答案
- 将产生的prompt送入EAS部署的LLM模型服务，实时获取到问题的答案
- 支持多种阿里云数据库（如AnalyticDB、Hologres、Elasticsearch）及本地FAISS向量库

## Step 1: 开发环境

### 方案一：本地conda安装

```bash
conda create --name llm_py310 python=3.10
conda activate llm_py310

git clone git@gitlab.alibaba-inc.com:pai_biz_arch/LLM_Solution.git
cd LLM_Solution

sh install.sh
```

### 方案二：Docker启动

1. 拉取已有的docker环境，防止因环境安装失败导致的不可用
```bash
docker pull registry.cn-beijing.aliyuncs.com/mybigpai/aigc_apps:1.0
```

2. 克隆项目
```bash
git clone git@gitlab.alibaba-inc.com:pai_biz_arch/LLM_Solution.git
cd LLM_Solution
```

3. 将本地项目挂载到docker并启动
```bash
sudo docker run -t -d --network host  --name llm_docker -v $(pwd):/root/LLM_Solution registry.cn-beijing.aliyuncs.com/mybigpai/aigc_apps:1.0
docker exec -it llm_docker bash
cd /root/LLM_Solution
```

## Step 2: 配置config.json

- embedding: embedding模型路径，可以用户自定义挂载，默认使用`embedding_model/SGPT-125M-weightedmean-nli-bitfit`。
- EASCfg: 配置已部署在`PAI-EAS`上LLM模型服务，可以用户自定义
- ADBCfg（可选）: AnalyticDB相关环境配置
- HOLOCfg（可选）: Hologres相关环境配置
- ElasticSearchCfg（可选）: ElasticSearch相关环境配置
- 注：如果不配置以上三种，则默认使用`FAISS`存储在本地根目录`/faiss_index`下（适合数据量很少的情况）
- create_docs: 知识库路径和相关文件配置，默认使用`/docs`下的所有文件
- query_topk: 检索返回的相关结果的数量
- prompt_template: 用户自定义的`prompt`

## Step 3: 运行main.py
1. 上传用户指定的知识库并建立索引
```bash
python main.py --config config.json --upload
```

2. 用户请求查询
```bash
python main.py --config config.json --query "用户问题"
```

## 效果展示：
```bash
python main.py --config myconfig.json --query 什么是机器学习PAI?

Output:
The answer is:  很抱歉，根据已知信息无法回答该问题。
```

```bash
python main.py --config myconfig.json --upload

Output:
Insert into AnalyticDB Success.
```

```bash
python main.py --config myconfig.json --query 什么是机器学习PAI?

Output:
The answer is:  机器学习PAI是阿里云人工智能平台，提供一站式的机器学习解决方案，包括有监督学习、无监督学习和增强学习等。它可以为用户提供从输入特征向量到目标值的映射，帮助用户解决各种机器学习问题，例如商品推荐、用户群体画像和广告精准投放等。
```
