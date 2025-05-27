# My AI Agent Portfolio: Advanced Content Automation Workflow

## Overview
This repository showcases a sophisticated, multi-platform AI automation system designed to streamline the generation, review, and publishing of cutting-edge tech news content to LinkedIn, complete with dynamic image generation. This project demonstrates robust integration capabilities between Zapier and Make.com.

## The Challenge
In the fast-paced world of emerging technologies, particularly within the African and Nigerian tech ecosystems, delivering fresh, relevant, and visually engaging content is paramount. Traditional methods often lead to:
* **Outdated Information:** Relying on LLM's static training data for news.
* **Manual Bottlenecks:** Time-consuming human intervention for drafting, fact-checking, and visual creation.
* **Lack of Visual Engagement:** Text-only posts struggling to capture attention on visually-driven platforms like LinkedIn.
* **Inefficient Workflow:** Disconnected processes for content generation, review, editing, and publishing.

## The Solution: A Hybrid AI Automation Agent
This system tackles these challenges by integrating advanced AI capabilities with automation platforms, creating a seamless, semi-automated workflow for high-quality content delivery. It acts as an "AI Agent" overseeing content creation and dissemination.

## Key Features
* **Automated Fresh News Sourcing:** Integrates with **NewsAPI.org** (or Serper API) to fetch the latest (last 48 hours) developments, specifically focusing on **agentic AI** and relevant tech/African/Nigerian news sources.
* **Multi-Stage AI Content Generation:** Leverages multiple **Google Gemini 1.5 Pro LLM** instances for:
    * News summarization.
    * Headline selection (best of three generated options).
    * LinkedIn post drafting (hook, summary, CTA, hashtags).
    * Fact-checking and humanizing of the post.
* **Dynamic AI Image Generation:** Utilizes **Google Vertex AI (Imagen)** to create visually stunning, AI-generated social media graphics (single headline cards) that accompany each post, ensuring visual engagement. Handles text rendering challenges with careful prompting.
* **Hybrid Platform Integration:** Seamless communication and robust data transfer between **Zapier** (for initial triggers, AI chain, web search, email sending) and **Make.com** (for robust Data Store management, synchronous webhook responses, Google Docs integration, and LinkedIn publishing).
* **Human-in-the-Loop Review & Conditional Workflow:** Sends email notifications to the user with:
    * A preview of the generated post text and image.
    * Clickable "Approve" and "Reject" links for conditional publishing or editing.
* **Conditional Publishing & Editing Workflow:**
    * **Approve:** Automatically publishes the content (text + image) to LinkedIn.
    * **Reject:** Sends the post to Google Docs for manual editing, providing a direct link to the document and a "Publish Edited" button/link.
* **Post-Editing Automated Publishing:** Allows for the automated publishing of the manually edited Google Docs content (along with the original image) to LinkedIn, ensuring revised content is published efficiently.
* **Robust Data Management:** Employs **Make.com's Data Store** for persistent, unique ID-based storage and retrieval of post drafts, images, and their statuses across various stages of the workflow. Data is linked via a dynamically generated `executionId`.

## Architecture Overview
The system is built as a hybrid automation pipeline, orchestrated across Zapier and Make.com to leverage each platform's strengths and overcome individual platform limitations.

```mermaid
graph TD
    subgraph Zapier (Source Zap)
        A[Schedule Trigger] --> B{Code: Web Search (NewsAPI.org)}
        B --> C{LLM 1: News Summaries}
        C --> D{LLM 2: Draft Post & JSON Output}
        D --> E{Code: UUID Generation}
        E --> F1[Formatter: Remove ```json]
        F1 --> F2[Formatter: Remove ```]
        F2 --> G{Code: JSON Parse & Text Clean}
        G --> H{Code: Array Joiner}
        H --> I{Google Vertex AI: Image Generation}
        I --> J{Webhooks POST to Make.com Save Draft}
        J --> K[Email: Review with Links (Approve/Reject)]
    end

    subgraph Make.com (Backend & Response)
        L[Webhook Trigger (from Zapier)] --> M{Data Store: Add/Update Record}
        M --> N[Webhook Response: Send back Unique ID]

        O[Webhook Trigger (from Approve Link)] --> P{Data Store: Retrieve Record}
        P --> Q[HTTP: Get Image File]
        Q --> R[LinkedIn: Create User Image Post]

        S[Webhook Trigger (from Reject Link)] --> T{Data Store: Retrieve Record}
        T --> U[Google Docs: Create Document for Editing]
        U --> V[Email: Editing Link & Publish Edited Link]

        W[Webhook Trigger (from 'Publish Edited' Link)] --> X{Google Docs: Get Document Content}
        X --> Y{Data Store: Retrieve Record}
        Y --> Z{HTTP: Get Image File}
        Z --> AA[LinkedIn: Create User Image Post]
    end

    K --> O
    K --> S
    V --> W
    N -- Unique ID --> J
    L -- Text + Image URL + ID --> M
    M -- Unique ID --> N
    P -- Text + Image URL --> Q
    T -- Text + Image URL --> U
    X -- Edited Text --> AA
    Y -- Image URL --> Z
