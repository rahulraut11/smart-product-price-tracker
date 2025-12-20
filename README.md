# Intelligent Product Price Tracker

[![Next.js](https://img.shields.io/badge/Next.js-16-black)](https://nextjs.org/)
[![Supabase](https://img.shields.io/badge/Supabase-PostgreSQL-green)](https://supabase.com/)
[![Firecrawl](https://img.shields.io/badge/Firecrawl-AI_Scraping-orange)](https://firecrawl.dev/)
[![Tailwind CSS](https://img.shields.io/badge/Tailwind-CSS-38B2AC)](https://tailwindcss.com/)

## 📖 Overview

**DealDrop** is a full-stack e-commerce intelligence application that automates product price monitoring. It solves the problem of manual price checking by allowing users to track products from major retailers (Amazon, Walmart, Zara) and receive instant alerts when prices drop.

Built with **Next.js 16** and **Supabase**, it leverages **Firecrawl's AI-powered scraping** to handle dynamic JavaScript content and anti-bot measures, ensuring reliable data extraction across different platforms.

## 🚀 Key Features

* **Universal Product Tracking:** Scrapes and tracks product metadata (price, stock, image) from any supported e-commerce URL.
* **Automated Data Pipelines:** Utilizes **Supabase `pg_cron`** to schedule daily server-side jobs that update prices without manual intervention.
* **Visual Price History:** Renders interactive charts using **Recharts** to visualize price trends over time.
* **Smart Notifications:** Integrates with **Resend** to dispatch transactional emails immediately when a discount threshold is met.
* **Secure Authentication:** Implements **Google OAuth** and database **Row Level Security (RLS)** to ensure users can only manage their own tracked items.

## 🛠️ Technical Architecture

### Tech Stack
* **Frontend:** Next.js 16 (App Router), React, Tailwind CSS, Shadcn UI
* **Backend:** Supabase (PostgreSQL, Edge Functions, Auth)
* **Scraping Engine:** Firecrawl API (Handles JS rendering & Proxy rotation)
* **Job Scheduling:** pg_cron (Database-level cron jobs)
* **Email Service:** Resend

### System Workflow
1.  **Ingestion:** User inputs a product URL. The app calls the **Firecrawl API**, which uses rotating proxies to scrape the live page data.
2.  **Storage:** Structured data is stored in **PostgreSQL**. Tables are protected by RLS policies so users cannot access each other's data.
3.  **Automation:** A daily `pg_cron` job triggers a secure API endpoint (`/api/cron/check-prices`).
4.  **Re-Scraping:** The system re-scrapes all active products in parallel.
5.  **Alerting:** If `Current Price < Historical Lowest`, a notification is triggered via the **Resend API**.

## ⚡ Getting Started

### Prerequisites
* Node.js 18+
* Supabase Account & Project
* Firecrawl API Key
* Resend API Key

### Installation

1.  **Clone the repository**
    ```bash
    git clone [https://github.com/your-username/dealdrop-tracker.git](https://github.com/your-username/dealdrop-tracker.git)
    cd dealdrop-tracker
    npm install
    ```

2.  **Environment Setup**
    Create a `.env.local` file with your credentials:
    ```env
    NEXT_PUBLIC_SUPABASE_URL=your_url
    NEXT_PUBLIC_SUPABASE_ANON_KEY=your_key
    SUPABASE_SERVICE_ROLE_KEY=your_service_role
    FIRECRAWL_API_KEY=your_firecrawl_key
    RESEND_API_KEY=your_resend_key
    CRON_SECRET=your_cron_secret
    ```

3.  **Run Development Server**
    ```bash
    npm run dev
    ```
