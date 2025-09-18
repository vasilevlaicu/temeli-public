# Temeli - Simple, Safe Websites for Specific Communities

Temeli is a purpose-built website builder for community organizations (local groups, NGOs, etc.).  
**Goal:** help non-technical maintainers publish clean, consistent websites that *can't* go off the rails.

---

## Why Temeli (when Wix/WordPress exist)?

Over the last years I was contacted by **multiple communities of the same general type (niche)** asking for "a simple website that always looks clean."

Common pain points I kept hearing:

- **They want to be able to easily manage content** without worrying about technical stuff, plugins, settings pages etc.
- **WordPress felt too technical** (hosting, themes, plugins, updates, security).  
- **Wix-like tools offer too much freedom** (layouts drift, inconsistent pages, or the site never ships).  
- **Editors aren't tech-savvy** and want to change *content*, mainly a few lines of text or pictures, not mess with structure or CSS.

Usually they always ended up either paying someone to build a webapp or WordPress website, and then pay someone to maintain it.

So I started working on **Temeli** specifically for this niche:

- A **rigid, column based, content structure** (pages → sections → blocks) so sites **can't look bad**.  
- **Switchable style templates** (theme colors/typography/spacing) so every site stays on-brand without custom CSS.  
- **Community (niche) specific** templates.
- **Exactly the blocks they need** (hero, schedule, gallery, staff, donations, community specific blocks, etc.) and nothing distracting.


I started working on Temeli as a **direct response to repeated requests** from real communities and as a **passion project** to learn modern scalable platforms. Existing tools are either **too heavy/technical** (WordPress) or give **too much freedom** (Wix, etc.), causing non-technical maintainers to struggle, abandon sites, or create messy results. Temeli fills this gap with a **rigid content model** (ready-made multilingual blocks like heroes, galleries, calendars, donations) and **style templates** that guarantee clean, usable sites. By focusing on this **specific niche of communities**, Temeli delivers a **tailor-made solution**.

---

## 🎥 Demos with placeholder content

**Firebase prototype (single-tenant, first client)**  
Single theme prototype to test the concept and functionalities.  
![Firebase App Demo](docs/images/firebase_app.gif)

**Visual refactor for Azure rewrite (multi-tenant, work-in-progress)**  
Visual refactor for a clean generalized layout with a few visual themes.  
![Azure App Demo](docs/images/azure_app.gif)

> ⚠️ The demos use **random placeholder community content** purely for testing.  
> They do **not** represent or reference the specific niche communities Temeli is built for.

---

## What editors can do

- Create pages made of **sections** and **blocks** (hero, rich text, image gallery, staff cards, calendar, donations, etc.).  
- **Multilingual** content using i18n.  
- **Media library** with folders, drag-drop upload, delete/replace.  
- **Safe design system**: pick a **theme** (colors/typography/layout density). Content fits the template - no broken pages.  
- **Preview → Publish**: draft safely, then publish the whole site in one click.

> Rigid content model + controlled theming = maintainers focus on content and don't create unusable layouts by accident.

---

## How it started (Firebase, single tenant)

The first version is a prototype which served a single community site with:
- **Vue 3** SPA.
- **Firebase Auth** (email link) for invited editors.
- **Cloud Firestore** for pages/sections/blocks with i18n fields, using offline persistence + cache-first reads for fast loads and limited network use.
- **Firebase Storage** for images; **download URLs** stored with captions/alt text cached by the browser/CDN.

It worked great for one customer and validated the editor UX concept.

---

## Why I'm moving to Azure

Multiple communities asked for **the same editor**, **the same niche content blocks/templates**, and **the same guardrails**.
This calls for a proper **SaaS**: multi-tenant, with reliable publishing and predictable costs.

**High-level Azure design (work in progress):**

* **Dashboard**: temeli.com → where users sign up, manage plans (Stripe), invite collaborators, check site metrics, and deploy/manage their sites.
* **Editor host**: `{slug}--edit.temeli.app` → authenticated Nuxt/Vue editor. Drafts are saved directly in the DB.
* **Publish pipeline**: on publish, an Azure Function runs **Static Site Generation (SSG)**, uploads a **versioned static bundle** to Blob Storage, then updates a tiny `current.json` pointer for instant release or rollback.
* **Public site**: `{slug}.temeli.app` or a custom domain → serves **immutable static files via CDN**, ensuring fast, safe, and SEO-friendly pages.
* **Core Azure services**: Blob Storage (media + bundles), Functions (API/build), Cosmos DB (content), API Management (auth/rate-limits), Application Insights (telemetry).
* **Edge routing**: a Cloudflare Worker checks the requested domain, finds the right tenant and its active published version, and then serves the correct static HTML and assets from storage/CDN.

**Why Azure over Firebase for this phase?**
Azure is a better fit for **multi-tenant SSG publishing**, **versioned static bundles with atomic rollbacks**, and **role-based access** across many sites.

---

## WIP, Deployment (Azure)

### Core architecture (simplified)

* **Identity** → Microsoft Entra External ID: secure sign-in (email + social logins, Google enabled now).
* **Database** → Cosmos DB (serverless NoSQL): JSON docs for users, sites, pages, files.
* **Storage** → Azure Blob Storage: holds user media and the generated static bundles.
* **Backend** → Azure Functions: serverless API + background jobs (pay-per-execution).
* **Editor** → Azure Static Web Apps: serves the Nuxt/Vue editor (admins only).

(Draft idea for serving the client websites:)
* **Public sites** → Published pages served as static files, versioned and immutable.
* **Edge routing** → Cloudflare Worker:
  * Maps domain → tenant and active version.
  * Fetches HTML via `current.json` pointer (short cache + SWR).
  * Serves assets directly from long-term cache (no repeated origin cost).

I believe it's essential to serve static files and to not keep an SPA Vue page that dynamically fetches data from a database for each client website if we want proper SEO optimization.

---

## 🧩 Content model (pages → sections → blocks)

Built-in blocks (selected) examples:
- **Hero / Heading / Rich text (sanitized)**
- **Image gallery** (captions, alt text, folders)
- **Staff cards** (photo, name, role, bio)
- **Calendar / Schedule**
- **Donation call-to-action** (embeds supported)
- **Video & social embeds** (YouTube, Vimeo, Facebook live)
- **Map / Contact info**
- **Other custom niche blocks for communities**

Themes control spacing, fonts, and colors, *not* content structure.

Users can drag and drop sections and blocks to rearrange content, and change column layout of sections (1, 2, 3 columns).

---