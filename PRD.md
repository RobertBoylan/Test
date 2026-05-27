# LinkStash

_Product Requirements Document — May 27, 2026_

## Overview

LinkStash replaces the mess of browser bookmarks, saved tabs, and "I'll read this later" links that never get read. Drop in any URL and the app fetches the page, generates a short AI summary, and auto-tags it by topic. Full-text search across all your saved links means you can find that article you read three months ago in seconds.

The app is designed for personal use — no social features, no sharing, no feeds. Just a fast, clean way to save and find anything you've come across online. Works as a web app with a browser extension for one-click saving.

## Features

### Must Have

#### 1. AI Summaries

Generates a 2-3 sentence summary of each saved link using AI. Summaries are searchable and help you remember why you saved it.

**Products:** Pro, Lifetime

**Acceptance criteria:**

- Summary appears within 10 seconds of saving a link
- Summary is 2-3 sentences, max 280 characters
- Failures fall back to the page description and surface a retry button

#### 2. Auto-Tagging

AI analyzes the content and assigns relevant tags automatically. You can also add custom tags manually.

**Products:** Free, Pro, Lifetime

**Acceptance criteria:**

- Assigns 2-5 tags per link, drawn from the user's existing tags when possible
- User can edit, remove, or add tags on any link
- Tag suggestions update when the link's content changes

#### 3. Full-Text Search

Search across titles, summaries, tags, and original page content. Find anything you've ever saved in seconds.

**Products:** Free, Pro, Lifetime

**Acceptance criteria:**

- Returns matching results in under 200ms for libraries up to 10K links
- Matches partial words and handles typos with fuzzy ranking
- Filters results by tag, collection, and date range

#### 4. Reading List Dashboard

Personal dashboard showing reading progress, time-to-read estimates, and recent saves at a glance.

**Products:** Pro, Lifetime

**Acceptance criteria:**

- Dashboard loads in under 300ms with up to 1k saved links
- Time-to-read estimates are derived from word count of the saved page
- Recent saves section shows the last 10 items with thumbnail and summary snippet

### Nice to Have

#### 1. Collections

Group links into themed collections (e.g., 'Design Inspiration', 'React Libraries'). Drag-and-drop to organize.

**Products:** Pro, Lifetime

**Acceptance criteria:**

- User can create, rename, and delete collections from the sidebar
- Links can be added to a collection via drag-and-drop or context menu
- A link can belong to multiple collections

#### 2. Read-later Mode

Mark links as 'read later' to surface them in a dedicated queue. Removes clutter from the main feed while keeping important links visible.

**Products:** Pro, Lifetime

**Acceptance criteria:**

- Toggle on any link card switches its state to read-later
- Dedicated 'Read later' view lists all flagged links
- Marking a link as read clears the flag and returns it to the main feed

#### 3. Bulk Import

Import bookmarks from Chrome, Firefox, or Raindrop in one step. Parses browser export files and auto-tags all imported links.

**Products:** Pro, Lifetime

**Acceptance criteria:**

- Accepts HTML export from Chrome / Firefox / Raindrop
- Shows progress bar and an error report for failed rows
- Auto-tags imported links using existing AI tagging pipeline

### Future

#### 1. Weekly Digest

Email digest of your unread saves from the past week with AI-generated highlights.

**Products:** Pro, Lifetime

#### 2. Tag Management

Dedicated tag editor to rename, merge, and delete tags across your entire library. See tag usage counts and clean up duplicates.

**Products:** Lifetime

**Acceptance criteria:**

- User can rename a tag everywhere it appears in one action
- Merging tags moves all assignments to the chosen target and removes the merged tag
- Tag list shows usage count per tag and supports sort by count or name

### Completed

#### 1. One-Click Save

Browser extension and web clipper that saves any URL with one click. Automatically fetches page title, description, and favicon.

**Products:** Free, Pro, Lifetime

**Acceptance criteria:**

- Saves any URL in under 1 second without leaving the current page
- Persists title, description, and favicon from OG metadata when available
- Shows a toast confirmation with an 'Undo' option
- Works on both Chrome and Firefox

#### 2. Browser Extension Sync

Chrome and Firefox extension that syncs saved links instantly with the web app. One-click save from any page without leaving the browser.

**Products:** Free, Pro, Lifetime

**Acceptance criteria:**

- Extension installs from Chrome Web Store and Firefox Add-ons
- Syncs new saves to the web app in under 2 seconds
- Works offline and queues saves until the browser reconnects


## Design

### Navigation Pattern

top_nav

### Onboarding Pattern

progressive

| **Design Style** | **Copy Tone** |
|:--|:--|
| Minimal and clean | Friendly & casual |

| **Colors** | **Fonts** |
|:--|:--|
| #6366f1 · #06b6d4 · #f59e0b | Header: Space Grotesk · Sub: Poppins · Body: Inter |

| **UX Patterns** | **First User Action** |
|:--|:--|
| Dashboard, Search / filter, Notifications | Paste your first URL into the save bar to see AI summaries and auto-tagging in action. |

## Technology

### Platforms

- **Web App** — Responsive web app that works on desktop and mobile browsers. PWA-ready for home screen install.
- **Browser Extension** — Chrome and Firefox extension for one-click saving from any page.

### Technology Stack

- **React / Vite / Tailwind** — React with Vite and Tailwind CSS. Minimal dependencies, fast page loads.
- **Supabase** — Supabase for auth, database, and full-text search. Edge functions for AI processing.
- **Vercel** — Vercel for frontend deployment. Automatic preview deployments on every PR.
- **WXT (WebExtension Tools)** — Chrome/Firefox extension built with WXT (WebExtension Tools). Shares API client with the web app.

## Architecture

### Authentication

email_password, oauth_google

### Collaboration

single_user

### Data Model

#### User
Registered end-user of the app.

| Field | Type | Notes |
|:--|:--|:--|
| `id` | uuid |  |
| `email` | string | unique |
| `created_at` | timestamp |  |
| `digest_opt_in` | boolean |  |
| `tier` | enum | free \| pro \| lifetime |

#### Link
A saved URL with AI-generated summary.

| Field | Type | Notes |
|:--|:--|:--|
| `id` | uuid |  |
| `user_id` | relation | FK User |
| `url` | string |  |
| `title` | string |  |
| `description` | text |  |
| `favicon_url` | string |  |
| `summary` | text | AI-generated |
| `content_hash` | string |  |
| `created_at` | timestamp |  |
| `read_later` | boolean |  |
| `collection_id` | relation | optional FK Collection |

#### Tag
Reusable label applied to Links.

| Field | Type | Notes |
|:--|:--|:--|
| `id` | uuid |  |
| `user_id` | relation | FK User |
| `name` | string |  |
| `slug` | string |  |
| `source` | enum | ai \| manual |
| `usage_count` | integer | derived |

#### Collection
User-curated bundle of Links.

| Field | Type | Notes |
|:--|:--|:--|
| `id` | uuid |  |
| `user_id` | relation | FK User |
| `name` | string |  |
| `cover_link_id` | relation | optional FK Link |
| `sort_order` | integer |  |
| `created_at` | timestamp |  |

#### Digest
Weekly email snapshot of saved highlights.

| Field | Type | Notes |
|:--|:--|:--|
| `id` | uuid |  |
| `user_id` | relation | FK User |
| `week_start_date` | date |  |
| `sent_at` | timestamp |  |
| `link_ids` | array |  |

### Notifications

- **Welcome email** — Sent on sign-up. Includes a 3-step quickstart (install extension, save first link, try search) and a link back to the app.
- **Password reset email** — Transactional email triggered from the forgot-password flow. Contains a magic link valid for 30 minutes.
- **Weekly digest email** — Sent every Monday to users with digest_opt_in=true. Summarises unread saves from the past 7 days with AI highlights.
- **Payment receipt** — Transactional email sent after successful Pro/Lifetime purchase. Includes invoice link.

### Admin Panel

Minimal (view users/data)

### External Services

- **Google Gemini API** — Generates AI summaries and auto-tags for saved links.
- **Resend** — Sends weekly digest emails with AI-generated link highlights.
- **Stripe** — Handles Pro and Lifetime payment processing and subscription management.
- **Sentry** — Captures and tracks runtime errors in the web app and browser extension.
- **PostHog** — Product analytics — tracks feature usage and funnel events (privacy-safe, no PII).

## Market

### Audience Model

- **Type:** B2C (Individual consumers)
- **Launch scale:** 100–1k users

### Customers

- **Developers & Designers** — People who save tons of articles, docs, and tools and need a better way to find them later.
- **Knowledge Workers** — Researchers and writers who collect reference material and want AI-powered organization.
- **Content Creators** — Bloggers and newsletter writers who research online and need fast retrieval of reference material.

### Personas

- **Marcus, freelance full-stack dev** — Saves 20+ articles per week — tutorials, libraries, Hacker News threads. Forgets why he saved them by Friday. Wants to search by 'that React debounce trick' and actually find it. Chromebook + iPhone.
- **Priya, indie newsletter writer** — Collects research for a weekly tech newsletter. Needs to find sources she bookmarked 2-3 months ago and group them by theme. Writes on a MacBook, reads on an iPad.
- **Theo, product designer** — Saves visual inspiration, case studies, and Figma resources. Already uses Are.na for mood boards — wants one place for everything text-heavy. Mobile-first browsing.

### Competitors

- **Raindrop.io** — Popular bookmark manager with good organization. No AI features, no auto-tagging or summaries.
- **Are.na** — Visual research tool popular with designers. Great for mood boards but not ideal for link management.
- **Browser Bookmarks** — Built into every browser but impossible to search, organize, or find anything after a month.

## Business

### Revenue Model

- Freemium
- One-time Purchase

### Products

- **Free** — USD 0 / month
- **Pro** — USD 4.99 / month
- **Lifetime** — USD 49

### Constraints

**Scope:** MVP is web app only. Browser extension is post-launch. No mobile native app in v1.

**Timeline:** 2026-06-01

**Budget:** 500 USD


**Additional Constraints:**

- **Privacy First** — No analytics tracking or data sharing. All link content is private to the user. No social features.
- **Rate Limits** — AI summary generation must respect Gemini API rate limits. Queue processing for bulk imports.
