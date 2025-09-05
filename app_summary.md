# PM Pathfinder - App Summary

## Overview
**PM Pathfinder** is an AI-powered, personalized learning companion designed to guide aspiring and current product managers through the diverse landscape of PM roles, with a specific focus on the tech industry and FAANG/MAANG standards. The app diagnoses a user's current knowledge, matches their interests and personality to a specialized PM career path, deeply assesses their skills for that path, and then generates a tailored, step-by-step learning plan to help them achieve their career goals.

---

## 1. Main Features

*   **AI-Powered Onboarding & Knowledge Assessment:** A conversational AI quiz that gauges the user's general PM knowledge and gives them an initial score (e.g., "Novice," "Competent," "Expert").
*   **Career Path Discovery Quiz:** A personality and interest assessment (multiple-choice, scenario-based questions) to recommend the most suitable PM specialization (e.g., Data PM, Technical PM, Growth PM).
*   **Role-Specific Deep Dive Assessment:** Once a path is chosen, a more advanced, role-specific AI quiz tests the user's depth of knowledge in that domain.
*   **Personalized Learning Roadmap (Flowchart) Generator:** The core feature. AI generates a custom learning path in a visual flowchart format, complete with topics, resources, project ideas, and milestones.
*   **Interactive Markdown Reports:** Detailed explanations for the generated roadmap, including key concepts, book recommendations, articles, videos, and case studies, all presented in a clean, downloadable Markdown format.
*   **User Profile & Progress Tracking:** Users can save their assessments, track progress on their learning roadmaps, and retake assessments as they grow.

---

## 2. App Flow (User Journey)

1.  **Welcome & Onboarding**
    *   User signs up/in.
    *   App explains its purpose: "Let's find your perfect PM role and build a plan to get you there."

2.  **General PM Knowledge Assessment (AI Quiz)**
    *   AI chatbot interface asks 10-15 fundamental PM questions.
    *   *Example: "How would you prioritize a backlog with conflicting stakeholder requests?"*
    *   User answers in free-form text.
    *   AI analyzes answers for understanding, terminology, and strategic thinking.
    *   **Output:** A score (e.g., `PMQ Score: 62/100 - You have a solid foundational knowledge!`) and a brief summary of strengths/weaknesses.

3.  **Career Path Discovery**
    *   User has two options:
        1.  **Self-Select:** Browse a list of PM specializations with descriptions and choose one.
        2.  **AI-Recommend:** Take a 5-minute personality quiz.
            *   *Example Q: "Do you enjoy analyzing A/B test results more than writing API specs?"*
    *   **Output:** A primary recommended PM role + 1-2 alternatives.

4.  **Role-Specific Deep Dive Assessment**
    *   AI asks 10-15 advanced questions specific to the chosen role.
    *   *For a **Technical PM**: "How would you handle a trade-off between using a microservices vs. a monolith architecture for a new feature?"*
    *   Users answer in free-form text.
    *   **Output:** A detailed breakdown of proficiency in key sub-skills for that role (e.g., for Data PM: SQL Proficiency, Experimentation Design). This pinpoints exact gaps.

5.  **Generate Personalized Learning Roadmap**
    *   User clicks "Generate My Plan."
    *   AI synthesizes all data: General score, chosen role, and deep-dive results.
    *   **Output:**
        *   **A Visual Flowchart:** Showing a learning sequence (e.g., "Step 1: Master SQL Basics -> Step 2: Learn Stats for A/B Testing...").
        *   **A Detailed Markdown Report:** Breaks down each step with:
            *   **Learning Objectives:** What you will achieve.
            *   **Resources:** Links to best-in-class articles, books, and courses.
            *   **Practice Ideas/Projects:** "Create a fake product and write a PRD."
            *   **Source Attribution:** Cites knowledge sources (e.g., "Based on the hiring rubric for a Facebook PMM").

6.  **Save, Track, and Iterate**
    *   The roadmap and report are saved to the user's profile.
    *   They can mark steps as complete and retake assessments later.

---

## 3. Database Schema

| Table: `users`              | Table: `assessments`                 | Table: `career_paths`         |
| --------------------------- | ------------------------------------ | ----------------------------- |
| `user_id` (PK)              | `assessment_id` (PK)                 | `path_id` (PK)                |
| `email`                     | `user_id` (FK to `users`)            | `title` (e.g., 'Technical PM') |
| `name`                      | `type` ('general', 'deep_dive_{role}') | `description`                 |
| `created_at`                | `score` (JSON)                       | `key_skills` (JSON list)      |
|                             | `responses` (JSON)                   |                               |
|                             | `created_at`                         |                               |

| Table: `learning_resources`     | Table: `user_roadmaps`            |
| ------------------------------- | --------------------------------- |
| `resource_id` (PK)              | `roadmap_id` (PK)                 |
| `title` (e.g., 'Cracking the PM Interview') | `user_id` (FK to `users`)         |
| `url`                           | `assessment_id` (FK)              |
| `type` ('book', 'article')      | `career_path_id` (FK)             |
| `topics` (JSON list of tags)    | `flowchart_data` (JSON)           |
| `associated_path_id` (FK, can be NULL) | `markdown_report` (Text)          |
|                                 | `created_at`                      |

---

## 4. Tech Stack & Implementation

*   **Frontend (Web App):** Next.js (React)
*   **Backend:** Python with FastAPI or Node.js with Express
*   **AI/LLM Integration:** OpenAI API (GPT-4)
*   **Database:** PostgreSQL
*   **Flowchart Visualization:** React Flow library
*   **Hosting:**
    *   Frontend: Vercel/Netlify
    *   Backend & DB: Railway/Render
