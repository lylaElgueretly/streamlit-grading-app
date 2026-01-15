import streamlit as st
import pandas as pd
import matplotlib.pyplot as plt

# -----------------------------
# Macro & Micro Skills Banks
# -----------------------------
macro_skills = {
    'Reading': ['R1 Literal', 'R2 Inference', 'R3 Vocabulary', 'R4 Sequencing', 'R5 Critical Eval', 'R6 Language Analysis'],
    'Writing': ['W1 Content', 'W2 Organisation', 'W3 Grammar', 'W4 Vocabulary', 'W5 Audience', 'W6 Creativity']
}

micro_skills = {
    'R1 Literal': ['Misread fact', 'Missed detail', 'Wrong number/figure'],
    'R2 Inference': ['Unsupported inference', 'Wrong conclusion', 'Misinterpretation'],
    'R3 Vocabulary': ['Wrong meaning', 'Figurative misunderstanding', 'Term misspelling'],
    'R4 Sequencing': ['Wrong order', 'Skipped step', 'Misunderstood process'],
    'R5 Critical Eval': ['Misjudged purpose', 'Did not identify bias', 'Did not compare info'],
    'R6 Language Analysis': ['Misinterpreted tone', 'Misread style', 'Ignored literary device'],
    'W1 Content': ['Fact incorrect', 'Off-topic', 'Missing detail'],
    'W2 Organisation': ['Poor paragraphing', 'Lack of linking', 'Illogical sequence'],
    'W3 Grammar': ['Punctuation error', 'Apostrophe misuse', 'Quotes misuse', 'Capitalisation error',
                   'Sentence fragment', 'Run-on sentence', 'Subject-verb agreement', 'Complex sentence error'],
    'W4 Vocabulary': ['Word choice inappropriate', 'Repetition', 'Technical term missing'],
    'W5 Audience': ['Tone too informal', 'Style inappropriate', 'Lack of formality'],
    'W6 Creativity': ['Ideas underdeveloped', 'Lack of interesting facts', 'Poor expression']
}

# -----------------------------
# 1. Exam Setup Section
# -----------------------------
st.title("Exam Setup")

num_questions = st.number_input("Number of Reading Comprehension Questions", min_value=1, max_value=50, value=5)

question_settings = []
for i in range(1, num_questions + 1):
    with st.expander(f"Question {i} Setup"):
        macro_choice = st.radio(f"Select Macro Skill for Q{i}", macro_skills['Reading'], key=f"macro_{i}")
        st.write("Select possible Micro Skills for Q{i} (click multiple if applicable):")
        micro_selected = []
        for tag in micro_skills[macro_choice]:
            if st.checkbox(tag, key=f"tag_{i}_{tag}"):
                micro_selected.append(tag)
        question_settings.append({
            'number': i,
            'macro': macro_choice,
            'micro_tags': micro_selected
        })

# Writing Rubric Setup
st.subheader("Writing Rubric Setup")
rubric_categories = []
st.write("Select Rubric Categories for Writing (click to include):")
for cat in macro_skills['Writing']:
    if st.checkbox(cat):
        rubric_categories.append(cat)

rubric_points = {}
for cat in rubric_categories:
    rubric_points[cat] = st.number_input(f"Max marks for {cat}", min_value=1, max_value=20, value=5, key=f"marks_{cat}")

# Store setup in session
st.session_state['question_settings'] = question_settings
st.session_state['rubric_points'] = rubric_points

st.success("Exam setup saved. Proceed to Grading Form.")

# -----------------------------
# 2. Grading Form Section
# -----------------------------
st.title("Grading Form")
num_students = st.number_input("Number of Students", min_value=1, max_value=50, value=3)

student_data = []

for s in range(1, num_students + 1):
    with st.expander(f"Student {s} Details"):
        student_name = st.text_input(f"Student Name {s}", key=f"name_{s}")
        student_class = st.text_input(f"Class {s}", key=f"class_{s}")

        # Reading marks and tags
        reading_marks = {}
        reading_tags_selected = {}
        for q in st.session_state['question_settings']:
            with st.expander(f"Q{q['number']} - {q['macro']}"):
                mark = st.number_input(f"Marks", min_value=0, max_value=5, value=0, key=f"mark_{s}_{q['number']}")
                st.write("Select Mistakes (click all that apply):")
                selected_tags = []
                for tag in micro_skills[q['macro']]:
                    if st.checkbox(tag, key=f"student_{s}_q{q['number']}_{tag}"):
                        selected_tags.append(tag)
                reading_marks[q['number']] = mark
                reading_tags_selected[q['number']] = selected_tags

        # Writing marks and tags
        writing_marks = {}
        writing_tags_selected = {}
        for cat in st.session_state['rubric_points']:
            with st.expander(f"{cat} (Max: {st.session_state['rubric_points'][cat]})"):
                mark = st.number_input(f"Marks for {cat}", min_value=0, max_value=st.session_state['rubric_points'][cat],
                                       key=f"w_mark_{s}_{cat}")
                st.write("Select Mistakes (click all that apply):")
                selected_tags = []
                for tag in micro_skills[cat]:
                    if st.checkbox(tag, key=f"w_student_{s}_{cat}_{tag}"):
                        selected_tags.append(tag)
                writing_marks[cat] = mark
                writing_tags_selected[cat] = selected_tags

        student_data.append({
            'name': student_name,
            'class': student_class,
            'reading_marks': reading_marks,
            'reading_tags': reading_tags_selected,
            'writing_marks': writing_marks,
            'writing_tags': writing_tags_selected
        })

st.session_state['student_data'] = student_data

# -----------------------------
# 3. Report & Download Section
# -----------------------------
st.title("Reports & Download Options")
report_type = st.radio("Choose report type", ['Summary', 'Detailed per student', 'Class Report'])

if st.button("Generate Report"):
    report_rows = []
    for student in st.session_state['student_data']:
        total_reading = sum(student['reading_marks'].values())
        total_writing = sum(student['writing_marks'].values())
        reading_mistakes = ', '.join([tag for q in student['reading_tags'].values() for tag in q])
        writing_mistakes = ', '.join([tag for cat in student['writing_tags'].values() for tag in cat])
        report_rows.append({
            'Student': student['name'],
            'Class': student['class'],
            'Reading Total': total_reading,
            'Writing Total': total_writing,
            'Reading Mistakes': reading_mistakes,
            'Writing Mistakes': writing_mistakes
        })

    df_report = pd.DataFrame(report_rows)

    if report_type == 'Summary':
        st.dataframe(df_report[['Student','Class','Reading Total','Writing Total']])
    elif report_type == 'Detailed per student':
        for idx, row in df_report.iterrows():
            st.subheader(f"Student: {row['Student']}")
            st.write(f"Class: {row['Class']}")
            st.write(f"Reading Total: {row['Reading Total']}")
            st.write(f"Writing Total: {row['Writing Total']}")
            st.write(f"Reading Mistakes: {row['Reading Mistakes']}")
            st.write(f"Writing Mistakes: {row['Writing Mistakes']}")
    elif report_type == 'Class Report':
        st.write("Average Reading:", df_report['Reading Total'].mean())
        st.write("Average Writing:", df_report['Writing Total'].mean())

    # Download CSV for all students
    csv = df_report.to_csv(index=False).encode('utf-8')
    st.download_button(
        label="Download CSV of All Students",
        data=csv,
        file_name="grades_all_students.csv",
        mime='text/csv'
    )

# -----------------------------
# 4. Skill Mastery Dashboard
# -----------------------------
st.title("Skill Mastery Dashboard")

# Aggregate marks per macro skill
reading_macro_totals = {macro: 0 for macro in macro_skills['Reading']}
writing_macro_totals = {macro: 0 for macro in macro_skills['Writing']}

for student in student_data:
    for q in st.session_state['question_settings']:
        reading_macro_totals[q['macro']] += student['reading_marks'][q['number']]
    for cat in st.session_state['rubric_points']:
        writing_macro_totals[cat] += student['writing_marks'][cat]

# Plot Reading Skill Mastery
st.subheader("Reading Skill Mastery")
plt.figure(figsize=(10,4))
plt.bar(reading_macro_totals.keys(), reading_macro_totals.values(), color='skyblue')
plt.ylabel("Total Marks")
plt.xticks(rotation=45)
st.pyplot(plt.gcf())

# Plot Writing Skill Mastery
st.subheader("Writing Skill Mastery")
plt.figure(figsize=(10,4))
plt.bar(writing_macro_totals.keys(), writing_macro_totals.values(), color='salmon')
plt.ylabel("Total Marks")
plt.xticks(rotation=45)
st.pyplot(plt.gcf())
