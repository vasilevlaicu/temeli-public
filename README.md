# Temeli - Simple, Safe Websites for Specific Communities

Temeli is a purpose-built website builder for community organizations (local groups, NGOs, etc.).

**Goal:** help non-technical maintainers publish clean, consistent websites that _can't_ go off the rails.

## Why Temeli (when Wix/WordPress exist)?

Over the last years I was contacted by **multiple communities of the same general type (niche)** asking for "a simple website that always looks clean."

Common pain points I kept hearing:

- **They want to be able to easily manage content** without worrying about technical stuff, plugins, settings pages etc.  
- **WordPress felt too technical** (hosting, themes, plugins, updates, security).    
- **Wix-like tools offer too much freedom** (layouts drift, inconsistent pages, or the site never ships).    
- **Editors aren't tech-savvy** and want to change _content_, mainly a few lines of text or pictures, not mess with structure or CSS.    

Usually they always ended up either paying someone to build a webapp or WordPress website, and then pay someone to maintain it.

So I started working on **Temeli** specifically for this niche:

- A **rigid, column based, content structure** (pages → sections → blocks) so sites **can't look bad**.    
- **Switchable style templates** (theme colors/typography/spacing) so every site stays on-brand without custom CSS.    
- **Community (niche) specific** templates.    
- **Exactly the blocks they need** (hero, schedule, gallery, staff, donations, community specific blocks, etc.) and nothing distracting.   

I started working on Temeli as a **direct response to repeated requests** from real communities and as a **passion project** to learn modern scalable platforms. Existing tools are either **too heavy/technical** (WordPress) or give **too much freedom** (Wix, etc.), causing non-technical maintainers to struggle, abandon sites, or create messy results. Temeli fills this gap with a **rigid content model** (ready-made multilingual blocks like heroes, galleries, calendars, donations) and **style templates** that guarantee clean, usable sites. By focusing on this **specific niche of communities**, Temeli delivers a **tailor-made solution**.

## 🎥 Demos with placeholder content

**Firebase prototype (single-tenant, first client)**  
Single theme prototype to test the concept and functionalities.  
![Firebase App Demo](docs/images/firebase_app.gif)

**Visual refactor for rewrite (multi-tenant, work-in-progress)**  
Visual refactor for a clean generalized layout with a few visual themes.  
![Rewrite App Demo](docs/images/azure_app.gif)

> ⚠️ The demos use **random placeholder community content** purely for testing.  
> They do **not** represent or reference the specific niche communities Temeli is built for.

## What editors can do

- Create pages made of **sections** and **blocks** (hero, rich text, image gallery, staff cards, calendar, donations, etc.).
    
- **Multilingual** content using i18n.
    
- **Media library** with folders, drag-drop upload, delete/replace.
    
- **Safe design system**: pick a **theme** (colors/typography/layout density). Content fits the template - no broken pages.
    
- **Preview → Publish**: draft safely, then publish the whole site in one click.
    

> Rigid content model + controlled theming = maintainers focus on content and don't create unusable layouts by accident.

## How it started (Firebase, single tenant)

The first version is a prototype which served a single community site with:

- **Vue 3** SPA.
    
- **Firebase Auth** (email link) for invited editors.
    
- **Cloud Firestore** for pages/sections/blocks with i18n fields, using offline persistence + cache-first reads for fast loads and limited network use.
    
- **Firebase Storage** for images; **download URLs** stored with captions/alt text cached by the browser/CDN.
    

It worked great for one customer and validated the editor UX concept.

## The Path to a Multi-Tenant SaaS on Firebase

The successful prototype proved the core concept, and multiple communities asked for the same solution. This called for evolving the project into a proper **SaaS**: multi-tenant, automated, and secure.

After evaluating other cloud platforms, the decision was made to build the SaaS version on **Firebase and Google Cloud**.

**Why Firebase for the multi-tenant SaaS?**

- **Developer Velocity:** Firebase's tightly integrated toolkit (Auth, Firestore, Functions, Hosting) dramatically reduces boilerplate and accelerates feature development.
    
- **Integrated Security Model:** Firestore Security Rules provide a powerful, declarative way to enforce complex, tenant-based data isolation directly at the database layer. This is simpler and more robust than writing authorization checks in backend code.
    
- **Low Operational Overhead:** As a fully serverless platform, Firebase lets us focus on building the application, not managing infrastructure.
    
- **Scalable and Cost-Effective:** The pay-as-you-go pricing model, combined with a generous free tier, is ideal for a startup, scaling smoothly from zero to millions of users.
    

## The Firebase SaaS Architecture

### Core architecture (simplified)

- **Identity** → **Firebase Authentication**: Secure sign-in (email, social logins) with a simple, powerful SDK.
    
- **Database & Authorization** → **Cloud Firestore**: A NoSQL database where all data is stored. **Firestore Security Rules** enforce that a user can _only_ ever access data for the sites they are a collaborator on.
    
- **Storage** → **Cloud Storage for Firebase**: Holds user media and the generated static site bundles. Also protected by robust security rules.
    
- **Backend** → **Cloud Functions for Firebase**: Serverless functions for backend logic like processing Stripe payments, automating site provisioning, and running the SSG publish pipeline.
    
- **Hosting & CDN** → **Firebase Hosting**: A global CDN that serves all the application frontends (dashboard, editor) and the public-facing static websites.
    

### The Publishing Pipeline on Firebase

The core principle of atomic, versioned deploys is implemented elegantly within the Firebase ecosystem:

1. **Dashboard & Editor**: A user signs up and manages their sites at `app.temeli.com`. The editor for their specific site is a Nuxt/Vue SPA hosted at `{slug}--edit.web.app`. Both are served by Firebase Hosting.
    
2. **Publish Trigger**: When an editor clicks "Publish", the app calls a secure Cloud Function.
    
3. **Static Site Generation (SSG)**: The Cloud Function fetches the site's content from Firestore, runs the SSG build process, and uploads the resulting static files (HTML, CSS, JS) to a versioned folder in **Cloud Storage**.
    
4. **Atomic Pointer Swap**: The function then updates a tiny `current.json` file in Cloud Storage to point to the new version's folder. This makes the release instant and rollbacks trivial.
    
5. **Serving the Public Site**: The public site at `{slug}.web.app` (or a custom domain) is served by **Firebase Hosting**. A simple rewrite rule directs all incoming requests to a lightweight `serveSite` Cloud Function. This function reads the `current.json` pointer and streams the correct static file from Cloud Storage to the user, ensuring fast, safe, and SEO-friendly pages.
    

This integrated approach provides all the power of the original design with significantly less complexity and faster development speed.

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
    

Themes control spacing, fonts, and colors, _not_ content structure.

Users can drag and drop sections and blocks to rearrange content, and change column layout of sections (1, 2, 3 columns).
