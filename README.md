# Design to Prototype Workflow

## Figma / Cursor / Astro / GitHub Pages

> A step-by-step guide for product design teams to build real, shareable prototypes — no deep engineering skills required. If you can use Figma, you can do this.

---

## Table of Contents

1. [Architecture Overview](#1-architecture-overview)
2. [Concepts for Designers — A Friendly Translation](#2-concepts-for-designers--a-friendly-translation)
3. [One-Time Setup](#3-one-time-setup)
4. [Workflow: From Figma to Live Prototype](#4-workflow-from-figma-to-live-prototype)
5. [Figma Auto Layout to Tailwind — A Translation Guide](#5-figma-auto-layout-to-tailwind--a-translation-guide)
6. [Prompt Library](#6-prompt-library)
7. [Scaling & Consistency](#7-scaling--consistency)
8. [Refinement Loop](#8-refinement-loop)
9. [Tips, Tricks & Pitfalls](#9-tips-tricks--pitfalls)

---

## 1. Architecture Overview

```
┌─────────────┐      ┌─────────────┐      ┌─────────────┐      ┌─────────────┐
│   FIGMA     │ ──▶  │   CURSOR    │ ──▶  │   GITHUB    │ ──▶  │  GH PAGES   │
│ (Design)    │      │ (Code Gen)  │      │ (Version    │      │ (Live       │
│             │      │             │      │  Control)   │      │  Prototype) │
│ Components  │      │ Astro +     │      │ Branches    │      │ Shareable   │
│ Tokens      │      │ Tailwind +  │      │ PRs         │      │ URL         │
│ Layouts     │      │ shadcn/ui   │      │ Reviews     │      │             │
└─────────────┘      └─────────────┘      └─────────────┘      └─────────────┘
     ▲                                                                │
     └────────────── Feedback Loop ◀──────────────────────────────────┘
```

**Why this stack?**

| Tool | Role | Design-Team Benefit |
|------|------|---------------------|
| **Figma** | Source of truth for design | Your designs stay in charge — code follows Figma, not the other way around |
| **Figma MVP Plugin** | Exports Figma → code scaffolds | One-click bridge from visual design to code — no hand-coding |
| **Cursor** | AI-powered code editor | Describe what you want in plain language, Cursor writes the code |
| **Astro** | Static site framework | Builds fast, component-based pages — perfect for prototypes |
| **TailwindCSS** | Utility-first CSS | Think of it as "Figma properties but in code" (color, padding, gap — all inline) |
| **shadcn/ui** | UI component library | Pre-built, polished buttons, cards, inputs — like a code-based UI Kit |
| **GitHub** | Version control & collaboration | Think "Figma Version History" but for code, with team review built in |
| **GitHub Pages** | Hosting | Free, automatic deploys — every merge = a live, shareable URL |

---

## 2. Concepts for Designers — A Friendly Translation

If Git and terminal commands feel alien, this section is for you. Every concept below has a Figma equivalent you already know.

| Engineering Concept | Figma Equivalent | What It Actually Means |
|---------------------|-----------------|----------------------|
| **Repository (Repo)** | Your Figma file | The folder that holds your entire project |
| **Git Commit** | "Save a named version" in Version History | A snapshot of your work with a message like "Add hero section" |
| **Branch** | A duplicate page in Figma to try ideas | A parallel copy of the project where you make changes without touching the original |
| **Main branch** | Your "final" Figma page | The live, approved version of the prototype |
| **Pull Request (PR)** | "Hey team, review my page before I merge it" | A request to merge your branch changes into `main`, with a review step |
| **Merge** | Moving your approved designs back to the main page | Combining your branch work into the main project |
| **Deploy** | Publishing / sharing a Figma prototype link | Making the latest version live at a public URL |
| **Terminal** | Quick Actions / Command Palette | A text-based way to run commands (like copy-pasting recipe steps) |
| **`npm run dev`** | Clicking "Play" on a prototype | Starts a local preview so you can see your page in the browser |
| **`client:load`** | Making a component interactive in a prototype | Tells Astro "this component needs JavaScript to work" (buttons, tabs, etc.) |

> **Encouragement:** You do not need to memorize these. Bookmark this table and refer back. Over time, these terms become second nature — just like layers, frames, and auto layout did in Figma.

---

## 3. One-Time Setup

> **Who does this:** One tech-savvy team member (the "Setup Lead") does this once. Everyone else just clones the repo and starts working.

### Step 3.1 — Install Prerequisites

Install these four tools on your machine:

1. **Node.js** (v20 or newer): [nodejs.org](https://nodejs.org) — this is the engine that runs your project
2. **Cursor**: [cursor.com](https://cursor.com) — your AI-powered code editor
3. **Git**: [git-scm.com](https://git-scm.com) — version control (comes pre-installed on most Macs)
4. **GitHub account**: [github.com](https://github.com) — where your project lives online
5. **Figma MVP plugin**: Install from Figma Community — bridges your designs to code

> **Tip:** To check if Node.js and Git are already installed, open Cursor's terminal (`Ctrl+`` ` or `Cmd+`` `) and type `node -v` and `git -v`. If you see version numbers, you're good.

### Step 3.2 — Create the Astro Project

Open Cursor's terminal and run these commands **one group at a time** (copy-paste each block):

```bash
# Step A: Create a new Astro project with the minimal template
npm create astro@latest prototype -- --template minimal --yes
```

```bash
# Step B: Move into your new project folder
cd prototype
```

```bash
# Step C: Add Tailwind CSS (for styling, like Figma's design properties)
npx astro add tailwind --yes
```

```bash
# Step D: Add React support (required for shadcn/ui interactive components)
npx astro add react --yes
```

```bash
# Step E: Set up shadcn/ui (the pre-built component kit)
npx shadcn@latest init
```

During `shadcn init`, you'll be asked a few questions. Choose:
- Style: **Default**
- Base color: **Slate** (or whichever matches your Figma tokens)
- CSS variables: **Yes**

> **Warning — "shadcn" not "shadcn-ui":** The correct CLI command is `npx shadcn@latest init`. If you see `shadcn-ui` in old tutorials, that's outdated and will fail.

### Step 3.3 — Project Structure

After setup, your project looks like this:

```
prototype/
├── src/
│   ├── components/           ← Reusable UI pieces (like Figma components)
│   │   ├── ui/               ← shadcn/ui components (Button, Card, etc.)
│   │   └── custom/           ← Your Figma-matched components
│   ├── layouts/
│   │   └── BaseLayout.astro  ← The shared page wrapper (like a Figma "Template" frame)
│   ├── pages/
│   │   ├── index.astro       ← Homepage (each .astro file = one page)
│   │   └── [...other pages]
│   └── styles/
│       └── tokens.css        ← Design tokens from Figma (colors, spacing, fonts)
├── public/                   ← Static files (images, icons, favicons)
├── astro.config.mjs          ← Project settings (important for deployment)
├── tailwind.config.mjs       ← Tailwind customization (your design tokens go here)
└── package.json              ← List of installed tools/dependencies
```

**Think of it this way:**
- `pages/` = Figma pages
- `components/` = Figma component library
- `layouts/` = Figma template frames
- `tokens.css` + `tailwind.config.mjs` = Figma design tokens / local styles

### Step 3.4 — Create the Base Layout

Use this prompt in Cursor (open Chat with `Cmd+L` or `Ctrl+L`):

````xml
<prompt>
  <role>You are a senior frontend developer building an Astro prototype.</role>
  <task>Create a BaseLayout.astro component in src/layouts/ that includes:
    - HTML5 boilerplate with proper meta tags
    - A slot for page content
    - Import of global styles from ../styles/tokens.css
    - Responsive viewport meta tag
    - A configurable page title via Astro.props
  </task>
  <stack>Astro, TailwindCSS</stack>
  <constraints>
    - Keep it minimal and clean
    - Use Astro's built-in slot mechanism
    - Add a "prototype-banner" div at the top with text "PROTOTYPE" 
      on a yellow background, so stakeholders know it's not production
    - All internal links must use import.meta.env.BASE_URL as prefix
      (this prevents broken links when deployed to GitHub Pages)
  </constraints>
</prompt>
````

### Step 3.5 — Set Up GitHub Repository & Pages

> **Think of this step as:** Creating a shared Figma file on your team's workspace, so everyone has access and changes sync automatically.

**Option A — Using GitHub CLI (faster):**

```bash
git init
git add .
git commit -m "Initial prototype setup"

# Create the repo and push in one command
gh repo create your-org/prototype --public --source=. --push
```

**Option B — Using github.com (visual):**

```bash
git init
git add .
git commit -m "Initial prototype setup"

# Go to github.com → New Repository → copy the URL, then:
git remote add origin https://github.com/your-username/prototype.git
git branch -M main
git push -u origin main
```

**Then, create the deploy workflow.** Use this prompt in Cursor:

````xml
<prompt>
  <task>Set up GitHub Actions for Astro deployment to GitHub Pages.</task>
  <context>I want to deploy this Astro project to GitHub Pages automatically 
    every time I push to main.</context>
  <instructions>
    1. Create the folder .github/workflows/ and a file deploy.yml inside it.
    2. Use the official Astro deployment action template.
    3. Update astro.config.mjs to include BOTH the 'site' and 'base' properties. 
       The 'site' should be 'https://YOUR-ORG.github.io' and the 'base' should 
       be '/YOUR-REPO-NAME'.
    4. Make sure the React integration is imported and included.
  </instructions>
</prompt>
````

**...or create the files manually:**

The deploy workflow:

```yaml
# .github/workflows/deploy.yml
name: Deploy Prototype to GitHub Pages

on:
  push:
    branches: [main]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
      - run: npm ci
      - run: npm run build
      - uses: actions/upload-pages-artifact@v3
        with:
          path: dist/

  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - id: deployment
        uses: actions/deploy-pages@v4
```

The Astro config (**pay close attention — this is where most deploy bugs come from**):

```javascript
// astro.config.mjs
import { defineConfig } from 'astro/config';
import tailwind from '@astrojs/tailwind';
import react from '@astrojs/react';

export default defineConfig({
  integrations: [tailwind(), react()],
  site: 'https://your-org.github.io',
  base: '/prototype',
});
```

> **Warning — Replace the placeholders:** You MUST change `your-org` and `prototype` to your actual GitHub org/username and repo name. If these don't match, your deploy will show a blank page or 404.

Finally, enable GitHub Pages in your repo settings:
1. Go to your repo on GitHub → **Settings** → **Pages**
2. Under "Build and deployment", set Source to **GitHub Actions**

### Common Mistakes — Setup Phase

| Mistake | Symptom | Fix |
|---------|---------|-----|
| Forgot `npx astro add react --yes` | shadcn/ui components don't render, blank spots on page | Run `npx astro add react --yes` in your terminal |
| Used `npx shadcn-ui@latest init` (outdated) | Command not found or install errors | Use `npx shadcn@latest init` instead |
| Forgot to include `react()` in astro.config.mjs | React components silently fail | Add `import react from '@astrojs/react';` and add `react()` to the integrations array |
| Wrong `site` or `base` in astro.config.mjs | Blank page after deploy, broken CSS, 404 errors | Make sure `site` is `https://YOUR-ORG.github.io` and `base` is `/YOUR-REPO-NAME` |
| Didn't set GitHub Pages source to "GitHub Actions" | Deploys run but site doesn't update | Go to repo Settings → Pages → set Source to "GitHub Actions" |

---

## 4. Workflow: From Figma to Live Prototype

### The 7-Step Cycle

```
 ┌──────────────────────────────────────────────────────────────┐
 │                    THE DESIGN-TO-CODE CYCLE                  │
 │                                                              │
 │  ① Extract    ② Export     ③ Generate   ④ Refine             │
 │  Tokens  ──▶  Component ─▶  Code   ──▶  in     ─────┐        │
 │  (Figma)      (Figma MVP)   (Cursor)    (Cursor)    │        │
 │                                                     ▼        │
 │  ⑦ Share   ◀── ⑥ Deploy  ◀── ⑤ Push                 │        │
 │  (URL)         (GH Pages)    (GitHub)  ◀────────────┘        │
 │                                                              │
 └──────────────────────────────────────────────────────────────┘
```

---

### Step 4.1 — Extract Design Tokens from Figma

> **Who:** Designer  
> **Where:** Figma  
> **Think of it as:** Documenting your local styles so code can use the exact same values.

In Figma, gather your design tokens — colors, typography, spacing, border radius, shadows. Then use this Cursor prompt to turn them into code:

````xml
<prompt>
  <role>You are a senior frontend developer converting Figma design tokens 
    to CSS custom properties and Tailwind config.</role>
  <task>Create two files:
    1. src/styles/tokens.css — CSS custom properties for all tokens
    2. Update tailwind.config.mjs — extend the theme with these tokens
  </task>
  <tokens>
    <colors>
      <!-- REPLACE THESE with your actual Figma color tokens -->
      <primary>#2563EB</primary>
      <primary-hover>#1D4ED8</primary-hover>
      <secondary>#7C3AED</secondary>
      <background>#FFFFFF</background>
      <surface>#F8FAFC</surface>
      <text-primary>#0F172A</text-primary>
      <text-secondary>#64748B</text-secondary>
      <border>#E2E8F0</border>
      <error>#DC2626</error>
      <success>#16A34A</success>
    </colors>
    <typography>
      <font-family>Inter, system-ui, sans-serif</font-family>
      <heading-1>2.25rem/2.5rem, 700</heading-1>
      <heading-2>1.875rem/2.25rem, 600</heading-2>
      <heading-3>1.5rem/2rem, 600</heading-3>
      <body>1rem/1.5rem, 400</body>
      <caption>0.875rem/1.25rem, 400</caption>
    </typography>
    <spacing>
      <xs>4px</xs>
      <sm>8px</sm>
      <md>16px</md>
      <lg>24px</lg>
      <xl>32px</xl>
      <2xl>48px</2xl>
    </spacing>
    <radii>
      <sm>4px</sm>
      <md>8px</md>
      <lg>12px</lg>
      <full>9999px</full>
    </radii>
  </tokens>
  <constraints>
    - Map tokens to shadcn/ui CSS variable naming conventions
    - Ensure dark mode support structure (even if not used yet)
    - Add comments grouping tokens by category
  </constraints>
</prompt>
````

> **Tip:** Replace every value above with YOUR actual Figma values. Open Figma's Inspect panel on any element to see the exact hex colors, font sizes, and spacing values.

> **Why this matters:** This is the single most important step. If your tokens are right, everything downstream — every component, every page — will automatically use the correct colors, fonts, and spacing. Figma stays the source of truth.

---

### Step 4.2 — Export Component from Figma MVP

> **Who:** Designer  
> **Where:** Figma + Figma MVP Plugin  
> **Think of it as:** "Copy as code" — similar to Figma's "Copy as CSS" but for full components.

1. Select the component or frame in Figma
2. Open the **Figma MVP** plugin
3. Export as **HTML/CSS** or **React** (either format works — Cursor will convert it)
4. Copy the generated code to your clipboard

> **Important:** The exported code is a **rough draft**, not the final result. Think of it as a wireframe-level code scaffold. Cursor will clean it up and make it production-quality in the next step.

> **Tip:** For best results, make sure your Figma component uses **Auto Layout** and has **clearly named layers**. The plugin uses layer names in the generated code, so "hero-title" is much more helpful than "Frame 427".

---

### Step 4.3 — Generate Astro Component in Cursor

> **Who:** Anyone on the team  
> **Where:** Cursor  
> **Think of it as:** Telling a developer assistant exactly what you want, and they write the code.

Paste the Figma MVP export into Cursor and use this prompt:

````xml
<prompt>
  <role>You are a senior frontend developer converting a Figma MVP export 
    into a clean Astro component.</role>
  <task>Convert the following Figma MVP export into a reusable Astro component 
    using TailwindCSS utility classes and shadcn/ui where applicable.</task>
  <figma-export>
    <!-- PASTE YOUR FIGMA MVP EXPORT HERE -->
  </figma-export>
  <requirements>
    - Use TailwindCSS utility classes (no inline styles, no custom CSS)
    - Replace any matching UI patterns with shadcn/ui components 
      (Button, Card, Input, etc.)
    - Make the component accept props for dynamic content
    - Use semantic HTML elements
    - Ensure responsive behavior (mobile-first)
    - Use design tokens from our tailwind.config.mjs 
      (e.g., "bg-primary", "text-text-primary")
    - Add TypeScript interface for props
    - If any part of this component is interactive (tabs, toggles, 
      dropdowns, forms), use a React component with client:load directive
  </requirements>
  <file-location>src/components/custom/[ComponentName].astro</file-location>
  <constraints>
    - Do NOT use any CSS modules or styled-components
    - Do NOT hardcode text content — use props
    - DO use Astro's component syntax (.astro files) for static parts
    - DO use React (.tsx files) with client:load for interactive parts
    - Match the visual design as closely as possible
  </constraints>
</prompt>
````

> **Warning — The `client:load` trap:** This is the **#1 hidden mistake** new Astro users make. If your component has ANY interactivity (clicking, typing, hovering effects, tabs, dropdowns), the React component **must** have a `client:load` or `client:visible` directive. Without it, the component renders as static HTML and all JavaScript is silently stripped out. Example:

```astro
---
// In your .astro file:
import { Tabs } from '../components/ui/Tabs';
---

<!-- WRONG: Tabs won't work. Clicks do nothing. -->
<Tabs items={tabData} />

<!-- CORRECT: client:load makes the component interactive -->
<Tabs client:load items={tabData} />
```

---

### Step 4.4 — Refine the Component

> **Who:** Anyone on the team  
> **Where:** Cursor  
> **Think of it as:** Doing a design QA pass — comparing the browser to Figma and listing every difference.

Open your prototype in the browser (`npm run dev`), put it side-by-side with Figma, and note every difference. Then use this prompt:

````xml
<prompt>
  <role>You are a pixel-perfect frontend developer refining a prototype 
    component.</role>
  <task>Refine this Astro component to match the Figma design more 
    closely.</task>
  <current-component>
    <!-- PASTE CURRENT COMPONENT CODE or reference the file with @filename -->
  </current-component>
  <differences>
    <!-- LIST WHAT LOOKS DIFFERENT — be specific with values: -->
    - The spacing between the title and description should be 12px, not 16px
    - The button border-radius should be 8px (rounded-lg)
    - The card shadow is too strong — use shadow-sm instead of shadow-md
    - The font weight on the subtitle should be 500 (font-medium)
    - The icon is 20px, should be 24px
  </differences>
  <constraints>
    - Only modify what's listed in differences
    - Keep all existing props and structure
    - Use Tailwind classes only
  </constraints>
</prompt>
````

> **Tip:** The more specific you are about the differences, the better the result. "It looks off" is vague. "The gap between the image and the title should be 16px (gap-4), currently it's 24px (gap-6)" is perfect.

---

### Step 4.5 — Build a Page

> **Who:** Anyone on the team  
> **Where:** Cursor

````xml
<prompt>
  <role>You are a senior frontend developer assembling a prototype page 
    from components.</role>
  <task>Create a new Astro page at src/pages/[page-name].astro that 
    composes the following components into a complete layout.</task>
  <layout>Use BaseLayout from ../layouts/BaseLayout.astro</layout>
  <page-structure>
    <!-- DESCRIBE THE PAGE STRUCTURE FROM YOUR FIGMA DESIGN: -->
    1. Navigation bar (src/components/custom/Navbar.astro)
    2. Hero section with headline, subheadline, and CTA button
    3. Three-column feature cards (src/components/custom/FeatureCard.astro)
    4. Testimonial section
    5. Footer (src/components/custom/Footer.astro)
  </page-structure>
  <content>
    <!-- PASTE OR DESCRIBE THE ACTUAL CONTENT from Figma: -->
    - Headline: "Build better products"
    - Subheadline: "The fastest way to go from idea to prototype"
    - CTA: "Get Started" (links to /get-started)
  </content>
  <constraints>
    - Use existing components — do not recreate them
    - Use Tailwind for layout (flex, grid, spacing)
    - Match Figma's spacing and layout precisely
    - Ensure the page is responsive
    - All internal links must be prefixed with import.meta.env.BASE_URL
    - Any interactive components must use client:load directive
  </constraints>
</prompt>
````

> **Warning — Internal Links:** When linking between pages, always use `import.meta.env.BASE_URL` as a prefix. Otherwise your links will break on GitHub Pages.

```astro
<!-- WRONG: Works locally, breaks on GitHub Pages -->
<a href="/about">About</a>

<!-- CORRECT: Works everywhere -->
<a href={`${import.meta.env.BASE_URL}about`}>About</a>
```

---

### Step 4.6 — Push to GitHub

> **Who:** Anyone on the team  
> **Where:** Cursor's terminal  
> **Think of it as:** Saving a new named version and uploading it for team review — like updating a shared Figma file.

```bash
# Step A: Get the latest version (like refreshing a shared Figma file)
git checkout main
git pull origin main

# Step B: Create your own working copy (like duplicating a Figma page to experiment)
git checkout -b feature/page-name

# Step C: Save a named version (like "Version: Added hero section" in Figma history)
git add .
git commit -m "Add [page-name] page with [components]"

# Step D: Upload to GitHub (like publishing your Figma changes for the team)
git push -u origin feature/page-name
```

Then go to GitHub and **create a Pull Request** — this is like saying "Hey team, I made some changes. Take a look before we make them live."

### Common Mistakes — Build & Push Phase

| Mistake | Symptom | Fix |
|---------|---------|-----|
| Forgot `client:load` on an interactive component | Buttons don't click, tabs don't switch, forms don't work | Add `client:load` to the component tag: `<MyComponent client:load />` |
| Used `<a href="/about">` without base URL | Links work locally but 404 on GitHub Pages | Use `` <a href={`${import.meta.env.BASE_URL}about`}> `` |
| Images referenced as `/image.png` | Images load locally but break on GitHub Pages | Use `` `${import.meta.env.BASE_URL}image.png` `` or put images in `public/` and reference them correctly |
| Pushed directly to `main` | Untested changes go live immediately | Always create a branch first: `git checkout -b feature/my-change` |
| Commit message is vague ("updates") | Hard to find or undo specific changes later | Use descriptive messages: "Add pricing page with comparison table" |

---

### Step 4.7 — Deploy & Share

> **Think of it as:** Clicking "Share Prototype" in Figma — but this time, it's a real website.

1. On GitHub, open your Pull Request
2. Have a teammate review it (or review it yourself for solo projects)
3. Click **"Merge pull request"**
4. GitHub Actions automatically builds and deploys your site

Your prototype is live at:

```
https://your-org.github.io/prototype/
```

Share this URL with stakeholders, user researchers, or anyone who needs to experience the prototype. It works on any device with a browser — no Figma account needed.

---

## 5. Figma Auto Layout to Tailwind — A Translation Guide

This is the bridge between your Figma skills and code. **If you understand Auto Layout, you already understand 80% of Tailwind layout.**

### Direction

| Figma Auto Layout | Tailwind CSS | What It Does |
|-------------------|-------------|-------------|
| Direction: **Horizontal** | `flex flex-row` | Items sit side by side (→) |
| Direction: **Vertical** | `flex flex-col` | Items stack top to bottom (↓) |
| Wrap | `flex flex-wrap` | Items wrap to next line when they don't fit |

### Spacing

| Figma Auto Layout | Tailwind CSS | Value |
|-------------------|-------------|-------|
| Gap: **8px** | `gap-2` | Space between items |
| Gap: **16px** | `gap-4` | Space between items |
| Gap: **24px** | `gap-6` | Space between items |
| Padding: **16px** all sides | `p-4` | Inner spacing |
| Padding: **16px** horizontal, **24px** vertical | `px-4 py-6` | Directional inner spacing |

> **Tip — the "4px = 1 unit" rule:** Tailwind spacing uses a 4px base unit. Divide your Figma value by 4 to get the Tailwind number. 8px = `2`, 16px = `4`, 24px = `6`, 32px = `8`, 48px = `12`.

### Alignment

| Figma Auto Layout | Tailwind CSS | Visual |
|-------------------|-------------|--------|
| Align items: **Top Left** (horizontal) | `flex items-start justify-start` | Items at start |
| Align items: **Center** | `flex items-center justify-center` | Items centered |
| Align items: **Space Between** | `flex justify-between` | Items spread across full width |
| Align items: **Stretch** (fill container) | `flex items-stretch` | Items fill the container height |

### Sizing

| Figma Property | Tailwind CSS | What It Does |
|---------------|-------------|-------------|
| Width: **Fixed 200px** | `w-[200px]` or `w-52` | Exact width |
| Width: **Fill Container** | `w-full` or `flex-1` | Grow to fill available space |
| Width: **Hug Contents** | `w-fit` | Shrink to fit content |
| Min Width: **100px** | `min-w-[100px]` | Won't get smaller than this |
| Max Width: **600px** | `max-w-[600px]` or `max-w-xl` | Won't get larger than this |

### A Complete Example

**In Figma**, you might have a card component with:
- Vertical Auto Layout, gap 16px, padding 24px
- Hug contents horizontally, fill container vertically
- Rounded corners 12px, shadow

**In Tailwind**, that becomes:

```html
<div class="flex flex-col gap-4 p-6 w-fit h-full rounded-lg shadow-md">
  <!-- Card content here -->
</div>
```

The mapping is almost 1:1 once you learn the naming pattern.

---

## 6. Prompt Library

A collection of copy-paste prompts for common tasks. Replace the placeholder content with your actual design details.

### 6.1 — Adding a shadcn/ui Component

````xml
<prompt>
  <role>You are a frontend developer adding shadcn/ui components to an 
    Astro project with React integration.</role>
  <task>Add the following shadcn/ui component(s) to the project and create 
    an Astro wrapper component that matches our design tokens.</task>
  <components>
    <!-- LIST COMPONENTS NEEDED, e.g.: -->
    - Button (with variants: primary, secondary, outline, ghost)
    - Card (with CardHeader, CardContent, CardFooter)
    - Input (with label and error state)
    - Dialog (modal)
  </components>
  <instructions>
    1. Install each shadcn/ui component: npx shadcn@latest add [component-name]
    2. Create an Astro wrapper in src/components/ui/ if needed
    3. Apply our design tokens (colors, radii, typography)
    4. Show usage examples with and without client:load
    5. Clearly indicate which components REQUIRE client:load (any with 
       interactivity: Dialog, Tabs, Accordion, DropdownMenu, etc.)
  </instructions>
  <constraints>
    - Use our existing design tokens from tailwind.config.mjs
    - Maintain accessibility (ARIA attributes, keyboard navigation)
    - Keep the API simple for non-technical team members
    - IMPORTANT: shadcn/ui components are React components — they need
      client:load when they have interactive behavior
  </constraints>
</prompt>
````

> **Quick reference — Which shadcn/ui components need `client:load`?**
> 
> | Needs `client:load` (interactive) | Does NOT need it (static display) |
> |---|---|
> | Dialog, Sheet, Drawer | Card, CardHeader, CardContent |
> | Tabs, Accordion | Badge, Separator |
> | DropdownMenu, Select | Avatar, AspectRatio |
> | Tooltip, Popover | Alert, Table |
> | Toggle, Switch | Skeleton |
> | Form, Input (with validation) | Input (display only) |

### 6.2 — Creating Interactive Prototype Elements

````xml
<prompt>
  <role>You are a frontend developer adding interactivity to an Astro 
    prototype.</role>
  <task>Add click-through prototype behavior to the current page.</task>
  <interactions>
    <!-- DESCRIBE INTERACTIONS, e.g.: -->
    - Clicking "Get Started" button navigates to /onboarding
    - Clicking a navigation item highlights it and scrolls to the section
    - The mobile menu toggles open/closed on hamburger click
    - Tabs switch content panels without page reload
    - Form inputs show validation states (but don't submit anywhere)
  </interactions>
  <constraints>
    - Use Astro's page routing for page-to-page navigation
    - Use React components with client:load for in-page interactivity 
      (tabs, toggles, mobile menus)
    - No backend/API calls — this is a prototype
    - Keep interactions simple and demonstrative
    - All navigation links must use import.meta.env.BASE_URL as prefix
  </constraints>
</prompt>
````

### 6.3 — Creating a New Page from Figma Screenshot

````xml
<prompt>
  <role>You are a pixel-perfect frontend developer.</role>
  <task>Look at the attached Figma screenshot and recreate this page as an 
    Astro page using TailwindCSS and shadcn/ui.</task>
  <image>
    <!-- ATTACH SCREENSHOT IN CURSOR (drag and drop into chat) or 
         describe the layout in detail -->
  </image>
  <existing-components>
    <!-- LIST COMPONENTS ALREADY BUILT that should be reused: -->
    - Navbar (src/components/custom/Navbar.astro)
    - Footer (src/components/custom/Footer.astro)
    - Button (src/components/ui/Button.tsx)
    - Card (src/components/ui/Card.tsx)
  </existing-components>
  <design-tokens>Reference tailwind.config.mjs for colors, fonts, 
    spacing</design-tokens>
  <constraints>
    - Reuse existing components wherever possible
    - Match spacing, colors, and typography precisely
    - Use CSS Grid or Flexbox for layout (via Tailwind)
    - Mobile-responsive (test at 375px, 768px, and 1280px widths)
    - Output as src/pages/[page-name].astro
    - Interactive components must use client:load
    - Internal links must use import.meta.env.BASE_URL
  </constraints>
</prompt>
````

### 6.4 — Bulk Page Generation

````xml
<prompt>
  <role>You are a senior frontend developer building a multi-page 
    prototype.</role>
  <task>Generate the following pages using our existing component library 
    and BaseLayout. Each page should follow the structure described.</task>
  <pages>
    <page name="index" path="/">
      <description>Landing page with hero, features, testimonials, 
        CTA</description>
    </page>
    <page name="pricing" path="/pricing">
      <description>Three-tier pricing cards with feature comparison 
        table</description>
    </page>
    <page name="about" path="/about">
      <description>Team section, mission statement, company 
        stats</description>
    </page>
    <page name="contact" path="/contact">
      <description>Contact form with name, email, message fields 
        + office map placeholder</description>
    </page>
  </pages>
  <shared-elements>
    - All pages use BaseLayout
    - All pages have Navbar and Footer
    - All pages use consistent section spacing (py-16)
    - All internal links use import.meta.env.BASE_URL
  </shared-elements>
  <constraints>
    - Reuse existing components
    - Add navigation links between all pages in the Navbar
    - Consistent visual style across all pages
    - Placeholder content is fine — mark with [PLACEHOLDER]
  </constraints>
</prompt>
````

---

## 7. Scaling & Consistency

### 7.1 — Component Registry

> **Think of it as:** A Storybook-style documentation page — like Figma's asset panel, but for your coded components.

Keep a living inventory of every component so the team knows what's available:

````xml
<prompt>
  <role>You are a frontend developer maintaining a component library.</role>
  <task>Create a component registry page at src/pages/components.astro 
    that displays all available components with live examples and usage 
    code.</task>
  <requirements>
    - List every component in src/components/custom/ and src/components/ui/
    - Show a live rendered example of each
    - Show the code snippet to use each component
    - Clearly mark which components need client:load
    - Group by category (Navigation, Cards, Forms, Feedback, Layout)
    - Make it well-organized with anchor links
  </requirements>
  <constraints>
    - This page is for the team's internal reference
    - Use our BaseLayout
    - Keep it clean and scannable
  </constraints>
</prompt>
````

### 7.2 — Design Token Sync Checklist

When Figma tokens change (new colors, updated spacing, font changes), follow this process:

```
 ┌──────────────────────────────────────────────────────┐
 │           TOKEN SYNC CHECKLIST                       │
 │                                                      │
 │  □ 1. Designer updates tokens in Figma               │
 │  □ 2. Designer documents changes (which token,       │
 │       old value → new value)                         │
 │  □ 3. Update src/styles/tokens.css                   │
 │  □ 4. Update tailwind.config.mjs                     │
 │  □ 5. Run npm run dev — visual check in browser      │
 │  □ 6. Push to feature branch → PR → merge            │
 │  □ 7. Verify on GitHub Pages URL                     │
 └──────────────────────────────────────────────────────┘
```

> **Tip:** Because all components reference tokens (not hardcoded values), changing a token in `tokens.css` and `tailwind.config.mjs` automatically updates every component that uses it. This is the same principle as Figma's local styles — change the style once, it updates everywhere.

### 7.3 — Branching Strategy

```
main (protected — auto-deploys to GitHub Pages)
 ├── feature/new-page-dashboard
 ├── feature/update-navbar
 ├── fix/button-colors
 └── tokens/update-typography
```

**Rules:**
- **Never push directly to `main`** — always use a branch
- Always go: **branch → PR → review → merge**
- Use descriptive branch names with prefixes:
  - `feature/` — new pages or components
  - `fix/` — visual fixes or corrections
  - `tokens/` — design token updates

> **Figma analogy:** `main` is your "final design" page. Branches are like duplicating that page to try a new idea. Pull Requests are showing your idea to the team. Merging is moving the approved idea into the final page.

---

## 8. Refinement Loop

The key to prototypes that match Figma is **iteration, not perfection on the first try**. Expect 2-3 rounds of refinement per component.

```
  ┌──────────┐    ┌───────────┐    ┌──────────┐
  │ Compare  │───▶│  List     │───▶│  Prompt  │──┐
  │ Figma vs │    │  Every    │    │  Cursor  │  │
  │ Browser  │    │  Diff     │    │  to Fix  │  │
  └──────────┘    └───────────┘    └──────────┘  │
       ▲                                         │
       └─────────────────────────────────────────┘
              Repeat until satisfied
```

### The Comparison Prompt

````xml
<prompt>
  <role>You are a QA-focused frontend developer doing a visual 
    review.</role>
  <task>I am going to describe the visual differences between my Figma 
    design and the current browser output. Fix ALL of the following 
    issues.</task>
  <file>[path to component or page]</file>
  <issues>
    <!-- BE SPECIFIC — use exact values from Figma's Inspect panel: -->
    1. Header section: top padding is 64px in Figma, currently looks 
       like ~48px → use pt-16
    2. "Get Started" button: background should be #2563EB (bg-primary), 
       currently appears as default blue
    3. Card grid: should be 3 columns on desktop with 24px gap 
       → use grid-cols-3 gap-6
    4. Body text: line-height should be 1.5 (leading-normal), currently 
       too tight
    5. Section divider: missing 1px bottom border in color #E2E8F0 
       → add border-b border-border
  </issues>
  <constraints>
    - Fix ONLY the listed issues
    - Do not change anything else
    - Use Tailwind utility classes
    - Preserve all existing props and functionality
  </constraints>
</prompt>
````

> **Pro tip for effective refinement:** Open Figma and your browser side by side. Use Figma's Inspect panel to get exact values (px, hex colors, font weights). The more precise your issue descriptions are, the fewer rounds of refinement you'll need.

---

## 9. Tips, Tricks & Pitfalls

### Do's

| Practice | Why |
|----------|-----|
| **Always start from Figma** | Your designs are the source of truth — code follows design, not the other way around |
| **Use Cursor's Composer mode (`Cmd+I`) for multi-file changes** | Creates/edits multiple files at once — ideal for new pages with components |
| **Commit often with descriptive messages** | Easy to undo specific changes if something breaks |
| **Preview locally with `npm run dev`** | Catch issues before pushing — faster feedback loop |
| **Compare browser to Figma side-by-side** | The most effective QA method — use Figma's Inspect panel for exact values |
| **Keep components small and single-purpose** | Smaller = easier to reuse, easier to debug, easier to update |
| **Document component props** | Future team members (and future you) will thank you |

### Don'ts

| Anti-pattern | Why It Hurts |
|-------------|-------------|
| **Don't skip the tokens step** | You'll end up with inconsistent colors/spacing that slowly drift from Figma |
| **Don't create "one giant page" components** | Impossible to reuse or maintain — break things into smaller pieces |
| **Don't write custom CSS files** | Breaks Tailwind conventions, creates specificity conflicts, hard to maintain |
| **Don't push directly to main** | Untested changes go live immediately — always use a branch and PR |
| **Don't ignore mobile responsiveness** | Stakeholders will view prototypes on phones and tablets |
| **Don't over-engineer** | It's a prototype, not production software — favor speed and clarity over perfection |
| **Don't forget `client:load`** | Interactive React components will silently render as dead HTML without it |

### Troubleshooting Quick Reference

| Problem | Likely Cause | Solution |
|---------|-------------|----------|
| **Page is blank after deploy** | Wrong `base` in astro.config.mjs | Make sure `base: '/your-repo-name'` matches your actual repo name |
| **Styles not applying** | Tailwind not scanning the right files | Check the `content` paths in `tailwind.config.mjs` include `./src/**/*.{astro,html,js,jsx,ts,tsx}` |
| **shadcn/ui component doesn't render** | Missing React integration | Run `npx astro add react --yes` and add `react()` to integrations in astro.config.mjs |
| **Interactive component doesn't work** (clicks do nothing) | Missing `client:load` directive | Add `client:load` to the component: `<MyComponent client:load />` |
| **GitHub Pages shows 404** | Pages not configured or missing 404 page | Check repo Settings → Pages → Source is "GitHub Actions"; add a `404.astro` page |
| **Images not loading on GitHub Pages** | Hardcoded paths without base URL | Use `` `${import.meta.env.BASE_URL}image.png` `` or put images in `public/` |
| **Links broken on GitHub Pages** | Hardcoded paths without base URL | Prefix all internal links with `import.meta.env.BASE_URL` |
| **Build fails on deploy** | Code error caught during build | Run `npm run build` locally first — fix any errors before pushing |
| **"Module not found" errors** | Missing dependency | Run `npm install` to ensure all packages are installed |
| **Component shows raw HTML instead of rendering** | Forgot to import or wrong file extension | Check imports match actual file paths and extensions (.astro, .tsx) |

### Cursor Power Tips

1. **`Cmd+K` (Inline Edit):** Select a block of code → describe what to change → quick, targeted fixes
2. **`Cmd+L` (Chat):** Ask questions, get explanations, paste errors for debugging
3. **`Cmd+I` (Composer):** Multi-file creation and editing — the best tool for creating a new page with its components
4. **`@file` reference:** In any prompt, type `@filename.astro` to give Cursor context about an existing file
5. **`@codebase`:** Let Cursor search your entire project — great for "where is this component?" questions
6. **Always review before accepting:** You are the quality gate. Read what Cursor generates, compare to Figma, and only accept if it looks right

### Figma MVP Plugin Tips

1. **Use Auto Layout** in Figma — it maps cleanly to Flexbox/Grid in code (see the Translation Guide in Section 5)
2. **Name your layers clearly** — "hero-cta-button" generates much better code than "Frame 427"
3. **Flatten complex visual groups** before exporting — simplifies the output code
4. **Export one component at a time** — smaller exports = better, cleaner results
5. **Use "Copy as code"** rather than file export — easier to paste directly into Cursor

---

## Quick-Start Cheat Sheet

```
┌────────────────────────────────────────────────────────────────┐
│                    DAILY WORKFLOW CHEAT SHEET                  │
│                                                                │
│  1. git pull origin main              ← Get latest changes     │
│  2. git checkout -b feature/my-page   ← Start a new branch     │
│  3. [Design in Figma]                 ← Create/update designs  │
│  4. [Export via Figma MVP plugin]     ← Get code scaffold      │
│  5. [Paste into Cursor + use prompt]  ← Generate component     │
│  6. npm run dev                       ← Preview in browser     │
│  7. [Compare to Figma → refine]       ← Iterate until matched  │
│  8. git add . && git commit -m "..."  ← Save your progress     │
│  9. git push origin feature/my-page   ← Push to GitHub         │
│ 10. [Create PR on GitHub]             ← Request team review    │
│ 11. [Merge PR]                        ← Auto-deploys to live!  │
│ 12. [Share URL with stakeholders]     ← Collect feedback       │
│                                                                │
│  Repeat for each page or component.                            │
└────────────────────────────────────────────────────────────────┘
```

---

> **Remember:** This workflow is a **cycle, not a straight line**. Feedback from stakeholders viewing the GitHub Pages prototype flows back into Figma updates, which flow back into code. Each iteration makes the prototype better. Start rough, refine progressively, and share often.
