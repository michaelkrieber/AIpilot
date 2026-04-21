# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

AIpilot is an early-stage project defining and implementing a **digital credit card sales platform** (currently for the Miles & More Kreditkarte product). The repository is in a documentation-first phase: business journeys are specified as structured Markdown before application code is written.

The intended deployment target is **AWS ECS** via Docker containers. The CI/CD pipeline builds and pushes Docker images to Amazon ECR on every push to `main`.

## Repository Structure

- `journeys/` — Business process specifications written in Markdown with embedded Mermaid flowcharts. These are the source of truth for feature requirements. New journeys follow the same format.
- `.github/workflows/aws.yml` — GitHub Actions pipeline that builds a Docker image and deploys to Amazon ECS. All placeholder values (`MY_AWS_REGION`, `MY_ECR_REPOSITORY`, etc.) must be replaced with real values before the pipeline is functional. Required GitHub Actions secrets: `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`.

## Architecture & Key Conventions

### Journey documents
Each file in `journeys/` describes one sales or onboarding flow. The structure is:
1. **Ziel** — goal of the journey
2. **Prozessübersicht** — Mermaid `flowchart TD` diagram of the happy path and branches
3. **Detaillierte Schritte** — numbered step-by-step descriptions
4. **Business-Regeln** — numbered business rules prefixed `BR-XX`
5. **Statusmodell** — the ordered list of application status values as code literals

When implementing application code, the status model and business rules in the journey document are the authoritative specification.

### Miles & More credit card journey (current)
The core flow for `journeys/miles-and-more-kreditkarte-journey.md`:
- Customers optionally upload a card complete invoice (≤ 3 months old) for automatic limit confirmation.
- If OCR parsing succeeds and customer data matches, the Risk Engine auto-confirms the limit (**Path A**).
- Any failure (missing upload, invoice too old, OCR failure, data mismatch) routes to manual risk review (**Path B**).
- A contract is only activated (`PRODUCT_ACTIVE`) after a digital signature is received.

### CI/CD pipeline
- Trigger: push to `main`
- Steps: checkout → configure AWS credentials → ECR login → `docker build` → `docker push` → update ECS task definition → ECS deploy
- The Dockerfile must exist at the repository root for the pipeline to work.
- ECS task definition JSON should be stored at the path set in `ECS_TASK_DEFINITION` (e.g. `.aws/task-definition.json`).

## What Needs to Be Built

The following are absent and required before the project can run:
1. A `Dockerfile` at the repository root
2. Application source code implementing the journeys
3. An ECS task definition JSON file (e.g. `.aws/task-definition.json`)
4. Populated environment variables in `.github/workflows/aws.yml`
5. A README.md
