# Design to Prototype Workflow

## Figma / Cursor / Astro / GitHub Pages

> A comprehensive, step-by-step guide for product design teams to build scalable, consistent prototypes — without needing deep engineering skills.

---

## 📋 Table of Contents

1. [Architecture Overview](#1-architecture-overview)
2. [One-Time Setup](#2-one-time-setup)
3. [Workflow: From Figma to Live Prototype](#3-workflow-from-figma-to-live-prototype)
4. [Prompt Library](#4-prompt-library)
5. [Scaling & Consistency](#5-scaling--consistency)
6. [Refinement Loop](#6-refinement-loop)
7. [Tips, Tricks & Pitfalls](#7-tips-tricks--pitfalls)

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

| Tool | Role | Why |
|------|------|-----|
| **Figma** | Source of truth for design | Visual, collaborative, familiar to designers |
| **Figma MVP Plugin** | Exports Figma → code scaffolds | Bridges design → code gap |
| **Cursor** | AI-powered code editor | Writes/refines code via natural language prompts |
| **Astro** | Static site framework | Simple, fast, component-based, great for GH Pages |
| **TailwindCSS** | Utility-first CSS | Maps cleanly to Figma tokens, no CSS files to manage |
| **shadcn/ui** | UI component library | Pre-built, accessible, customizable components |
| **GitHub** | Version control & collaboration | Branching, history, team collaboration |
| **GitHub Pages** | Hosting | Free, automatic deploys, shareable URLs |

---

## 2. One-Time Setup

### Step 2.1 — Install Prerequisites

> 🎯 **Who does this:** One tech-savvy team member (the "Tech Lead") sets this up once, then shares the repo with the team.

1. **Install Node.js** (v20+): [nodejs.org](https://nodejs.org)
2. **Install Cursor**: [cursor.com](https://cursor.com)
3. **Install Git**: [git-scm.com](https://git-scm.com)
4. **Create a GitHub account** if you don't have one
5. **Install Figma MVP plugin** in Figma

### Step 2.2 — Create the Astro Project

Open Cursor's terminal and run:

```bash name=terminal-commands.sh
# Create new Astro project
npm create astro@latest prototype -- --template minimal --yes

# Navigate into the project
cd prototype

# Install Tailwind CSS
npx astro add tailwind --yes

# Enable React integration
npx astro add react --yes

# Install shadcn/ui dependencies
npx shadcn-ui@latest init
```

During `shadcn-ui init`, choose:
- Style: **Default**
- Base color: **Slate** (or match your Figma tokens)
- CSS variables: **Yes**

### Step 2.3 — Project Structure

```
prototype/
├── src/
│   ├── components/        ← Reusable UI components
│   │   ├── ui/            ← shadcn/ui components
│   │   └── custom/        ← Your Figma-matched components
│   ├── layouts/
│   │   └── BaseLayout.astro  ← Shared page shell
│   ├── pages/
│   │   ├── index.astro    ← Homepage
│   │   └── [...other pages]
│   └── styles/
│       └── tokens.css     ← Design tokens from Figma
├── public/                ← Static assets (images, icons)
├── astro.config.mjs
├── tailwind.config.mjs
└── package.json
```

### Step 2.4 — Create the Base Layout

Use this prompt in Cursor (Cmd+K / Ctrl+K):

````xml name=prompt-base-layout.xml
<prompt>
  <role>You are a senior frontend developer building an Astro prototype.</role>
  <task>Create a BaseLayout.astro component in src/layouts/ that includes:
    - HTML5 boilerplate with proper meta tags
    - A slot for page content
    - Import of global styles from ../styles/tokens.css
    - Responsive viewport meta tag
    - A configurable page title via props
  </task>
  <stack>Astro, TailwindCSS</stack>
  <constraints>
    - Keep it minimal and clean
    - Use Astro's built-in slot mechanism
    - Add a "prototype-banner" div at the top that says "PROTOTYPE" 
      with a yellow background so stakeholders know it's not production
  </constraints>
</prompt>
````

### Step 2.5 — Set Up GitHub Repository & Pages

```bash name=github-setup.sh
# Initialize git
git init
git add .
git commit -m "Initial prototype setup"

# Create GitHub repo using GitHub CLI
gh repo create your-org/prototype --public --source=. --push

# ...or create GitHub repo via github.com, copy URL and:
git remote add origin https://github.com/your-username/prototype.git
git branch -M main
git push -u origin main

# Enable GitHub Pages via GitHub Actions
```

Create the deployment workflow using Cursor

```<prompt_github_actions>
  <task>Setup GitHub Actions for Astro deployment</task>
  <context>I want to deploy this project to GitHub Pages.</context>
  <instructions>
    1. Create a folder .github/workflows and a file deploy.yml inside.
    2. Use the official Astro deployment action template.
    3. Update astro.config.mjs to include the 'site' and 'base' properties (use my repo name as base).
  </instructions>
</prompt_github_actions>
```
...or create the deployment workflow manually:

```yaml name=.github/workflows/deploy.yml
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
        env:
          SITE: https://your-org.github.io
          BASE: /prototype
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

Update `astro.config.mjs`:

```javascript name=astro.config.mjs
import { defineConfig } from 'astro/config';
import tailwind from '@astrojs/tailwind';

export default defineConfig({
  integrations: [tailwind()],
  site: 'https://your-org.github.io',
  base: '/prototype',
});
```

---

## 3. Workflow: From Figma to Live Prototype

### The 7-Step Cycle

```
 ┌──────────────────────────────────────────────────────────────┐
 │                    THE DESIGN-TO-CODE CYCLE                  │
 │                                                              │
 │  ① Extract    ② Export     ③ Generate   ④ Refine             │
 │  Tokens  ──▶  Component ─▶  Code   ──▶  in     ────┐         │
 │  (Figma)      (Figma MVP)   (Cursor)    (Cursor)   │         │
 │                                                    ▼         │
 │  ⑦ Share   ◀── ⑥ Deploy  ◀── ⑤ Push                │         │
 │  (URL)         (GH Pages)    (GitHub)  ◀───────────┘         │
 │                                                              │
 └──────────────────────────────────────────────────────────────┘
```

---

### Step 3.1 — Extract Design Tokens from Figma

> 🎨 **Who:** Designer
> 📍 **Where:** Figma

In Figma, document your design tokens (colors, typography, spacing, radii, shadows). Then use this Cursor prompt to create your tokens file:

````xml name=prompt-design-tokens.xml
<prompt>
  <role>You are a senior frontend developer converting Figma design tokens to CSS custom properties and Tailwind config.</role>
  <task>Create two files:
    1. src/styles/tokens.css — CSS custom properties
    2. Update tailwind.config.mjs — extend theme with these tokens
  </task>
  <tokens>
    <colors>
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

> 💡 **Tip:** Replace the token values above with YOUR actual Figma token values. Copy them directly from Figma's inspect panel.

---

### Step 3.2 — Export Component from Figma MVP

> 🎨 **Who:** Designer
> 📍 **Where:** Figma + Figma MVP Plugin

1. Select the component or frame in Figma
2. Open the **Figma MVP** plugin
3. Export as **HTML/CSS** or **React** (either works — Cursor will convert)
4. Copy the generated code output

> ⚠️ **Important:** The exported code is a *starting point*, not the final result. Cursor will refine it.

---

### Step 3.3 — Generate Astro Component in Cursor

> 💻 **Who:** Anyone on the team
> 📍 **Where:** Cursor

Paste the Figma MVP export into Cursor and use this prompt:

````xml name=prompt-convert-component.xml
<prompt>
  <role>You are a senior frontend developer converting a Figma MVP export into a clean Astro component.</role>
  <task>Convert the following Figma MVP export into a reusable Astro component 
    using TailwindCSS utility classes and shadcn/ui where applicable.</task>
  <figma-export>
    <!-- PASTE YOUR FIGMA MVP EXPORT HERE -->
  </figma-export>
  <requirements>
    - Use TailwindCSS utility classes (no inline styles, no custom CSS)
    - Replace any matching UI patterns with shadcn/ui components (Button, Card, Input, etc.)
    - Make the component accept props for dynamic content
    - Use semantic HTML elements
    - Ensure responsive behavior (mobile-first)
    - Use design tokens from our tailwind.config.mjs (e.g., "bg-primary", "text-text-primary")
    - Add TypeScript interface for props
  </requirements>
  <file-location>src/components/custom/[ComponentName].astro</file-location>
  <constraints>
    - Do NOT use any CSS modules or styled-components
    - Do NOT hardcode text content — use props
    - DO use Astro's component syntax (.astro files)
    - Match the visual design as closely as possible
  </constraints>
</prompt>
````

---

### Step 3.4 — Refine the Component

> 💻 **Who:** Anyone on the team
> 📍 **Where:** Cursor

After generating the component, visually compare it to Figma. Use the refinement prompt:

````xml name=prompt-refine-component.xml
<prompt>
  <role>You are a pixel-perfect frontend developer refining a prototype component.</role>
  <task>Refine this Astro component to match the Figma design more closely.</task>
  <current-component>
    <!-- PASTE CURRENT COMPONENT CODE or reference the file -->
  </current-component>
  <differences>
    <!-- LIST WHAT LOOKS DIFFERENT from Figma, for example: -->
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

---

### Step 3.5 — Build a Page

> 💻 **Who:** Anyone on the team
> 📍 **Where:** Cursor

````xml name=prompt-build-page.xml
<prompt>
  <role>You are a senior frontend developer assembling a prototype page from components.</role>
  <task>Create a new Astro page at src/pages/[page-name].astro that composes 
    the following components into a complete page layout.</task>
  <layout>Use BaseLayout from ../layouts/BaseLayout.astro</layout>
  <page-structure>
    <!-- DESCRIBE THE PAGE STRUCTURE FROM FIGMA, for example: -->
    1. Navigation bar (src/components/custom/Navbar.astro)
    2. Hero section with headline, subheadline, and CTA button
    3. Three-column feature cards (src/components/custom/FeatureCard.astro) 
    4. Testimonial section
    5. Footer (src/components/custom/Footer.astro)
  </page-structure>
  <content>
    <!-- PASTE OR DESCRIBE THE ACTUAL CONTENT from Figma -->
    - Headline: "Build better products"
    - Subheadline: "The fastest way to go from idea to prototype"
    - CTA: "Get Started" (links to /get-started)
  </content>
  <constraints>
    - Use existing components — do not recreate them
    - Use Tailwind for layout (flex, grid, spacing)
    - Match Figma's spacing and layout precisely
    - Ensure the page is responsive
  </constraints>
</prompt>
````

---

### Step 3.6 — Push to GitHub

> 💻 **Who:** Anyone on the team
> 📍 **Where:** Cursor terminal

```# 0. Always start by getting the latest version from main
git checkout main
git pull origin main

# 1. Create a feature branch
git checkout -b feature/page-name

# 2. Stage and Commit (The "All-in-one" way)
git add .
git commit -m "Add [page-name] page with [components]"

# 3. Push to GitHub
git push -u origin feature/page-name
```

Then go to GitHub and **create a Pull Request**. The team can review the changes visually once deployed.

---

### Step 3.7 — Deploy & Share

> Merge the PR into `main` → GitHub Actions auto-deploys → share the URL.

Your prototype is live at:
```
https://your-org.github.io/prototype/
```

---

## 4. Prompt Library

### 4.1 — Adding a shadcn/ui Component

````xml name=prompt-add-shadcn.xml
<prompt>
  <role>You are a frontend developer adding shadcn/ui components to an Astro project.</role>
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
    1. Install the shadcn/ui component via CLI
    2. Create an Astro wrapper in src/components/ui/
    3. Apply our design tokens (colors, radii, typography)
    4. Show usage examples
  </instructions>
  <constraints>
    - Use our existing design tokens from tailwind.config.mjs
    - Maintain accessibility (ARIA attributes, keyboard navigation)
    - Keep the API simple for non-technical team members
  </constraints>
</prompt>
````

### 4.2 — Creating Interactive Prototype Elements

````xml name=prompt-interactive.xml
<prompt>
  <role>You are a frontend developer adding interactivity to a prototype.</role>
  <task>Add click-through prototype behavior to the current page.</task>
  <interactions>
    <!-- DESCRIBE INTERACTIONS, e.g.: -->
    - Clicking "Get Started" button navigates to /onboarding
    - Clicking a navigation item highlights it and navigates to the section
    - The mobile menu toggles open/closed on hamburger click
    - Tabs switch content panels without page reload
    - Form inputs show validation states (but don't submit anywhere)
  </interactions>
  <constraints>
    - Use minimal JavaScript (vanilla JS or Astro client:load)
    - No backend/API calls — this is a prototype
    - Keep interactions simple and demonstrative
    - Use Astro's built-in page routing for navigation
  </constraints>
</prompt>
````

### 4.3 — Creating a New Page from Figma Screenshot

````xml name=prompt-from-screenshot.xml
<prompt>
  <role>You are a pixel-perfect frontend developer.</role>
  <task>Look at the attached Figma screenshot and recreate this page as an 
    Astro component using TailwindCSS and shadcn/ui.</task>
  <image>
    <!-- ATTACH SCREENSHOT IN CURSOR or describe the layout in detail -->
  </image>
  <existing-components>
    <!-- LIST COMPONENTS ALREADY BUILT that should be reused: -->
    - Navbar (src/components/custom/Navbar.astro)
    - Footer (src/components/custom/Footer.astro)
    - Button (src/components/ui/Button.astro)
    - Card (src/components/ui/Card.astro)
  </existing-components>
  <design-tokens>Reference tailwind.config.mjs for colors, fonts, spacing</design-tokens>
  <constraints>
    - Reuse existing components wherever possible
    - Match spacing, colors, and typography precisely
    - Use CSS Grid or Flexbox for layout (via Tailwind)
    - Mobile-responsive
    - Output as src/pages/[page-name].astro
  </constraints>
</prompt>
````

### 4.4 — Bulk Page Generation

````xml name=prompt-bulk-pages.xml
<prompt>
  <role>You are a senior frontend developer building a multi-page prototype.</role>
  <task>Generate the following pages using our existing component library 
    and BaseLayout. Each page should follow the structure described.</task>
  <pages>
    <page name="index" path="/">
      <description>Landing page with hero, features, testimonials, CTA</description>
    </page>
    <page name="pricing" path="/pricing">
      <description>Three-tier pricing cards with feature comparison table</description>
    </page>
    <page name="about" path="/about">
      <description>Team section, mission statement, company stats</description>
    </page>
    <page name="contact" path="/contact">
      <description>Contact form with name, email, message fields + office map placeholder</description>
    </page>
  </pages>
  <shared-elements>
    - All pages use BaseLayout
    - All pages have Navbar and Footer
    - All pages use consistent spacing (py-16 for sections)
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

## 5. Scaling & Consistency

### 5.1 — Component Registry

Maintain a simple registry file so the team knows what's available:

````xml name=prompt-component-registry.xml
<prompt>
  <role>You are a frontend developer maintaining a component library.</role>
  <task>Create a component registry page at src/pages/components.astro that 
    displays all available components with live examples and usage code.</task>
  <requirements>
    - List every component in src/components/custom/ and src/components/ui/
    - Show a live rendered example of each
    - Show the code snippet to use each component
    - Group by category (Navigation, Cards, Forms, Feedback, Layout)
    - Make it searchable or at least well-organized with anchor links
  </requirements>
  <constraints>
    - This page is for the team's internal reference
    - Use our BaseLayout
    - Keep it clean and scannable
  </constraints>
</prompt>
````

### 5.2 — Design Token Sync Checklist

When Figma tokens change, follow this process:

```
 ┌──────────────────────────────────────────────────┐
 │           TOKEN SYNC CHECKLIST                   │
 │                                                  │
 │  □ 1. Designer updates tokens in Figma           │
 │  □ 2. Designer documents changes in a table      │
 │  □ 3. Dev updates src/styles/tokens.css          │
 │  □ 4. Dev updates tailwind.config.mjs            │
 │  □ 5. Run prototype locally — visual check       │
 │  □ 6. Push to feature branch → PR → merge        │
 │  □ 7. Verify on GitHub Pages                     │
 └──────────────────────────────────────────────────┘
```

### 5.3 — Branching Strategy

```
main (protected — auto-deploys to GitHub Pages)
 ├── feature/new-page-dashboard
 ├── feature/update-navbar
 ├── fix/button-colors
 └── tokens/update-typography
```

**Rules:**
- Never push directly to `main`
- Always create a branch → PR → review → merge
- Use descriptive branch names: `feature/`, `fix/`, `tokens/`

---

## 6. Refinement Loop

The key to making prototypes look close to the Figma source is an **iterative refinement loop**:

```
  ┌──────────┐    ┌───────────┐    ┌──────────┐
  │ Compare  │───▶│  List     │───▶│  Prompt  │──┐
  │ Figma vs │    │  Diffs    │    │  Cursor  │  │
  │ Browser  │    │           │    │  to Fix  │  │
  └──────────┘    └───────────┘    └──────────┘  │
       ▲                                         │
       └─────────────────────────────────────────┘
              Repeat until satisfied
```

### The Comparison Prompt

````xml name=prompt-comparison.xml
<prompt>
  <role>You are a QA-focused frontend developer doing a visual review.</role>
  <task>I am going to describe the visual differences between my Figma design 
    and the current browser output. Fix ALL of the following issues.</task>
  <file>[path to component or page]</file>
  <issues>
    <!-- BE SPECIFIC. Use exact values: -->
    1. Header section: top padding is 64px in Figma, currently looks like ~48px → use pt-16
    2. "Get Started" button: background should be #2563EB, currently appears as default blue
    3. Card grid: should be 3 columns on desktop with 24px gap → use grid-cols-3 gap-6
    4. Body text: line-height should be 1.5 (leading-normal), currently too tight
    5. Section divider: missing 1px border-bottom in color #E2E8F0 → add border-b border-border
  </issues>
  <constraints>
    - Fix ONLY the listed issues
    - Do not change anything else
    - Use Tailwind utility classes
    - Preserve all existing props and functionality
  </constraints>
</prompt>
````

---

## 7. Tips, Tricks & Pitfalls

### ✅ Do's

| Practice | Why |
|----------|-----|
| **Always start from Figma** | Single source of truth prevents design drift |
| **Use Cursor's Composer mode** for multi-file changes | Creates/edits multiple files at once — great for pages + components |
| **Commit often with clear messages** | Easy to rollback if something breaks |
| **Use `npm run dev` to preview locally** | Catch issues before pushing |
| **Screenshot your browser & compare to Figma side-by-side** | Most effective QA method |
| **Keep components small and single-purpose** | Easier to reuse and maintain |
| **Document component props** | Future team members will thank you |

### ❌ Don'ts

| Anti-pattern | Why it hurts |
|-------------|-------------|
| **Don't skip the tokens step** | You'll end up with inconsistent colors/spacing everywhere |
| **Don't create "one giant page" components** | Impossible to reuse or maintain |
| **Don't write custom CSS** | Breaks Tailwind conventions, creates conflicts |
| **Don't push directly to main** | Breaks the live prototype without review |
| **Don't ignore mobile responsiveness** | Stakeholders will view on phones |
| **Don't over-engineer** | It's a prototype, not production — favor speed over perfection |

### 🔧 Troubleshooting Quick Reference

| Problem | Solution |
|---------|----------|
| Page is blank after deploy | Check `base` in `astro.config.mjs` matches your repo name |
| Styles not applying | Run `npx tailwindcss init` and check content paths in config |
| shadcn/ui component not rendering | Ensure you have the React integration: `npx astro add react` |
| GitHub Pages 404 | Add a `404.astro` page; check the repo Pages settings |
| Images not loading | Put them in `public/` folder, reference with `/{base}/image.png` |
| Build fails on deploy | Run `npm run build` locally first to catch errors |

### 🏗️ Cursor Power Tips

1. **`Cmd+K` (Inline Edit):** Select code → describe what to change → quick fixes
2. **`Cmd+L` (Chat):** Ask questions about your codebase, get explanations
3. **Composer (`Cmd+I`):** Multi-file creation/editing — best for new pages
4. **`@file` reference:** In any prompt, type `@filename.astro` to give Cursor context about existing files
5. **`@codebase`:** Let Cursor search your entire project for relevant context
6. **Accept/Reject changes:** Always review what Cursor generates before accepting — you are the quality gate

### 📐 Figma MVP Plugin Tips

1. **Flatten complex groups** before exporting — simplifies the output
2. **Use Auto Layout** in Figma — it maps much better to Flexbox/Grid
3. **Name your layers clearly** — the plugin uses layer names in the generated code
4. **Export one component at a time** — more manageable, better results
5. **Use the "Copy as code" feature** rather than exporting files

---

## Quick-Start Cheat Sheet

```
┌─────────────────────────────────────────────────────────────────┐
│                    DAILY WORKFLOW CHEAT SHEET                   │
│                                                                 │
│  1. git pull origin main              ← Get latest              │
│  2. git checkout -b feature/my-page   ← New branch              │
│  3. [Design in Figma]                 ← Update/create designs   │
│  4. [Export via Figma MVP]            ← Get code scaffold       │
│  5. [Paste into Cursor + Prompt]      ← Generate component      │
│  6. npm run dev                       ← Preview locally         │
│  7. [Compare to Figma, refine]        ← Iterate in Cursor       │
│  8. git add . && git commit -m "..."  ← Save work               │
│  9. git push origin feature/my-page   ← Push to GitHub          │
│ 10. [Create PR on GitHub]             ← Team review             │
│ 11. [Merge PR]                        ← Auto-deploys!           │
│ 12. [Share URL with stakeholders]     ← Get feedback            │
│                                                                 │
│ Repeat for each page/component                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

> **Remember:** This workflow is a **cycle, not a line**. Feedback from stakeholders viewing the GitHub Pages prototype flows back into Figma updates, which flow back into code. Each iteration makes the prototype better. Start rough, refine progressively, and ship often. 🚀
