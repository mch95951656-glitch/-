# -import streamlit as st
import google.generativeai as genai

# -----------------------------
# 페이지 설정
# -----------------------------
st.set_page_config(
    page_title="AI 연애상담 챗봇",
    page_icon="💌",
    layout="centered"
)

st.title("💌 AI 연애상담 챗봇")
st.caption("Gemini 2.5 Flash Lite 기반")

# -----------------------------
# API 키 불러오기
# -----------------------------
try:
    api_key = st.secrets["GEMINI_API_KEY"]
    genai.configure(api_key=api_key)

except Exception:
    st.error("API 키를 불러오지 못했습니다.")
    st.stop()

# -----------------------------
# 모델 설정
# -----------------------------
model = genai.GenerativeModel(
    model_name="gemini-2.5-flash-lite",
    system_instruction="""
    너는 공감 능력이 좋은 연애상담 AI다.

    규칙:
    - 친근하고 자연스럽게 대답한다.
    - 사용자를 존중한다.
    - 강압적이거나 공격적인 표현은 피한다.
    - 너무 짧게 답하지 않는다.
    - 상황에 따라 현실적인 조언을 제공한다.
    - 위험하거나 유해한 행동은 권장하지 않는다.
    """
)

# -----------------------------
# 채팅 기록 초기화
# -----------------------------
if "messages" not in st.session_state:
    st.session_state.messages = []

# 이전 채팅 표시
for msg in st.session_state.messages:
    with st.chat_message(msg["role"]):
        st.markdown(msg["content"])

# -----------------------------
# 사용자 입력
# -----------------------------
user_input = st.chat_input("연애 고민을 입력해보세요...")

if user_input:

    # 사용자 메시지 저장
    st.session_state.messages.append({
        "role": "user",
        "content": user_input
    })

    # 사용자 메시지 출력
    with st.chat_message("user"):
        st.markdown(user_input)

    try:
        # Gemini용 히스토리 변환
        history = []

        for msg in st.session_state.messages[:-1]:
            role = "user" if msg["role"] == "user" else "model"

            history.append({
                "role": role,
                "parts": [msg["content"]]
            })

        # 채팅 세션 생성
        chat = model.start_chat(history=history)

        # 응답 생성
        response = chat.send_message(user_input)

        bot_reply = response.text

    except Exception as e:
        bot_reply = f"오류가 발생했습니다.\n\n{str(e)}"

    # AI 응답 저장
    st.session_state.messages.append({
        "role": "assistant",
        "content": bot_reply
    })

    # AI 응답 출력
    with st.chat_message("assistant"):
        st.markdown(bot_reply)

# -----------------------------
# 사이드바
# -----------------------------
with st.sidebar:
    st.header("⚙️ 설정")

    if st.button("채팅 기록 초기화"):
        st.session_state.messages = []
        st.rerun()

    st.markdown("---")
    st.markdown(
        """
        ### 💡 사용 예시
        - 썸이 식은 것 같아
        - 고백해도 될까?
        - 답장이 느린 이유가 뭘까?
        - 헤어진 뒤 연락해도 될까?
        """
    )
