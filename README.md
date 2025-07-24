以下是根据您提供的微博情感分析爬虫项目整理的规范化文档，包含项目结构、配置说明、环境要求和详细使用指南：

---

# 多模型融合的微博评论情感分类微博情感分析 

![Python](https://img.shields.io/badge/Python-3.9%2B-blue)
![MySQL](https://img.shields.io/badge/MySQL-8.0.32-green)
![Dependencies](https://img.shields.io/badge/dependencies-see%20requirements.txt-orange)

> 基于 Python 的多线程微博文章+评论采集器，支持自动情感分析并存储至 MySQL 数据库

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
pip install -r requirements.txt

# requirements.txt内容：
requests==2.31.0
pandas==2.1.4
...
...
```

---

## 🚀 使用指南
运行./flask2/app.py来启动项目
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