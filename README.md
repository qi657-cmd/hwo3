运行命令：streamlit run app.py
项目结构：
hw03/
├── app.py                # Streamlit主程序
├── requirements.txt      # 依赖清单
├── README.md             # 项目说明文档
├── known_faces/          # 已知人脸库（可选，用于识别）
│   ├── person1.jpg
│   └── person2.jpg
└── tests/                # 测试文件（可选）
    └── test_face.py
功能说明：
1.  **人脸检测**：使用`face_recognition`库定位图片中的人脸位置
2.  **特征编码**：自动提取每张人脸的128维特征向量
3.  **人脸识别（可选）**：与`known_faces`目录下的已知人脸进行比对识别
4.  **Web界面**：基于Streamlit实现图片上传、结果可视化

## 运行说明
### 1. 环境准备
```bash
pip install -r requirements.txt
准备人脸库：
在 known_faces 目录下放置图片，文件名格式为 姓名.jpg ，系统会自动加载并用于识别。
运行与访问：
运行成功后，访问本地URL： http://localhost:8501
