# Jekyll Personal Homepage Refactor — Design Spec

**Date**: 2026-07-06
**Status**: Draft → Pending Implementation
**Context**: Forked from [AcadHomepage](https://github.com/RayeRen/acad-homepage.github.io) (academic template), customized for a non-academic personal resume site.

## 1. Problem Statement

- All content lives in a single `_pages/about.md` (~100 lines) — hard to maintain and extend.
- The template is designed for academics (Google Scholar crawler, publications, invited talks, 30+ academic social fields) but the user is an **Engineering Project Manager** — most of these features are irrelevant.
- Heavy dead code: 60+ SCSS files, old jQuery 1.12.4, unused includes.
- The user has **no frontend/backend development background**. All maintenance is done via agent assistance in VS Code.
- The user wants to add: project showcase cards, career timeline, contact page, and downloads page.

## 2. Goals

1. **Modular content**: split monolithic `about.md` into structured `_data/*.yml` files that are easy for both human and agent to edit.
2. **Remove academic cruft**: delete Google Scholar crawler, academic social fields, unused styles.
3. **Add new components**: timeline, project cards, skills tag cloud.
4. **Add new pages**: Contact, Downloads.
5. **Preserve existing deployment**: GitHub Pages auto-deploy on push — no change to the deployment pipeline.
6. **Agent-friendly**: YAML data files are the primary content interface. Agent edits structured data, not raw HTML.

## 3. Target File Structure

```
1juary.github.io/
├── _config.yml              # Site settings + author info (trimmed)
├── _data/                   # CONTENT HUB — primary maintenance surface
│   ├── navigation.yml       # Nav menu items
│   ├── experience.yml       # Work experience (timeline data)
│   ├── education.yml        # Education (timeline data)
│   ├── projects.yml         # Project showcase (card data)
│   └── skills.yml           # Skill tags
├── _pages/                  # Independent pages
│   ├── home.md              # Homepage (assembles components via includes)
│   ├── contact.md           # Contact page
│   └── downloads.md         # Downloads page
├── _includes/               # Reusable components
│   ├── author-profile.html  # Sidebar (trimmed)
│   ├── timeline.html        # NEW: renders experience/education timelines
│   ├── project-cards.html   # NEW: renders project card grid
│   └── skills-cloud.html    # NEW: renders skill tag cloud
├── _layouts/default.html    # Main layout (unchanged structure)
├── assets/                  # CSS, JS, fonts (trimmed)
├── images/                  # Images
└── files/                   # NEW: downloadable files (resume PDF, etc.)
```

## 4. Data Schemas

### 4.1 `_data/experience.yml`
```yaml
- company: "Sunny Optical Technology"
  role: "Engineering Project Manager (EPM)"
  period: "Jun 2023 - Present"
  highlights:
    - "Managed full lifecycle of mobile camera modules for North American clients"
    - "Eliminated 500+ man-hours of ineffective meeting time"
    - "Achieved $2X,XXX total cost savings"
```

### 4.2 `_data/education.yml`
```yaml
- school: "Hangzhou Dianzi University"
  degree: "B.S. in Cyberspace Security"
  period: "Sep 2019 - Jun 2023"
  notes:
    - "English Proficiency: CET-6 Certified"
```

### 4.3 `_data/projects.yml`
```yaml
- name: "Ultra-wide Rear Camera NPI"
  client: "North American Client"
  summary: "Led NPI phase for two generations of ultra-wide rear cameras"
  highlights:
    - "Comprehensive yield rate >99%"
    - "100% delivery for Mass Production"
    - "Reduced FA analysis time by 48 hours"
  image: ""  # optional, leave empty to show placeholder
  tags: ["NPI", "Camera Module", "Yield Optimization"]
```

### 4.4 `_data/skills.yml`
```yaml
- category: "Project Management"
  items: ["Gantt Chart", "NPI", "Lean Management", "Risk Management (5M1E)"]
- category: "Technical"
  items: ["Python", "Data Analysis", "Excel", "AI Tools"]
- category: "Languages"
  items: ["Chinese (Native)", "English (Professional, CET-6)"]
```

## 5. Component Designs

### 5.1 Timeline Component (`_includes/timeline.html`)
- **Input**: array of entries with `title`, `subtitle`, `period`, `details[]`
- **Rendering**: vertical left-bordered timeline, each entry has a dot + period badge + content
- **Used for**: experience, education (passed a `type` parameter to pull different data sources)
- **Order**: reverse chronological

### 5.2 Project Cards (`_includes/project-cards.html`)
- **Input**: array of projects from `_data/projects.yml`
- **Rendering**: responsive 2-column grid (1 column on mobile), each card has optional image, title, client tag, summary, highlight bullets, skill tags
- **Empty image**: shows a gradient placeholder if no image provided

### 5.3 Skills Cloud (`_includes/skills-cloud.html`)
- **Input**: array of skill categories from `_data/skills.yml`
- **Rendering**: grouped by category, each item is a styled tag/badge

## 6. Page Designs

### 6.1 Homepage (`_pages/home.md`)
- Keep sidebar (author profile) unchanged
- Content area assembles components:
  1. About Me brief (from `_config.yml` author.bio)
  2. Work Experience timeline
  3. Education timeline
  4. Project cards grid
  5. Skills cloud
- Uses `{% include %}` tags to pull in components

### 6.2 Contact Page (`_pages/contact.md`)
- Email, location (from `_config.yml`)
- Optional: WeChat QR code image placeholder
- Social links (GitHub, LinkedIn — from `_config.yml`)

### 6.3 Downloads Page (`_pages/downloads.md`)
- List of downloadable files (resume PDF, certifications)
- Data from `_data/downloads.yml` (simple list: name + file path)

## 7. Deletion List

| Delete | Reason |
|---|---|
| `google_scholar_crawler/` | Academic feature, not needed |
| `.github/workflows/google_scholar_crawler.yaml` | Academic feature |
| `_includes/fetch_google_scholar_stats.html` | Academic feature |
| `_config.yml` academic social fields (pubmed, orcid, dblp, researchgate, googlescholar, impactstory) | Not relevant |
| Commented-out navigation items in `_data/navigation.yml` | Dead code cleanup |
| Commented-out sections in `_pages/about.md` (news, publications, honors, invited talks, internships) | Dead code cleanup |
| Unused vendor SCSS (magnific-popup if not used, etc.) | Reduce noise |

## 8. Modification List

| File | Changes |
|---|---|
| `_config.yml` | Remove academic fields, keep: name, avatar, bio, location, email, github, linkedin. Simplify description. |
| `_pages/about.md` → `_pages/home.md` | Replace raw content with `{% include %}` component tags |
| `_data/navigation.yml` | Add Contact, Downloads entries; remove commented dead items |
| `_includes/author-profile.html` | Remove academic-only social icons (researchgate, dblp, orcid, pubmed, impactstory, googlescholar) |
| `_includes/scripts.html` | Remove `fetch_google_scholar_stats.html` include |
| `_layouts/default.html` | No structural change needed |

## 9. What Does NOT Change

- GitHub Pages auto-deploy (push to main → live in ~30s)
- Jekyll build pipeline (`bundle exec jekyll serve` for local preview)
- Domain (`https://1juary.github.io`)
- Overall visual style (sidebar + content layout, color scheme)
- Existing images and favicon

## 10. Success Criteria

- [ ] `bundle exec jekyll serve` runs without errors locally
- [ ] Homepage renders: about text, timeline (experience + education), project cards, skills cloud
- [ ] Contact page accessible from navigation, renders correctly
- [ ] Downloads page accessible from navigation, renders file list
- [ ] No 404 errors, no broken links
- [ ] Sidebar shows only relevant social links
- [ ] Google Scholar crawler and all academic references fully removed
- [ ] Git diff shows net deletion of dead code + clean additions
