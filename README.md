knn-ml-app: KNN 分类器机器学习应用
 
 
 
项目概述
knn-ml-app 是一个基于 K 近邻（KNN）算法的机器学习分类应用，支持：
加载 Iris 数据集或自定义合成数据集
自动化模型训练与超参数配置（K 值可调）
模型性能评估（准确率、分类报告）
MLflow 实验跟踪（记录参数、指标、模型文件）
Docker 容器化部署与 GitHub CI/CD 自动化
适用于机器学习入门实践、KNN 算法验证及自动化部署流程学习。
目录结构
knn-ml-app/
├── app/                      # 核心应用代码
│   ├── __init__.py
│   ├── app.py                # 应用入口（模型训练与评估）
│   ├── data.py               # 数据加载与预处理
│   └── model.py              # KNN模型定义与保存
├── .env.example              # 环境变量配置示例
├── .gitignore                # Git忽略文件配置
├── .github/workflows/        # GitHub Actions CI/CD流程
│   ├── ci.yml                # 代码检查、测试、镜像构建
│   └── docker.yml            # Docker镜像推送（可选）
├── Dockerfile                # 基础Docker镜像配置
├── Dockerfile.dagshub        # DagsHub协作专用镜像（含Git）
├── requirements.txt          # 项目依赖清单
├── run.py                    # 项目启动脚本
├── pytest.ini                # Pytest测试配置
└── README.md                 # 项目说明文档（本文档）

环境准备
1. 本地环境（Python 3.11+）
步骤 1：克隆仓库
# 克隆远程仓库
git clone https://github.com/gongdayixuezha/knn-ml-app.git
cd knn-ml-app

步骤 2：创建虚拟环境并安装依赖
# 创建虚拟环境（Windows）
python -m venv venv
venv\Scripts\activate

# 创建虚拟环境（macOS/Linux）
python3 -m venv venv
source venv/bin/activate

# 安装依赖
pip install -r requirements.txt

步骤 3：配置环境变量
复制 .env.example 为 .env，并根据需求修改参数：
# 数据集配置：true=使用合成数据，false=使用Iris数据集
USE_SYNTHETIC_DATA=false

# KNN模型配置：邻居数量
KNN_NEIGHBORS=3

# MLflow配置：跟踪URI（本地路径或DagsHub地址）
MLFLOW_TRACKING_URI=mlruns

# 日志配置：true=开启详细日志
VERBOSE=true

2. Docker 环境（推荐，避免依赖冲突）
步骤 1：构建 Docker 镜像
# 构建基础镜像（本地运行）
docker build -t knn-ml-app:latest .

# 构建DagsHub协作镜像（需先配置DagsHub MLflow地址）
docker build -f Dockerfile.dagshub -t knn-ml-app:dagshub .

步骤 2：验证镜像
# 查看构建的镜像
docker images | grep knn-ml-app

核心功能与使用步骤
1. 运行项目（本地环境）
方式 1：通过启动脚本运行（推荐）
python run.py

脚本执行流程：
加载 .env 配置
加载数据集（Iris 或合成数据）
划分训练集 / 测试集（默认 8:2）
训练 KNN 模型
输出模型准确率、分类报告
保存模型到 model.pkl
记录实验到 MLflow（生成 mlruns 目录）
方式 2：通过模块直接运行
python -m app.app

方式 3：安装为本地包后运行（可选）
# 可编辑模式安装（修改代码实时生效）
pip install -e .

# 直接通过命令启动
knn-ml-app

2. 运行项目（Docker 环境）
基础运行（使用默认配置）
docker run --rm knn-ml-app:latest

自定义环境变量运行
docker run --rm \
  -e "USE_SYNTHETIC_DATA=true" \
  -e "KNN_NEIGHBORS=5" \
  -e "MLFLOW_TRACKING_URI=mlruns" \
  knn-ml-app:latest

挂载本地目录（保存模型 / MLflow 记录）
# 挂载本地 ./output 目录到容器 /app/output，保存模型和日志
docker run --rm \
  -v $(pwd)/output:/app/output \
  -e "MODEL_SAVE_PATH=/app/output/model.pkl" \
  -e "MLFLOW_TRACKING_URI=/app/output/mlruns" \
  knn-ml-app:latest

暴露 MLflow UI 端口（查看实验记录）
docker run --rm \
  -p 5000:5000 \
  -v $(pwd)/mlruns:/app/mlruns \
  knn-ml-app:latest \
  sh -c "mlflow ui --host 0.0.0.0"

然后在浏览器访问：http://localhost:5000
3. 执行单元测试
本地环境测试
# 运行所有测试用例
pytest

# 生成测试覆盖率报告（需安装 pytest-cov）
pip install pytest-cov
pytest --cov=app --cov-report=html  # 报告保存到 htmlcov/ 目录

Docker 环境测试
docker run --rm knn-ml-app:latest python -m pytest

4. 查看 MLflow 实验记录
本地环境启动 MLflow UI：
mlflow ui

浏览器访问 http://localhost:5000
查看实验记录：
参数：K 值（n_neighbors）、数据集类型（use_synthetic_data）
指标：准确率（accuracy）、精确率（precision）、召回率（recall）
artifacts：模型文件（model.pkl）、分类报告（classification_report.txt）
配置说明
1. 环境变量（.env）核心参数
参数名
类型
默认值
说明
USE_SYNTHETIC_DATA
boolean
false
是否使用合成数据集（true/false）
KNN_NEIGHBORS
integer
3
KNN 算法的邻居数量
TEST_SIZE
float
0.2
测试集占比（0.1~0.5）
RANDOM_STATE
integer
42
随机种子（保证结果可复现）
MODEL_SAVE_PATH
string
model.pkl
模型保存路径
MLFLOW_TRACKING_URI
string
mlruns
MLflow 跟踪地址（本地路径或远程 URL）
VERBOSE
boolean
true
是否输出详细日志（true/false）

2. GitHub Actions CI/CD 流程
项目已配置自动化流水线（.github/workflows/ci.yml），触发条件：
推送代码到 main/dev 分支
提交 Pull Request 到 main/dev 分支
流水线执行步骤：
代码检查：使用 flake8 检查代码规范
依赖安装：安装 requirements.txt 依赖
单元测试：运行 pytest 并生成覆盖率报告
Docker 构建：构建 Docker 镜像并验证运行
MLflow 验证：检查实验记录是否正常生成
如需推送镜像到容器仓库（如 GitHub Container Registry），可参考 .github/workflows/docker.yml 配置，需添加仓库密钥（DOCKERHUB_USERNAME/DOCKERHUB_TOKEN）。
分支管理与协作
1. 分支说明
分支类型
命名格式
作用
main
-
生产环境分支，保持稳定可部署
dev
-
开发分支，集成已完成的功能
feature/*
feature/功能名
功能开发分支（如 feature/add-log）
hotfix/*
hotfix/问题名
紧急修复分支（如 hotfix/fix-model）
release/*
release/版本号
版本发布分支（如 release/v1.0.0）

2. 协作流程
开发新功能
从 dev 分支创建 feature 分支：
 
开发完成后提交代码：
 
在 GitHub 创建 Pull Request（基础分支：dev）
审核通过后合并，删除远程 feature 分支
本地同步 dev 分支并清理：
 
紧急修复
从 main 分支创建 hotfix 分支：
 
修复完成后提交并推送，创建 PR 到 main
合并后同步到 dev 分支：
 
常见问题解决
1. 推送代码时提示 “fetch first”
错误：! [rejected] dev -> dev (fetch first)
原因：远程分支有本地未同步的更新
解决：
 
2. Docker 构建缓慢
解决：
确保 Dockerfile 中依赖安装步骤在代码复制之前（利用 Docker 缓存）
使用国内镜像源（如在 Dockerfile 中添加：RUN pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple）
3. MLflow UI 无法访问
检查点：
确认端口映射正确（-p 5000:5000）
确认 MLflow UI 绑定地址为 0.0.0.0（而非 localhost）
查看容器日志：docker logs 容器ID
贡献指南
Fork 本仓库到个人账号
从 dev 分支创建 feature 分支开发功能
提交代码时遵循语义化提交规范（如 feat: 新增XX功能、fix: 修复XX问题）
确保代码通过 flake8 检查和 pytest 测试
创建 Pull Request 到本仓库的 dev 分支，等待审核
许可证
MIT License（如需添加许可证，可创建 LICENSE 文件并在此引用）
