---
title: "Think Better, Solve Smarter: Personalized AI Feedback for Math Education"
categories:
  - EventRecap
tags:
  - AI
  - 후기
  - English
---

## 1. Why We Started This Project

In South Korea, high school students take several official mock exams between March and November each year.
These exams are not just practice tests—they closely resemble the actual College Scholastic Ability Test (CSAT),
which plays a pivotal role in college admissions. Because they are created by national or regional education authorities,
these mock tests serve as crucial checkpoints for students to evaluate their progress and adjust their study strategy.

However, even after these critical assessments, students are typically left with only their scores and answer sheets.
There is no structured feedback to help them understand why they got a question wrong or how they can improve.

In this Kaggle Capstone project, we set out to build an AI-powered system that analyzes students’ thinking processes,
delivers personalized feedback, and recommends similar problems that align with the reasoning they need to strengthen.

We hope this system can empower students to take charge of their own learning direction—and ultimately,
help reduce educational inequality by making reflective feedback more accessible.

---

## 2. Problem Statement
* After an exam, students’ thought processes are lost—only correct or incorrect answers remain.
* There is no structured or quantifiable way to analyze reasoning patterns or weaknesses by topic.
* When students ask, “What kind of problems should I practice next?”, both teachers and students often respond in vague terms.

---

## 3. Solution Approach

We addressed the problem through the following three-stage pipeline:

1. Input
After completing a mock exam, students record their thought process for each question via a web interface or post-exam survey.
The input includes:
* A written description of the student’s initial reasoning
* Whether the student guessed the answer
* The actual answer submitted (multiple-choice or open-ended)

2. Processing
Using the Gemini API, we evaluate the student’s reasoning against the ideal approach—not just for correctness,
but for conceptual alignment, strategy, and direction.
We also search for semantically similar problems by comparing embeddings of ideal thinking patterns using cosine similarity.

3. Output
* Personalized feedback per question, including a reasoning score and guidance
* A summary of the student’s performance and reasoning by topic
* Recommended problems that align with the student’s thinking gaps

---

## 4. Gen AI Capabilities Used
* Structured Output / JSON: Consistent, machine-readable responses from Gemini in structured JSON format
* Few-shot Prompting: Carefully designed examples guide the model to assess reasoning with appropriate generosity
* Generative AI Evaluation: Instead of binary grading, the system evaluates the direction and conceptual accuracy of the student’s thinking
* Text Embedding + Vector Search: Embedding-based semantic search is used to recommend problems that reflect similar thought processes

---

## 5. Results
* Visualized reasoning scores for each question
![Image](https://github.com/user-attachments/assets/cada221d-7428-45f2-8338-7b5fa2e992d3)
* Aggregated reasoning scores and accuracy by chapter
![Image](https://github.com/user-attachments/assets/82408b04-b178-44e0-8e29-393f7c7cbe6e)
* Highlighted weaknesses in high-priority concepts
* Semantically recommended follow-up problems based on student thinking gaps
![Image](https://github.com/user-attachments/assets/4ac8ba83-8e32-4734-805c-d11ee5ea8cf0)

---

## 6. Conclusion & Future Work

This project was not just about “solving the questions you got wrong.”
It began with a more fundamental question:
“How can we help students improve the way they think?”

In the future, we plan to:
* Deploy the system using real student data from our learning platform
* Automate feedback generation with LLM-powered scoring and recommendations
* Develop dashboards that visualize student thinking patterns and conceptual growth over time
