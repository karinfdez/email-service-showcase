# Email Service – General Overview

A specialized Cloudflare Worker that leverages the Cloudflare AI Agents SDK to automate email generation for legal outreach campaigns.This email service is composed of multiple Cloudflare AI Agents, each responsible for a specific task (fetch, generation, send), and exposed via individual endpoints to ensure flexibility, scalability, and clear separation of concerns.

## Note  
This repository is a public showcase of a private project. The actual source code is private, but this outlines the architecture, technologies used, and demo links.

## Live Demo

[YouTube demo](https://www.youtube.com/watch?v=062hPy_t3XI)


## Tech Highlights

Cloudflare AI Agents · Durable Objects · Cloudflare Queues · OpenAI (GPT-4) · Supabase · Resend API


## Features

- Automated email generation for legal outreach campaigns
- Uses the profile's data to generate personalized emails
- Integration with Supabase for data storage
- Powered by OpenAI's GPT-4 for content generation
- Durable Object Storage for persistent state management
- Secure authentication with JWT token validation


### AI Agents

- **Email Dispatcher** – Entry point. Validates the request, fetches profiles from Supabase, put them into batches of up to 50, and enqueues each batch to a Cloudflare Queue for asynchronous processing.
- **Email Processor** – Triggered by a queue message. Generates up to five personalized emails in parallel with GPT-4, captures token usage & errors, and stores both content and metrics back in Supabase.
- **Email Sender** – Fetches Supabase for emails marked *approved*, delivers them via the Resend API, and updates delivery status (`sent`, `failed`, `retried`) with timestamps back to Supabase.


## Architecture of the Email Processor Agent
The following schema illustrates how profiles are fetched, batched, queued, and processed concurrently by the Email Processor Agent:

```
        ┌───────────────┐
        │ Supabase (DB) │
        └───────┬───-───┘
                │
                ▼
     ┌─────────────────────────────┐
     │ Email Dispatcher Agent      │
     │ (fetch, filter, batch)      │
     └───────┬───────┬───────┬─────┘
             │       │       │
             │   ... │       │
             ▼       ▼       ▼
   ┌────────────────┐ ┌────────────────┐ ┌────────────────┐
   │ Batch #1       │ │ Batch #2       │ │ Batch #N       │
   │ (N profiles)   │ │ (N profiles)   │ │ (N profiles)   │
   └───────┬────────┘ └───────┬────────┘ └───────┬────────┘
           │                  │                  │
           └─────────┬────────┴─────────┬────────┘
                     │                  │
                     ▼                  ▼
           ┌───────────────────────────────────────────┐
           │           Background Queue                │
           └───────┬────────────┬────────────┬────-────┘
                   │            │            │
                   ▼            ▼            ▼
   ┌───────────────────────────────┐ ┌───────────────────────────────┐ ┌───────────────────────────────┐
   │ Email Processor Agent (inst 1)│ │ Email Processor Agent (inst 2)│ │ Email Processor Agent (inst N)
   │   processes Batch #1          │ │   processes Batch #2          │ │   processes Batch #N          │
   └──────────────┬────────────────┘ └──────────────┬────────────────┘ └──────────────┬────────────────┘
                  │                                 │                                 │
                  ▼                                 ▼                                 ▼
         ┌───────────────────────┐        ┌───────────────────────┐        ┌───────────────────────┐
         │ 5 profiles in parallel│        │ 5 profiles in parallel│        │ 5 profiles in parallel│
         │   (max, per instance) │        │   (max, per instance) │        │   (max, per instance) │
         └───────────────────────┘        └───────────────────────┘        └───────────────────────┘
                  │                                 │                                 │
                  ▼                                 ▼                                 ▼
         ┌───────────────┐                ┌───────────────┐                ┌───────────────┐
         │ Save results  │                │ Save results  │                │ Save results  │
         │ to Supabase   │                │ to Supabase   │                │ to Supabase   │
         └───────────────┘                └───────────────┘                └───────────────┘
```

**Concurrency Model:**
- The Email Dispatcher Agent splits profiles into batches (up to 50 per batch) and enqueues them.
- Each Email Processor Agent instance processes a batch, generating up to 5 emails in parallel (concurrency = 5).
- If there are multiple batches, multiple processor agent instances can run in parallel, multiplying the total parallelism (e.g., 4 batches × 5 concurrency = 20 emails processed in parallel).
- This design helps avoid API timeouts and OpenAI rate/token limits by controlling both batch size and per-agent concurrency.


## Project Structure

```text
├── src/
│   ├── agents/
│   │   ├── email-dispatcher.ts
│   │   ├── email-processor.ts
│   │   ├── email-sender.ts
│   ├── utils/
│   │   ├── supabase.ts
│   │   └── ...
│   ├── worker.ts            # Worker entry (routes, DO routing)
│   └── ...                  # React UI components, hooks, libs, etc.
├── wrangler.toml            # Worker & queue configuration
├── package.json             # Dependencies & scripts
└── README.md
```

