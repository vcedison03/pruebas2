from io import BytesIO
import random
from difflib import SequenceMatcher
from gtts import gTTS
import streamlit as st

# Page configuration
st.set_page_config(
    page_title="Practice of English Tenses", page_icon="🎧", layout="centered"
)

st.title("🎧 Practice of English Tenses")
st.write(
    "Choose your practice mode: random sentences or reading comprehension from random texts."
)

# Banco de vocabulario por categorías (Modo Frases)
subjects = [
    "The teacher",
    "A student",
    "My friend",
    "The chef",
    "An artist",
    "Our neighbor",
    "The doctor",
    "A mechanic",
]
verbs_present = [
    ("drinks", "drinks"),
    ("prepares", "prepares"),
    ("reads", "reads"),
    ("paints", "paints"),
    ("cleans", "cleans"),
]
verbs_future = [
    ("visit", "will visit"),
    ("build", "will build"),
    ("buy", "will buy"),
    ("learn", "will learn"),
    ("organize", "will organize"),
]
objects = [
    ("coffee", "coffee"),
    ("a book", "a book"),
    ("a house", "a house"),
    ("a car", "a car"),
    ("a project", "a project"),
]
locations = [
    ("in the morning", "in the morning"),
    ("at home", "at home"),
    ("in the office", "in the office"),
    ("every day", "every day"),
]

# Banco de Textos Aleatorios (Modo Comprensión de Lectura)
reading_texts = [
    {
        "text": "Every morning, David wakes up early and drinks a cup of black coffee. Then, he drives his car to the office where he works as a software developer. In the afternoon, he usually reads a technical book to learn new programming languages.",
        "questions": [
            {"q": "What does David drink every morning?", "a": "a cup of black coffee"},
            {"q": "Where does David drive his car?", "a": "to the office"},
            {"q": "What does David read in the afternoon?", "a": "a technical book"}
        ]
    },
    {
        "text": "Sarah is a professional artist who loves painting beautiful landscapes. Next week, she will travel to Canada to buy a new house near the mountains. She hopes to find great inspiration for her next art project there.",
        "questions": [
            {"q": "What is Sarah's profession?", "a": "artist"},
            {"q": "Where will Sarah travel next week?", "a": "to canada"},
            {"q": "What does she want to buy near the mountains?", "a": "a new house"}
        ]
    }
]


def generate_all_sentences():
    """Genera un banco completo de frases posibles."""
    all_sentences = []
    for subject in subjects:
        for verb_en, _ in verbs_present:
            for obj_en, _ in objects:
                for loc_en, _ in locations:
                    all_sentences.append({"type": "sentence", "en": f"{subject} {verb_en} {obj_en} {loc_en}."})

    for subject in subjects:
        for verb_en, _ in verbs_future:
            for obj_en, _ in objects:
                all_sentences.append({"type": "sentence", "en": f"{subject} will {verb_en} {obj_en} tomorrow."})

    for subject in subjects:
        for verb_en, _ in [("working on", "is working on"), ("studying", "is studying")]:
            for obj_en, _ in objects:
                all_sentences.append({"type": "sentence", "en": f"{subject} is {verb_en} {obj_en} right now."})
                
    return all_sentences


# Initialize session state
if "configured" not in st.session_state:
    st.session_state.configured = False
    st.session_state.step = 0
    st.session_state.score = 0
    st.session_state.answered = False
    st.session_state.last_result = None
    st.session_state.test_items = []
    st.session_state.chosen_tolerance = 0.85
    st.session_state.current_text = ""

# Initial configuration screen
if not st.session_state.configured:
    st.subheader("⚙️ Test Settings")
    
    # Selector de Modo de Práctica
    practice_mode = st.radio(
        "Select practice mode:",
        ["Random Sentences (Listening)", "Reading Comprehension from Random Texts"]
    )

    strictness_mode = st.select_slider(
        "Grading strictness level:",
        options=["Very Flexible (70%)", "Flexible (80%)", "Normal (85%)", "Strict (95%)", "Exact (100%)"],
        value="Normal (85%)"
    )

    tolerance_map = {
        "Very Flexible (70%)": 0.70,
        "Flexible (80%)": 0.80,
        "Normal (85%)": 0.85,
        "Strict (95%)": 0.95,
        "Exact (100%)": 1.00
    }
    
    if st.button("🚀 Start Test"):
        st.session_state.chosen_tolerance = tolerance_map[strictness_mode]
        st.session_state.step = 0
        st.session_state.score = 0
        
        if practice_mode == "Random Sentences (Listening)":
            temp_bank = generate_all_sentences()
            total_chosen = st.slider("How many questions?", 3, min(20, len(temp_bank)), 5, key="slider_sentences")
            st.session_state.test_items = random.sample(temp_bank, total_chosen)
            st.session_state.mode = "sentences"
        else:
            # Seleccionar un texto aleatorio y sus preguntas asociadas
            selected_text_obj = random.choice(reading_texts)
            st.session_state.current_text = selected_text_obj["text"]
            st.session_state.test_items = selected_text_obj["questions"]
            st.session_state.mode = "reading"
            
        st.session_state.configured = True
        st.rerun()

else:
    total_questions = len(st.session_state.test_items)

    if st.session_state.step < total_questions:
        current_item = st.session_state.test_items[st.session_state.step]
        
        st.subheader(f"Question {st.session_state.step + 1} of {total_questions}")

        # Si el modo es lectura, mostramos el texto en pantalla
        if st.session_state.mode == "reading":
            st.info(f"📖 **Read the text:**\n\n{st.session_state.current_text}")
            st.write(f"**Question:** {current_item['q']}")
            target_answer = current_item['a']
            prompt_label = "Type your answer based on the text:"
            
            # Generar audio de la pregunta si se desea escuchar
            if "audio_bytes" not in st.session_state or st.session_state.get("audio_step") != st.session_state.step:
                tts = gTTS(text=current_item['q'], lang="en")
                fp = BytesIO()
                tts.write_to_fp(fp)
                st.session_state.audio_bytes = fp.getvalue()
                st.session_state.audio_step = st.session_state.step
            st.audio(st.session_state.audio_bytes, format="audio/mp3")

        else:  # Modo Frases (Listening)
            target_answer = current_item["en"]
            prompt_label = "Type what you heard in English:"
            
            if "audio_bytes" not in st.session_state or st.session_state.get("audio_step") != st.session_state.step:
                tts = gTTS(text=target_answer, lang="en")
                fp = BytesIO()
                tts.write_to_fp(fp)
                st.session_state.audio_bytes = fp.getvalue()
                st.session_state.audio_step = st.session_state.step
            st.audio(st.session_state.audio_bytes, format="audio/mp3")

        # Answer submission form
        with st.form(key=f"form_{st.session_state.step}"):
            user_response = st.text_input(prompt_label, key=f"input_{st.session_state.step}")
            submit_button = st.form_submit_button(label="Submit Answer")

            if submit_button:
                clean_original = target_answer.rstrip("?.!").lower()
                clean_response = user_response.rstrip("?.!").lower()

                similarity = SequenceMatcher(None, clean_response, clean_original).ratio()

                st.session_state.answered = True
                if similarity >= st.session_state.chosen_tolerance:
                    st.session_state.score += 1
                    st.session_state.last_result = "correct"
                else:
                    st.session_state.last_result = "incorrect"
                st.rerun()

        # Display feedback
        if st.session_state.get("answered", False):
            if st.session_state.last_result == "correct":
                st.success("Correct! 🎉")
            else:
                st.error("❌ Not quite right.")
                st.write(f"**Expected answer:** {target_answer}")
                st.write(f"**Your answer:** {st.session_state.get('input_' + str(st.session_state.step), '')}")

            if st.button("Next Question ➡️"):
                st.session_state.step += 1
                st.session_state.answered = False
                st.rerun()

    else:
        st.balloons()
        st.success("Test Completed!")
        st.write(f"Your final score: **{st.session_state.score} out of {total_questions}** correct answers.")

        if st.button("🔄 Configure New Test"):
            st.session_state.configured = False
            st.rerun()
