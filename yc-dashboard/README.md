YC Company Scraper & Intelligence Dashboard


1. Architecture Overview

This project operates as a three-tier local application, completely decoupled to ensure modularity and ease of debugging.

Frontend (Dashboard): Built with Next.js 14 (App Router). It serves the UI on localhost:3000 and communicates with the database via server-side API routes.

Database: PostgreSQL (running locally on port 5432). It acts as the centralized source of truth for both the scraper and the dashboard.

Data Engine (Scraper): A standalone Python script that executes independently to fetch data from the web and populate the database.

Data Flow:

Python Scraper ➜ Writes to ➜ Local PostgreSQL ➜ Reads from ➜ Next.js API


2. Database Schema
The database is normalized to support Time-Travel Analysis, separating immutable company identity from changing attributes.

companies Table
Stores static identity. Prevents duplicates.

id (PK): Unique Integer.

name: Company Name (indexed for search).

domain: Official URL.

company_snapshots Table
Stores historical states. A new row is created for every scrape run.

id (PK): Unique Integer.

company_id (FK): Links to companies.

batch: YC Batch (e.g., W24, S23).

stage: Status (Active, Acquired, Dead).

location: HQ Location.

description: Pitch/Bio.

scraped_at: Timestamp (UTC).


3. Incremental Scraping Logic

To ensure data integrity and avoid overwriting history, the scraper uses a Snapshot Strategy:

Identity Check: The scraper first checks if a company exists in the companies table using its name or domain.

If New: Insert into companies → Get new id.

If Existing: Retrieve existing id.

Snapshot Insertion: It never updates old rows. Instead, it inserts a new record into company_snapshots with the current timestamp.

Latest View: The dashboard queries only the latest snapshot using a specific SQL strategy:

SQL

SELECT * FROM companies c
LEFT JOIN company_snapshots s ON c.id = s.company_id
WHERE s.scraped_at = (
    SELECT MAX(scraped_at) FROM company_snapshots WHERE company_id = c.id
)


4. Performance Metrics


Query Latency: Optimized SQL joins ensure sub-50ms response times on local datasets.

Connection Pooling: The Next.js backend uses pg pooling to handle multiple API requests without exhausting database connections.

Pagination: Implements server-side LIMIT and OFFSET to render data efficiently, regardless of database size.

Visual Performance: Charts utilize fixed-height containers to prevent layout shifts and ensure responsiveness.


5. Local Setup Guide


Follow these steps to run the project on your machine.

Prerequisites
Node.js 18+

Python 3.10+

PostgreSQL (Local Service)

Step 1: Database Configuration
Ensure your local PostgreSQL service is running.

Port: 5432

Database Name: postgres

User/Password: postgres / 12341 (or your specific credentials).

Step 2: Run the Scraper (Python)
The scraper must run first to populate the database.

Navigate to the project folder.

Open scraper_v6.py and verify DB_CONFIG matches your local DB.

Install dependencies and run:

Bash

pip install psycopg2-binary
python scraper_v6.py

Step 3: Run the Dashboard (Next.js)
Create/Update the .env file in the root directory:

Code snippet

DATABASE_URL="postgresql://postgres:12341@localhost:5432/postgres"
Start the development server:

Bash

npm install
npm run dev
Open http://localhost:3000.