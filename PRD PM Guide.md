Product Requirements Document (PRD Simple version): PM Pathfinder
Version: 1.0
Date: August 26, 2025
Author: Syarif Ayaturahman
Status: Draft

1. Overview & Vision
Product Name: PM Pathfinder
Vision: To become the definitive personalized career companion for anyone navigating a product management career in tech, demystifying the path to specialized roles at top companies.
Problem: The path to becoming a Product Manager, especially in specialized domains (Data, Growth, Technical) at FAANG/MAANG companies, is unclear. Aspiring PMs lack a structured way to assess their skills, identify the right specialization for their strengths, and get a customized, actionable plan to bridge their skill gaps.
Solution: An AI-powered platform that diagnoses a user's current PM knowledge, matches them to an ideal PM specialization, performs a deep skill assessment for that role, and generates a personalized, resource-rich learning roadmap.

2. Goals & Success Metrics
Goal	Description	Key Metric (KPIs)
User Acquisition	Drive initial sign-ups and product discovery.	• 1,000 registered users in the first 3 months
User Engagement	Measure core feature usage and value.	• 40% of users complete a full assessment cycle
• 25% roadmap completion rate
Product Efficacy	Measure the perceived value of the output.	• Average user rating of generated roadmaps > 4.0/5.0
• "Would you recommend this tool?" (NPS) > 30
Learning	Validate core assumptions.	• Assumption: Users need help choosing a path.
*Metric: >60% use the AI-Recommendation quiz.*
3. User Personas
Aspiring Alex: A software engineer or recent grad who wants to break into PM but doesn't know where to start or how to stand out. Needs direction and a foundational plan.

Climbing Chloe: A current APM or PM at a mid-size company who wants to transition to a FAANG company or specialize (e.g., move from general PM to Technical PM). Needs advanced, role-specific gap analysis and a targeted plan.

4. User Stories & Requirements
Epic 1: User Onboarding & Core Assessment
US-1.1: As a new user (Alex), I can create an account using my email so that my progress is saved.

Acceptance Criteria (AC): User table is updated. JWT token is issued upon successful sign-up.

US-1.2: As a user, I can take a conversational, AI-driven quiz on general PM principles so that I can get a baseline score of my knowledge.

AC: Quiz presents 10-15 free-text questions sequentially. Answers are sent to the backend and stored.

Epic 2: Career Path Discovery
US-2.1: As a user, I can choose to either browse all available PM specializations or take a guided personality quiz to get a AI-recommended path.

US-2.2: As a user, I want to see a description of each PM specialization (Data, Technical, Growth, etc.) before I make a choice.

AC: The career_paths table is populated with compelling descriptions.

Epic 3: Deep Dive & Roadmap Generation (The Core AI Magic)
US-3.1: As a user who has chosen a path, I can take a second, more advanced AI quiz specific to that role so that my exact skill gaps can be identified.

US-3.2: As a user, I can click a "Generate My Plan" button to create a personalized learning roadmap based on all my assessment data.

AC: The backend calls the OpenAI API with a engineered prompt containing the user's path, skills gaps, and general score. The response is parsed and saved to the user_roadmaps table.

US-3.3: As a user, I can view my personalized plan as both an interactive visual flowchart and a detailed markdown document with resources and explanations.

Epic 4: User Profile & Progress Tracking
US-4.1: As a user, I can view a dashboard of my completed assessments and generated roadmaps.

US-4.2: As a user, I can mark steps on my roadmap as "Complete" to track my progress.

5. Out of Scope (V1)
Social features / sharing roadmaps.

A dedicated mobile app (iOS/Android); V1 is a responsive web app.

Video content hosted directly on the platform.

Paid subscriptions / monetization.

6. Design & UX Guidelines
Style: Clean, modern, minimalist. Use a professional color scheme (blues, grays, greens). Avoid playful or childish themes.

Tone: Conversational, supportive, and expert—like a knowledgeable career coach.

Interaction: Single-page application (SPA) feel. Minimize page reloads. Use clear progress indicators during assessments.


----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------


-- Users table to store core user information
CREATE TABLE users (
    user_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email VARCHAR(255) UNIQUE NOT NULL,
    full_name VARCHAR(100),
    hashed_password VARCHAR(255) NOT NULL, -- For email/password auth. Use `NULL` if using OAuth only.
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Table to store the pre-defined career paths user can choose from
CREATE TABLE career_paths (
    path_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    title VARCHAR(100) NOT NULL UNIQUE, -- e.g., 'Technical Product Manager'
    description TEXT NOT NULL,
    key_skills JSONB NOT NULL DEFAULT '[]' ::JSONB, -- e.g., ["System Design", "API Design", "Technical Debt Prioritization"]
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Central table to store every assessment a user takes
CREATE TABLE assessments (
    assessment_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(user_id) ON DELETE CASCADE,
    type VARCHAR(50) NOT NULL, -- 'general_pm', 'career_quiz', 'deep_dive_technical', 'deep_dive_growth'
    title VARCHAR(255) NOT NULL, -- e.g., 'General PM Skills Assessment'
    score_summary JSONB, -- Flexible field for scores. e.g., {"overall": 75, "strategy": 80, "execution": 70}
    raw_responses JSONB NOT NULL, -- Stores the full Q&A log for the assessment
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    CONSTRAINT fk_user FOREIGN KEY(user_id) REFERENCES users(user_id)
);

-- Table to store pre-curated learning resources (books, articles, courses)
CREATE TABLE learning_resources (
    resource_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    title VARCHAR(255) NOT NULL, -- e.g., 'Cracking the PM Interview'
    url VARCHAR(500),
    type VARCHAR(50) NOT NULL, -- 'book', 'article', 'video', 'course', 'podcast'
    description TEXT,
    topics JSONB NOT NULL DEFAULT '[]' ::JSONB, -- e.g., ["Interviewing", "Strategy", "Metrics"]
    associated_path_id UUID REFERENCES career_paths(path_id), -- If NULL, it's a general PM resource
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- The core table storing the AI-generated roadmap for a user
CREATE TABLE user_roadmaps (
    roadmap_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(user_id) ON DELETE CASCADE,
    assessment_id UUID NOT NULL REFERENCES assessments(assessment_id), -- The deep-dive assessment that triggered this
    career_path_id UUID NOT NULL REFERENCES career_paths(path_id),
    title VARCHAR(255) NOT NULL DEFAULT 'My Learning Roadmap',
    flowchart_data JSONB NOT NULL, -- The structured JSON for the React Flow chart
    markdown_report TEXT NOT NULL, -- The full AI-generated markdown text report
    is_active BOOLEAN NOT NULL DEFAULT TRUE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Table to track user progress on their roadmap steps
CREATE TABLE user_progress (
    progress_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(user_id) ON DELETE CASCADE,
    roadmap_id UUID NOT NULL REFERENCES user_roadmaps(roadmap_id) ON DELETE CASCADE,
    -- This points to a specific node ID within the roadmap's `flowchart_data`
    step_id VARCHAR(100) NOT NULL,
    -- Status: not_started, in_progress, completed, skipped
    status VARCHAR(20) NOT NULL DEFAULT 'not_started',
    notes TEXT,
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    -- A user can only have one status per step per roadmap
    UNIQUE(user_id, roadmap_id, step_id)
);

-- Indexes for performance
CREATE INDEX idx_assessments_user_id ON assessments(user_id);
CREATE INDEX idx_user_roadmaps_user_id ON user_roadmaps(user_id);
CREATE INDEX idx_user_progress_roadmap_id ON user_progress(roadmap_id);
