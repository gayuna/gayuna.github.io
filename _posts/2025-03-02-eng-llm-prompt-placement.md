---
title: Storing LLM Prompts for Better Productivity
categories:
  - LLM
tags:
  - LLM
  - Prompt
  - Prompt Engineering
  - English
---

With AI evolving rapidly, keeping up with the latest tools and technologies can feel overwhelming. At one point, I considered going to graduate school, but I quickly realized that technology is advancing faster than I could complete a degree. So instead, I decided to focus on using these tools effectively.

The LLM (Large Language Model) space is highly competitive, with rankings shifting frequently. Rather than constantly switching between services to use the latest model, I’ve chosen to stick with one platform and focus on optimizing my workflow within it.

Prompt engineering is key to using LLMs effectively, especially for repetitive tasks where the same prompts are used repeatedly. Instead of saving prompts in a text file and copying them when needed, organizing them systematically for quick access is a much better approach.

Here are three ways to manage and use prompts more efficiently. This post was inspired by [Jeff Su’s video](https://youtu.be/j63bBK_ct-M?si=yYAQIGinhIMdO1KX).

#### Save General Prompts as Snippets

Some prompts are used frequently regardless of specific projects or schedules. These can be saved as snippets for quick access. Mac users often use Alfred, but I personally prefer Raycast, which has a built-in snippet feature. When saving a snippet, it’s best to use a memorable but unique keyword like `prompt-taskname` or `chatgpt-function` to avoid conflicts with regular words.

##### 1. Saving Prompts for JD Analysis

For example, I frequently analyze JDs for Canadian companies, tailoring resumes, generating interview questions, and formatting experiences using the STAR method. Instead of typing the same prompt repeatedly, I created a snippet triggered by `chatgpt-gd`, which expands into:

```
You’re a seasoned hiring manager with over 20 years of experience. You are responsible for this job posting. Highlight the 3 most important responsibilities in this job description:
```

Now, I simply paste the JD, and ChatGPT quickly identifies the key responsibilities.

##### 2. Using Snippets for Development Work

Snippets are also helpful in software development. A good example is generating curl commands. While tools like HTTPie simplify API testing on a local machine, they might not be available on remote servers. Rather than manually constructing lengthy curl commands, I store a prompt in my snippet manager. When needed, I trigger the snippet, input the API spec, and let ChatGPT generate the correct command. This approach reduces friction and speeds up the workflow.

Most frequently used prompts can be stored as snippets, and they can also be combined with other management techniques. For instance, instead of saving full prompts separately, you can maintain a list of snippet keywords and call them when needed.

#### Store Repeating Task Prompts in Calendar Events

##### 1. Managing Recurring Work with Calendar Prompts

If a prompt is tied to a recurring task, storing it in a calendar event can be an effective approach.

For example, many teams have weekly meetings or monthly one-on-ones with their manager. Preparing for these meetings often involves summarizing recent work. If you already keep track of your tasks, you can use a prompt to automatically generate a concise summary.

A simple way to do this is by adding the prompt to the event description when scheduling the meeting. Set a 10-minute time block for preparation, and when the reminder pops up, run the prompt to generate your summary quickly. (Just remember to update the time block if the meeting schedule changes.)

##### 2. Storing Prompts for Performance Review Prep

This approach also works well for quarterly or biannual performance reviews. These reviews typically require summarizing work done over a specific period, which takes time to prepare.

I often ask myself: “How can I quantify what I’ve accomplished?” To streamline this process, I created a prompt that converts written work descriptions into measurable metrics.

The best way to use this is by scheduling a calendar reminder when it’s time to start working on the review. That way:

1.	You get a reminder that it’s time to start.
2.	The event description contains the prompt, so you can immediately run it to structure your review.

Using calendar events to store prompts helps reduce prep time and ensures consistency in how you approach recurring tasks.

#### Save Project-Specific Prompts as Files

For prompts tied to a specific project, storing them in a dedicated file within the project’s repository is a practical approach. These files can be added to `.gitignore` to keep them out of version control.

After using `.http` files in IntelliJ for API testing, I started saving frequently used SQL queries and other related prompts as files. Since working within an IDE is already part of my routine, having prompts readily accessible within the same environment makes sense.

##### 1. Prompt to use for Repetitive DML Operations

In many cases, bulk data updates require writing repetitive DML queries.

For instance, suppose you need to update flight details for hundreds of shipping records. A common approach is to receive an Excel file, use CONCAT functions to generate UPDATE queries, and run them manually. Since these requests follow a predictable format, you can streamline the process by saving a prompt that generates the necessary SQL updates from an input dataset. If the approach works well, you can store it as a file for future use.

##### 2. Managing Prompts for Refine Blog Posts

Prompts are also useful when writing blog posts. Before publishing, I often ask an LLM to check grammar or suggest readability improvements. Since these requests are repetitive, I keep a prompt file in my GitHub repository and reuse it whenever needed.

##### 3. Team-Wide Prompt Management

While adding prompt files to .gitignore helps prevent personal settings from being committed, the opposite approach can also be useful. If your team uses LLMs frequently and has a set of shared prompts, storing them in a repository can be beneficial. Just like code, prompts can be refined, updated, and version-controlled to improve their effectiveness over time.

We’ve explored several approaches so far, but as the saying goes, "A tool is only useful when put to good use." Rather than storing 100 prompts that rarely get used, having 10 well-placed, frequently used prompts will likely make LLMs much more effective in practice. If you have any additional ideas, feel free to reach out to me on LinkedIn and share!
