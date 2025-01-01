---
title: The Journey of a Backend Engineer Building an LLM Application with Prompt Engineering
categories:
  - LLM
tags:
  - 회고
---

#### The Beginning of the Story and Simplifying the Front-End with No-Code Tools


Ddangwoomath is an innovative online academy geared toward students preparing for college entrance exams. In this digital landscape, students must engage in independent learning, including the uploading of study materials. My initial responsibility was to develop a Learning Management System (LMS) tailored to different membership tiers, enabling students to access lectures, submit homework, and receive evaluations. As someone with a strong backend development background, I explored frontend solutions and ultimately landed on the no-code platform [bubble.io](https://bubble.io/). Given that the system wouldn’t be overwhelmed with high traffic or require intricate UI/UX, I decided Bubble was the optimal choice. While I set about mastering Bubble, I enlisted a proficient user of the platform to handle the front-end development, allowing me to concentrate on creating the backend workflows for homework reviews.

#### Setting Goals

At Ddangwoomath, homework goes beyond solving traditional problems. We embrace the philosophy of “Study as if you are teaching someone else.” Each student is tasked with recording a video of themselves explaining a concept, effectively simulating a teaching experience. Until recently, these submissions were manually reviewed, which was time-consuming and often inconsistent. However, the rise of AI-powered video summarization tools opened the door for automation, allowing us to deliver more value to our students. For example, a solid foundation in mathematics is crucial for tackling advanced topics. Rather than merely assessing submission completion, AI can evaluate students' comprehension and provide personalized feedback. Over time, this also benefits instructors by pinpointing collective student struggles and encouraging more targeted explanations in future lectures.

#### Understanding Your Input

Initially, I planned to evaluate homework accuracy by comparing it to textbook content or ideal explanations. However, this one-shot approach proved ineffective due to a fundamental misunderstanding of the input. Students' explanations are not polished or scripted; they are natural and often include hesitations or repetitions. Furthermore, Ddangwoomath primarily helps students ranked in the lower 30%, whose explanations differ significantly from those of top-performing students. Comparing such explanations with textbook content resulted in even diligent students receiving failing evaluations from the LLM.

To address this, I redesigned the prompts to generate feedback in stages. First, I predefined essential conceptual keywords for each assignment. Then, the system evaluated whether these keywords were present in the submitted homework using a true/false approach. Based on this data, feedback was generated for students, ensuring fair and meaningful evaluations.

#### Reasoning and Supporting Evidence


Throughout this process, I immersed myself in various articles and presentations on language model applications, with the most enlightening being [LLM Application Building Journey (feat. Precious Customer Reviews) - Part 1](https://helloworld.kurly.com/blog/2024-review-llm-application/) from the Kurly Tech Blog. Initially, I intended to rely solely on a true/false framework, but I was inspired to craft prompts that encouraged the language model to articulate its reasoning. This enhancement significantly improved the accuracy of evaluations and feedback.

The following table illustrates the model's evaluation of a student’s explanation:

| Topic                   | Included | Explanation                                |
|-------------------------|----------|--------------------------------------------|
| Discrete Probability    | true     | Explained that the sum of probabilities equals 1. |
| Mean                    | true     | Explained the process of calculating mean and summing multiplied values before dividing. |
| Variance                | true     | Mentioned variance and its relationship to constants, though some principles were unclear. |
| Properties of Probability Distributions | false    | Did not include properties of probability distributions. |
| Binomial Distribution    | true     | Discussed independence of trials, success probability, and other conditions. |
| Properties of Binomial Distribution | true     | Explained mean and variance of np and n(1-p), respectively. |

```
Based on this table, you provided good explanations for discrete probability, mean, variance, binomial distribution, and its properties. The explanation of how to calculate the mean and the characteristics of binomial distribution was particularly well-done. However, the explanation lacked details about the properties of probability distributions.

Next time, try to provide a more thorough explanation of the properties of probability distributions. This is fundamental to understanding probability, so make sure to include it! It's great to see your dedication to studying math. Looking forward to more excellent work in the future!
```

#### Summarizing Lectures with LLM to Provide Reasoning

Initially, I considered using textbook content as a basis for reasoning. However, this approach proved ineffective, so I decided to summarize lecture content instead. I adopted a map-reduce strategy to summarize lecture transcripts generated from speech recognition. While the first input and final output were in Korean, I did not specify the language for intermediate results, allowing them to be generated in English if necessary. The actual results often included English terms alongside Korean, such as "English(Korean)," which was acceptable.

The prompt used to generate the final output is as follows:

```
This is a lecture of a Korean high school math class regarding "순열과 조합".
I want to create a "handout" for students.
Write a summary of the following in Korean.
SUMMARIZE contents that ONLY regard "순열과 조합". We don't need details about how to take this class or study strategies.
DO NOT USE English terms.
DO NOT USE content from external resources. Only summarize the content of the input.
Organize the summary in an outline format:
  1. Keyword: description
  2. Keyword: description
  3. Continue this structure as needed.
SUMMARY as detailed as possible:

"{text}"

CONCISE SUMMARY:
```

The prompts were primarily written in English, but keywords were provided in Korean. Emphasis was added using uppercase letters or quotation marks where necessary. The prompts were designed to focus solely on summarizing lecture content, excluding external references. Additionally, since lectures often included motivational content or unrelated anecdotes, these were omitted from the summaries. The final output was reviewed by an instructor, who then established grading criteria, such as "Did the explanation of A align with B?" for practical application.

Being a software engineer does not necessarily imply prior experience with AI or LLMs. However, by leveraging background knowledge commonly used in software development, exploring technical blogs, forming hypotheses, and iteratively testing and improving them, it is possible to create a functioning LLM application. If you have something in mind that you want to build but are unsure whether it is feasible, I highly encourage you to give it a try!
