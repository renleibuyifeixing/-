import streamlit as st
from streamlit_gsheets import GSheetsConnection
import pandas as pd
from datetime import datetime
import time
from PIL import Image

# 页面配置
st.set_page_config(page_title="五步学习法-深度监控版", layout="wide")

# 连接 Google Sheets
conn = st.connection("gsheets", type=GSheetsConnection)

st.title("🛡️ 五步学习法：闭环监督系统 (证据链版)")
st.info("💡 提示：所有标注【红笔】的动作，请在下方上传照片存证。")

# --- 侧边栏：基础信息 ---
with st.sidebar:
    st.header("👤 个人中心")
    student_id = st.text_input("学号", "NF2025001")
    student_name = st.text_input("姓名", "黄宇瑞")
    subject = st.selectbox("监控科目", ["数学", "物理", "化学", "英语", "语文"])
    lesson_name = st.text_input("课程小节", placeholder="例如：圆的切线判定")
    
    if st.button("⏱️ 开始计时"):
        st.session_state.start_time = time.time()
        st.toast("监控已开启，请专注学习")

# --- 主界面：五步执行区 ---
col1, col2 = st.columns([3, 2])

with col1:
    st.subheader("📝 动作执行与证据上传")
    
    # 第一步：预习
    with st.container(border=True):
        st.markdown("### 01 预习 (3-5min)")
        st.write("📌 **要求：** 用红笔圈画不理解的知识点。")
        step1_check = st.checkbox("已完成红笔预习", key="s1")
        img_pre = st.file_uploader("上传预习课本照片", type=['png', 'jpg', 'jpeg'], key="img1")

    # 第二步/第三步/第四步：学习与练习
    with st.container(border=True):
        st.markdown("### 02-04 学习/小测/练习")
        st.write("📌 **要求：** 笔记、错题必须有【红笔】二次订正痕迹。")
        step234_check = st.checkbox("已完成笔记与练习纠错", key="s234")
        img_work = st.file_uploader("上传笔记/练习本纠错照片", type=['png', 'jpg', 'jpeg'], key="img2")

    # 第五步：自讲
    with st.container(border=True):
        st.markdown("### 05 费曼自讲 (核心)")
        st.write("📌 **要求：** 讲清逻辑、考点及错因。")
        feynman_eval = st.select_slider("自讲熟练度", options=["无法复述", "看书能讲", "脱稿复述", "逻辑清晰", "举一反三"])
        img_mindmap = st.file_uploader("上传自讲思维导图/草稿", type=['png', 'jpg', 'jpeg'], key="img3")

with col2:
    st.subheader("📈 监控看板")
    
    # 时长监控
    if 'start_time' in st.session_state:
        elapsed = int((time.time() - st.session_state.start_time) / 60)
        st.metric("本次专注时长", f"{elapsed} 分钟")
    else:
        st.metric("本次专注时长", "待开始")

    # 证据链完整性检查
    evidence_count = sum([img_pre is not None, img_work is not None, img_mindmap is not None])
    st.write(f"证据上传进度：{evidence_count}/3")
    st.progress(evidence_count / 3)

# --- 数据锁定与提交 ---
st.divider()
if st.button("🚀 锁定进度并提交给老师", use_container_width=True):
    if not (step1_check and step234_check):
        st.error("请确保所有学习步骤已勾选完成！")
    elif evidence_count < 2:
        st.warning("证据不足！请至少上传‘预习’和‘练习’的照片。")
    else:
        # 生成数据
        new_row = pd.DataFrame([{
            "日期": datetime.now().strftime("%Y-%m-%d %H:%M"),
            "学号": student_id,
            "姓名": student_name,
            "章节": lesson_name,
            "时长(分)": elapsed if 'start_time' in st.session_state else 0,
            "掌握度": feynman_eval,
            "证据数": evidence_count,
            "状态": "审核中"
        }])
        
        # 写入云端表格
        try:
            existing_df = conn.read(worksheet="Sheet1")
            updated_df = pd.concat([existing_df, new_row], ignore_index=True)
            conn.update(worksheet="Sheet1", data=updated_df)
            st.success(f"同步成功！{student_name} 的学习闭环已生成。")
            st.balloons()
        except Exception as e:
            st.error("云端同步失败，请检查网络。")
