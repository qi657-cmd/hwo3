import streamlit as st
import face_recognition
import cv2
import numpy as np
from PIL import Image
import os

# --------------------------
# 配置项
# --------------------------
KNOWN_FACES_DIR = "known_faces"  # 已知人脸库路径
TOLERANCE = 0.6                  # 识别容忍度（越小越严格）

# --------------------------
# 加载已知人脸（可选）
# --------------------------
@st.cache_resource
def load_known_faces():
    known_encodings = []
    known_names = []
    
    if not os.path.exists(KNOWN_FACES_DIR):
        os.makedirs(KNOWN_FACES_DIR)
        return known_encodings, known_names
    
    for filename in os.listdir(KNOWN_FACES_DIR):
        if filename.lower().endswith(('.png', '.jpg', '.jpeg')):
            path = os.path.join(KNOWN_FACES_DIR, filename)
            name = os.path.splitext(filename)[0]
            image = face_recognition.load_image_file(path)
            encodings = face_recognition.face_encodings(image)
            if encodings:
                known_encodings.append(encodings[0])
                known_names.append(name)
    return known_encodings, known_names

known_encodings, known_names = load_known_faces()

# --------------------------
# Streamlit界面
# --------------------------
st.title("人脸检测与识别工具")
st.sidebar.header("配置")
uploaded_file = st.file_uploader("上传图片", type=["jpg", "jpeg", "png"])

if uploaded_file is not None:
    # 读取图片
    image = Image.open(uploaded_file)
    img_array = np.array(image)
    rgb_img = cv2.cvtColor(img_array, cv2.COLOR_BGR2RGB)
    
    # 人脸检测
    face_locations = face_recognition.face_locations(rgb_img)
    face_encodings = face_recognition.face_encodings(rgb_img, face_locations)
    
    # 绘制人脸框和识别结果
    for (top, right, bottom, left), face_encoding in zip(face_locations, face_encodings):
        # 检测框
        cv2.rectangle(img_array, (left, top), (right, bottom), (0, 255, 0), 2)
        
        # 识别（如果有人脸库）
        if known_encodings:
            matches = face_recognition.compare_faces(known_encodings, face_encoding, tolerance=TOLERANCE)
            name = "未知"
            if True in matches:
                first_match_index = matches.index(True)
                name = known_names[first_match_index]
            
            # 绘制标签
            cv2.rectangle(img_array, (left, bottom - 35), (right, bottom), (0, 255, 0), cv2.FILLED)
            font = cv2.FONT_HERSHEY_DUPLEX
            cv2.putText(img_array, name, (left + 6, bottom - 6), font, 1.0, (255, 255, 255), 1)
    
    # 显示结果
    st.image(img_array, caption="检测结果", use_column_width=True)
    st.success(f"检测到 {len(face_locations)} 张人脸")
else:
    st.info("请上传一张图片进行人脸检测")

# 示例图
st.sidebar.subheader("示例图")
if st.sidebar.button("使用示例图"):
    example_img_path = "example.jpg"  # 你可以准备一张示例图放在同目录
    if os.path.exists(example_img_path):
        image = Image.open(example_img_path)
        img_array = np.array(image)
        rgb_img = cv2.cvtColor(img_array, cv2.COLOR_BGR2RGB)
        face_locations = face_recognition.face_locations(rgb_img)
        for (top, right, bottom, left) in face_locations:
            cv2.rectangle(img_array, (left, top), (right, bottom), (0, 255, 0), 2)
        st.image(img_array, caption="示例图检测结果", use_column_width=True)
        st.success(f"检测到 {len(face_locations)} 张人脸")
    else:
        st.warning("请在项目目录下放置 example.jpg 作为示例图")
