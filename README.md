# Email Service – General Overview

A cloud-native email delivery system that automates personalized, large-scale legal outreach. Built on Cloudflare Workers with AI Agents (GPT-4) and Supabase.

## Note  
This repository is a public showcase of a private project. The actual source code is private, but this outlines the architecture, technologies used, and demo links.

## Live Demo

[video demo link]([https://drive.google.com/drive/folders/1Hnw7jDZAr7XQ2aRq-83AOX2CdviJN0YK](https://www.youtube.com/watch?v=062hPy_t3XI))


## Key Features

• AI-generated, company-specific outreach emails  
• GPT-4 personalization using profile data  
• Supabase for storage & analytics  
• Cloudflare Durable Objects for per-company state  
• Secure JWT-based auth  
• Resend API for reliable delivery

## AI Agents

| Agent | What it does |
|-------|--------------|
| **Email Dispatcher** | Validates request ➜ fetches profiles ➜ batches (≤50) ➜ enqueues each batch. |
| **Email Processor** | Triggered by queue ➜ generates ≤5 emails in parallel with GPT-4 ➜ saves content & metrics. |
| **Email Sender** | Polls Supabase for *approved* emails ➜ sends with Resend ➜ marks `sent/failed/retried`. |



## Tech Highlights

Cloudflare Workers · Durable Objects · Cloudflare Queues · OpenAI GPT-4 · Supabase · Resend API
