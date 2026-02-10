# Full-Stack-Programmer-Take-Home-Evaluation

This document will list the steps a candidate for the **Full Stack Programmer** position will perform to evaluate skill level, proficiency, and ability to deliver a solution meeting the requirements in a timely manner.

## Requirements

- IDE or Text Editor: Visual Studio / VS Code / whatever you prefer.
- Database engine: **PostgreSQL** (Docker preferred).
- Backend: **C#/.NET**
- Frontend: **Angular + TypeScript** (React is fine if you explain why).
- Testing Framework:
  - Backend: xUnit/NUnit (or equivalent)
  - Web UI automation: Playwright/Selenium

- Version Control: Git + GitHub
- CI/CD Tool: **Azure DevOps** (YAML pipeline)
- Optional: Android environment if you choose to do the bonus task.

## Overview

You will be building a small, realistic feature slice that looks like the kind of work we do in POS backoffice:

**Inventory Adjustment Batch** — create a batch, scan/search SKUs, adjust quantities/cost, and post the batch safely (idempotent).

## Instructions

1. **Clone the Repository**: Start by cloning the provided repository. The repository URL will be provided in the job posting.

2. **Set Up the Environment**: Ensure you have the necessary tools and dependencies installed to run the API and Postgres locally. Docker is preferred.

3. **Database Schema**: Create or extend a schema that supports inventory adjustment batches.

   Create tables (or equivalents) for:
   - `store`
   - `item` (must include `sku`, `title`)
   - `inventory_adjustment_batch` (header)
   - `inventory_adjustment_line` (lines)
   - `inventory_ledger` (or equivalent posted results table)

   Minimum requirements:
   - indexes on `item.sku` and `inventory_adjustment_line.batch_id`
   - foreign keys + sensible constraints
   - seed at least:
     - 2 stores
     - 25 items
     - 1 draft batch with 2-5 lines

4. **Backend API**: Implement endpoints to support the workflow.

   ### A) Create Draft Batch

   `POST /api/inventory/adjustments/batches`

   Request:

   ```json
   {
     "storeId": "S-100",
     "reasonCode": "DAMAGE",
     "notes": "Backroom water leak"
   }
   ```

   ### B) Add/Update Line by SKU (Scan-friendly)

   `PUT /api/inventory/adjustments/batches/{batchId}/lines/by-sku`

   Request:

   ```json
   {
     "sku": "ABC-123",
     "qtyDelta": -1,
     "unitCost": 3.25,
     "note": "Damaged packaging"
   }
   ```

   Rules:
   - If line exists, **increment qty** by `qtyDelta`
   - If qty becomes **0**, remove the line
   - SKU must exist
   - `qtyDelta` cannot be 0
   - If `reasonCode` is `"RECEIVING"`, require `unitCost` (otherwise optional)

   ### C) Get Batch Detail

   `GET /api/inventory/adjustments/batches/{batchId}`
   Returns header + expanded line details (item title, sku, qty, etc.).

   ### D) Post Batch (Idempotent)

   `POST /api/inventory/adjustments/batches/{batchId}/post`

   Rules:
   - Draft only
   - Must have ≥ 1 line
   - Posting writes immutable ledger entries (or equivalent) and marks batch as Posted
   - **Retry-safe:** multiple calls must not double-apply inventory changes

   Implementation notes:
   - Use either a transactional "status transition + unique constraint" approach, or support an `Idempotency-Key` persisted server-side.
   - Explain your approach in `NOTES.md`.

   Backend expectations:
   - consistent errors (ProblemDetails preferred)
   - validation layer (FluentValidation or equivalent)
   - transaction boundaries on Post
   - basic structured logging

5. **Frontend (Angular + TypeScript)**: Build a small UI for the workflow.

   Screen 1: **Batches**
   - list draft + posted batches
   - create new draft batch (store, reason, notes)
   - open a batch detail

   Screen 2: **Batch Detail**
   - SKU input ("Scan or type SKU")
   - Enter key adds item with default `qtyDelta = +1`
   - line list with quick +/- controls
   - editable unit cost (only show/enable when reasonCode = RECEIVING)
   - "Post Batch" button with confirm dialog
   - mobile-friendly layout (responsive)

6. **Automated Tests**: Add automation that proves the key behaviors work.

   Minimum:
   - Backend tests for:
     - increment logic
     - line removal when qty hits 0
     - post idempotency

   - Web automation:
     - Playwright test covering create batch → add SKU → post → verify "Posted"

7. **CI Pipeline**: Set up a CI pipeline to automate build + test.

   The pipeline should:
   - restore and build
   - spin up Postgres using Docker Compose
   - run backend tests
   - run Playwright tests (headless)
   - publish test results
   - fail if code coverage is below **70%**

8. **Documentation**: Document everything needed to run and validate the solution.
   - how to run API, web, tests locally
   - environment variables
   - anything special for Playwright/DB setup
   - add `NOTES.md` with tradeoffs, assumptions, and what you'd improve next

9. **Bonus Task (Optional)**: If you're comfortable with Android, write an Espresso/Robolectric test that simulates scanning two barcodes and verifies quantity increments (or similar lightweight POS workflow automation).

10. **Submission**: Once you've completed the tasks, push your code to a GitHub repository.

- Include your full name and the method/job site you used to reach the repository in the README.
- Send an invitation to your GitHub repository to Rapid POS at:
  `recruiting.programmer@rapidpos.com`
