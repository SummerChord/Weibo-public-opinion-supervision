

# 多模型融合的微博评论情感分类  

![Python](https://img.shields.io/badge/Python-3.9%2B-blue)
![MySQL](https://img.shields.io/badge/MySQL-8.0.32-green)
![Dependencies](https://img.shields.io/badge/dependencies-see%20requirements.txt-orange)

> 基于 Python 的多线程微博文章+评论采集器，支持自动情感分析并存储至 MySQL 数据库

> 项目附加文件Git地址： https://github.com/SummerChord/Sentiment-Classification-Analysis-of-Weibo-comments

---

## 项目结构

```bash
flask2/
├── BERT/
│   ├── bert-base-chinese/
│   ├── models/
│   │   ├── TextRCNN.py
│   │   ├── TextRNN_Att.py
│   │   └── Transformer.py
│   ├── MyBERT10/
│   ├── tensorboard_logs/
│   └── THUCNews/
│       ├── data/
│       │   └── 3fenlei_2/
│       │       ├── class.txt
│       │       ├── dev.txt
│       │       ├── test.txt
│       │       ├── train.txt
│       │       ├── vocab.pkl
│       │       └── embedding_3fenlei_2.npz
│       ├── log/
│       │   ├── TextRCNN/
│       │   ├── TextRNN_Att/
│       │   └── Transformer/
│       └── saved_dict/
│           └── 3fenlei_2/
│               ├── TextRCNN.ckpt
│               ├── TextRNN_Att.ckpt
│               └── Transformer.ckpt
├── spiders/
│   ├── data/
│   │   ├── articleCategory.csv
│   │   ├── articleComments_2025-07-22_17-25-15.csv
│   │   └── articleContent_2025-07-22_17-25-15.csv
│   ├── clearData.py
│   ├── main.py
│   ├── spiderArticleCategory.py
│   ├── spiderComments.py
│   └── spiderContent.py
├── static/
├── templates/
│   └── 404.html
├── utils/
│   ├── databaseManage.py
│   ├── getEchartsData.py
│   ├── stopWords.txt
│   └── wordCloudPicture.py
├── views/
│   ├── page/
│   │   ├── templates/
│   │   │   ├── articleChar.html
│   │   │   ├── articleData.html
│   │   │   ├── articleData_temp.html
│   │   │   ├── base_page.html
│   │   │   ├── deleteData.html
│   │   │   ├── index.html
│   │   │   └── spiderData.html
│   │   └── page.py
├── analysis_comments.py
├── data.csv
├── run.py
├── train3.py
├── train_eval.py
├── utils.py
└── requirements.txt
```

---

## ⚙️ 系统配置

### 1. 环境变量 (.env)
```ini
# MySQL 数据库配置
DB_HOST = localhost
DB_USER = root
DB_PWD = your_password
DB_NAME = weibo_sentiment

# 微博Cookie配置
COOKIE = SUB=your_weibo_cookie; SSOLoginState=xxxx;
# 获取Cookie配置具体方法：
1. 登录微博网页版（https://weibo.com）
2. 打开浏览器开发者工具（F12）→ Network → 刷新页面
3. 找一个以 .weibo.com 开头的请求 → 查看 Request Headers → 找到 Cookie 字段
4. 复制整个 Cookie 字符串，粘贴到你的配置中即可 
```


---

## 🐳 环境要求

### 硬件配置
| 组件 | 最低配置 | 推荐配置 |
|------|----------|----------|
| **CPU** | 4核 3.0 GHz | 8核 3.0 GHz |
| **内存** | 8 GB | 16 GB |
| **显卡** | NVIDIA RTX 2070 | NVIDIA RTX 3060+ |
| **存储** | 100 GB SSD | 500 GB NVMe |

### 软件依赖
```bash
# 安装Python依赖
pip install -r ./requirements.txt

# requirements.txt内容：
requests==2.31.0
pandas==2.1.4
...
...
```

---
## 🍩 模型概况
### 1. 数据集概况
#### 数据集组成
训练采用的数据集由数据集1与数据集2合并、处理后构成。均为公共数据集。


数据集1来源： https://aistudio.baidu.com/datasetdetail/70546


数据集2来源： https://aistudio.baidu.com/datasetdetail/70542

#### 数据字段
包含两个核心字段

**text** : 用户评价文本，内容为中文自然语言描述。

**label** : 情感标签，取值为0、1、2，分别代表负面评价、中性评价、正面评价。

|text|label|
|----|----|
|写的一般，完全没有价值，不值得买，买了也不值得看。|0|
|我下了订单，但是此书没有送给我，其他书却送到了，真是奇怪。也没说是什么原因|1|
|好精致的！特别适合妹妹！|2|

#### 数据分布

| 数据集 | 总样本数/条 | 负面样本数/条 | 中性样本数/条 | 正面样本数/条 | 缺失样本数/条 | 相似样本数/条 | 含英文样本数/条 | 文本平均长度/字符 | 文本长度范围/字符 |
|---------|---------|---------|---------|---------|---------|---------|---------|---------|---------|
|训练集|42595|14663|12922|15010|0|1783|5167|83.8|[1,7369]|
|验证集|10293|3847|2667|2779|0|577|1691|94.0|[1,2876]|
|测试集|8023|3080|2198|2925|0|259|1251|92.1|[1,1985]|

### 模型效果
#### Negative
| 模型 | Precision | Recall | f1-score|
|---|---|---|---|
|Transformer|0.8815|*0.9552*|0.9169|
|Att-BLSTM|*0.9544*|0.9172|0.9354|
|TextRCNN|0.9256|0.8791|0.8816|
|微调BERT|0.9480|0.9550|*0.9615*|

#### Neutral
| 模型 | Precision | Recall | f1-score|
|---|---|---|---|
|Transformer|0.9068|0.8921|0.8994|
|Att-BLSTM|0.8835|0.9172|0.8780|
|TextRCNN|0.8842|0.8791|0.8816|
|微调BERT|*0.9320*|*0.9150*|*0.9234*|

#### Positive
| 模型 | Precision | Recall | f1-score|
|---|---|---|---|
|Transformer|0.9047|0.9218|0.8913|
|Att-BLSTM|0.8965|*0.9278*|0.8814|
|TextRCNN|0.8897|0.8626|0.8795|
|微调BERT|*0.9250*|0.9108|*0.9225*|

---

## 🚀 使用指南
### 运行./flask2/app.py来启动项目
### 1. 微博评论采集
```python
from spiders.spiderComments import start_comment_spider

# 基于文章数据采集评论
start_comment_spider(
    max_pages=50,  # 每篇文章最多50页评论
    batch_size=5   # 并发处理5篇文章
)
```

**输出示例 (output/articleComments.csv):**
| articleId | likes_counts | content | authorGender | authorAddress |
|-----------|--------------|---------|--------------|---------------|
| 49283... | 342 | 这部国产动画质量超预期！ | m | 广东 |
| 49283... | 89 | 剧情有点幼稚，不推荐 | f | 上海 |
### 2. 模型使用
```
# 训练并测试：

# TextRNN_Att
python  BERT/run.py --model TextRNN_Att
可以在TextRNN_Att.py中修改数据集路径，调整参数

# TextRCNN
python  BERT/run.py --model TextRCNN
可以在TextRCNN.py中修改数据集路径，调整参数

# Transformer
python  BERT/run.py --model Transformer
可以在Transformer.py中修改数据集路径，调整参数

# Bert
python BERT/train3.py 
词向量下载链接：https://github.com/Embedding/Chinese-Word-Vectors
词向量在./flask2/util.py的pretrain_dir中使用，构建好的词嵌入在./flask2/run.py调用。
```
---

## 🔧 高级配置

### 反爬策略配置 (utils/globalVariable.py)
```python
# 请求头配置
headers = {
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) ...",
    "Cookie": os.getenv("COOKIE"),
    "X-Requested-With": "XMLHttpRequest"
}

# 代理配置
proxies = {
    "http": "http://user:pass@proxy_ip:port",
    "https": "https://user:pass@proxy_ip:port"
}

# 请求间隔控制
REQUEST_DELAY = random.uniform(0.5, 2.0)  # 随机延时0.5-2秒
```



---

## 📊 数据存储设计

### 在命令行执行 weiboarticles.sql 创建数据库
#### weiboarticles.sql可在项目GitHub上下载

### 文章表 (articles)
```sql
CREATE TABLE articles (
    id BIGINT PRIMARY KEY,
    likeNum INT,
    commentsLen INT,
    reposts_count INT,
    region VARCHAR(255),
    content TEXT,
    created_at DATE,
    type VARCHAR(50),
    detailUrl VARCHAR(255),
    authorName VARCHAR(255),
    authorDetail VARCHAR(255)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

### 评论表 (comments)
```sql
CREATE TABLE comments (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    articleId BIGINT,
    created_at DATETIME,
    likes_counts INT,
    region VARCHAR(255),
    content TEXT,
    authorName VARCHAR(255),
    authorGender ENUM('m','f'),
    authorAddress VARCHAR(255),
    sentiment TINYINT COMMENT '0:负面 1:中性 2:正面',
    FOREIGN KEY (articleId) REFERENCES articles(id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

---

## ⚠️ 注意事项

1. **Cookie有效期**
   - 微博Cookie通常有效期为2-3天，过期后需重新获取
   - 可在[微博开发者平台](https://open.weibo.com/)申请长期Token

2. **请求频率限制**
   - 单IP限制：≤ 50请求/分钟
   - 建议配置代理池应对IP封锁

3. **数据清洗规则**
   - 移除HTML标签
   - 过滤表情符号
   - 替换特殊字符
   - 删除广告内容


---

## 📜 许可证
[MIT License](https://opensource.org/licenses/MIT) © 2025 WeiboSentimentSpider Project
