# Jekyll Personal Homepage Refactor — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Refactor a monolithic Jekyll academic homepage into a modular, data-driven personal resume site with timeline, project cards, contact page, and downloads page.

**Architecture:** Content moves from one large `_pages/about.md` into structured `_data/*.yml` files. New Liquid include components (`timeline.html`, `project-cards.html`, `skills-cloud.html`) render data into HTML. Academic dead code (Google Scholar, 30+ academic social fields) is removed. GitHub Pages auto-deploy is preserved.

**Tech Stack:** Jekyll (Ruby), Liquid templates, YAML data files, SCSS (existing theme), GitHub Pages

## Global Constraints

- `bundle exec jekyll serve` must run without errors
- Push to main → auto-deploy via GitHub Pages (unchanged)
- Keep existing visual style: sidebar + content layout, color scheme, typography
- All new content driven by `_data/*.yml` files — agent edits YAML, not HTML
- No new Ruby gem dependencies
- No changes to `Gemfile`, `Gemfile.lock`, or `_layouts/default.html` structure

---

## Phase 1: Cleanup — Remove Academic Dead Code

### Task 1: Delete Google Scholar crawler and workflow

**Files:**
- Delete: `google_scholar_crawler/` (entire directory)
- Delete: `.github/workflows/google_scholar_crawler.yaml`

- [ ] **Step 1: Remove google_scholar_crawler directory**

```bash
rm -rf google_scholar_crawler
```

- [ ] **Step 2: Remove GitHub Actions workflow**

```bash
rm .github/workflows/google_scholar_crawler.yaml
```

- [ ] **Step 3: Remove fetch_google_scholar_stats include**

```bash
rm _includes/fetch_google_scholar_stats.html
```

- [ ] **Step 4: Commit**

```bash
git add -A
git commit -m "chore: remove Google Scholar crawler and workflow"
```

---

### Task 2: Clean up _config.yml — remove academic-only fields

**Files:**
- Modify: `_config.yml`

**What to change:**

- [ ] **Step 1: Update `_config.yml`**

Replace lines 1-58 of `_config.yml` (the Site Settings and Site Author sections) with this trimmed version:

```yaml
# Site Settings
title                    : "Juary Wang"
description              : "Engineering Project Manager (EPM). Specializing in NPI, lean management, and mobile camera module projects."
repository               : "1juary/1juary.github.io"

# google analytics (optional — uncomment and add ID to enable)
google_analytics_id      :  # get google_analytics_id from https://analytics.google.com/analytics/

# SEO Related
google_site_verification :  # get google_site_verification from https://search.google.com/search-console/about
bing_site_verification   :  # get bing_site_verification from https://www.bing.com/webmasters/about
baidu_site_verification  :  # get baidu_site_verification from https://ziyuan.baidu.com/login/index?u=/site/index

# Site Author
author:
  name             : "Juary Wang"
  avatar           : "images/personal_profile.png"
  bio              : "EPM | Data analysis | Solution-oriented"
  location         : "Zhejiang, China"
  email            : "juarywong@163.com"
  github           : # e.g., "github username"
  linkedin         : # e.g., "linkedin username"
```

This removes: `google_scholar_stats_use_cdn`, `employer`, `pubmed`, `googlescholar`, `researchgate`, `uri`, `bitbucket`, `codepen`, `dribbble`, `flickr`, `facebook`, `foursquare`, `google_plus`, `keybase`, `instagram`, `impactstory`, `lastfm`, `dblp`, `orcid`, `pinterest`, `soundcloud`, `stackoverflow`, `steam`, `tumblr`, `twitter`, `vine`, `weibo`, `xing`, `youtube`, `wikipedia`.

The rest of `_config.yml` (lines 62-164, Reading Files through HTML Compression) stays unchanged.

- [ ] **Step 2: Verify the YAML is valid**

```bash
ruby -ryaml -e "YAML.load_file('_config.yml')" && echo "YAML valid"
```

Expected: `YAML valid`

- [ ] **Step 3: Commit**

```bash
git add _config.yml
git commit -m "chore: trim _config.yml to relevant fields only"
```

---

### Task 3: Clean up _data/navigation.yml

**Files:**
- Modify: `_data/navigation.yml`

- [ ] **Step 1: Replace navigation.yml content**

Replace the entire contents of `_data/navigation.yml`:

```yaml
# main links
main:
  - title: "About Me"
    url: "/#about-me"

  - title: "Contact"
    url: "/contact/"

  - title: "Downloads"
    url: "/downloads/"
```

- [ ] **Step 2: Commit**

```bash
git add _data/navigation.yml
git commit -m "chore: clean up navigation, add Contact and Downloads"
```

---

### Task 4: Remove dead SCSS files not used by the current layout

**Files:**
- Delete: `_sass/_archive.scss`, `_sass/_tables.scss`, `_sass/_notices.scss`, `_sass/_syntax.scss`, `_sass/_print.scss`, `_sass/_footer.scss`, `_sass/_buttons.scss`, `_sass/_forms.scss`
- Delete: `_sass/vendor/magnific-popup/` (entire directory)

**Rationale:** These files style features (code syntax highlighting, tables, buttons, forms, print styles, archive pages, lightbox popups) that the site does not use. Removing them reduces the file count without affecting any rendered output.

- [ ] **Step 1: Remove unused SCSS files**

```bash
rm _sass/_archive.scss
rm _sass/_tables.scss
rm _sass/_notices.scss
rm _sass/_syntax.scss
rm _sass/_print.scss
rm _sass/_footer.scss
rm _sass/_buttons.scss
rm _sass/_forms.scss
rm -rf _sass/vendor/magnific-popup
```

- [ ] **Step 2: Verify Jekyll still builds**

```bash
bundle exec jekyll build 2>&1
```

Expected: No errors, site builds successfully.

- [ ] **Step 3: Commit**

```bash
git add -A
git commit -m "chore: remove unused SCSS files"
```

---

## Phase 2: Create Data Files

### Task 5: Create `_data/experience.yml`

**Files:**
- Create: `_data/experience.yml`

**Interfaces:**
- Produces: `site.data.experience` — array of `{company, role, period, highlights[]}` objects, consumed by `_includes/timeline.html`

- [ ] **Step 1: Write the data file**

```yaml
# Work Experience
# To add a new entry, copy the block below and fill in your details.
# Order: displayed in file order (top = most recent recommended)

- company: "Sunny Optical Technology"
  role: "Engineering Project Manager (EPM)"
  period: "Jun 2023 - Present"
  highlights:
    - "Managed the full lifecycle (R&D to Mass Production) of mobile camera modules for North American clients, with a primary focus on on-time delivery and quality assurance."
    - "Enhanced team efficiency by eliminating over 500 man-hours (approx. 30,000 mins) of ineffective meeting time and establishing 10+ high-quality SOPs."
    - "Orchestrated cost reduction initiatives to boost client satisfaction and profitability, achieving total savings of over $2X,XXX."
```

- [ ] **Step 2: Commit**

```bash
git add _data/experience.yml
git commit -m "feat: add experience data file"
```

---

### Task 6: Create `_data/education.yml`

**Files:**
- Create: `_data/education.yml`

**Interfaces:**
- Produces: `site.data.education` — array of `{school, degree, period, notes[]}` objects, consumed by `_includes/timeline.html`

- [ ] **Step 1: Write the data file**

```yaml
# Education
# To add a new entry, copy the block below and fill in your details.

- school: "Hangzhou Dianzi University"
  degree: "B.S. in Cyberspace Security"
  period: "Sep 2019 - Jun 2023"
  notes:
    - "English Proficiency: Capable of seamless technical communication in all-English environments (CET-6 Certified)."
```

- [ ] **Step 2: Commit**

```bash
git add _data/education.yml
git commit -m "feat: add education data file"
```

---

### Task 7: Create `_data/projects.yml`

**Files:**
- Create: `_data/projects.yml`

**Interfaces:**
- Produces: `site.data.projects` — array of `{name, client, summary, highlights[], image, tags[]}` objects, consumed by `_includes/project-cards.html`

- [ ] **Step 1: Write the data file**

```yaml
# Project Showcase
# To add a new project, copy the block below and fill in your details.
# image: leave empty ("") to show a gradient placeholder, or put a path like "images/project-xxx.jpg"

- name: "Mobile Camera NPI Management"
  client: "North American Client"
  summary: "Led the NPI phase for two generations of ultra-wide rear cameras, achieving >99% comprehensive yield rates and 100% on-time delivery for Mass Production. Directed R&D of new Compact IR Camera technology."
  highlights:
    - "Comprehensive yield rates >99% and 100% delivery for Mass Production"
    - "Directed R&D of new Compact IR Camera technology with >99% yield"
    - "Optimized Failure Analysis workflows, reducing analysis time by 48 hours"
    - "Managed 50+ Engineering Change Orders per project, coordinating with 10+ internal stakeholders and 10+ client representatives"
    - "Achieved 100% delivery success across 100+ batches with >10% gross margin"
  image: ""
  tags: ["NPI", "Camera Module", "Yield Optimization", "Risk Management"]

- name: "Core Supplier Management"
  client: "Multiple Suppliers"
  summary: "Led technical evaluation and integration of 5+ chipsets for diverse camera modules, spanning 1-inch sensors, ultra-wide, main, and drone camera applications."
  highlights:
    - "Achieved >50% cost reduction in R&D projects"
    - "Maintained 100% record in schedule adherence, client satisfaction, and issue resolution"
  image: ""
  tags: ["Supplier Management", "Chipset Evaluation", "Cost Reduction"]
```

- [ ] **Step 2: Commit**

```bash
git add _data/projects.yml
git commit -m "feat: add projects data file"
```

---

### Task 8: Create `_data/skills.yml`

**Files:**
- Create: `_data/skills.yml`

**Interfaces:**
- Produces: `site.data.skills` — array of `{category, items[]}` objects, consumed by `_includes/skills-cloud.html`

- [ ] **Step 1: Write the data file**

```yaml
# Skills
# Grouped by category. Add or remove items freely.

- category: "Project Management"
  items:
    - "Gantt Chart"
    - "NPI Management"
    - "Lean Manufacturing"
    - "Risk Management (5M1E)"
    - "ECO Management"
    - "Cross-functional Collaboration"

- category: "Technical"
  items:
    - "Python"
    - "Data Analysis"
    - "Excel"
    - "Process Automation"
    - "AI Tools"

- category: "Languages"
  items:
    - "Chinese (Native)"
    - "English (Professional, CET-6)"
```

- [ ] **Step 2: Commit**

```bash
git add _data/skills.yml
git commit -m "feat: add skills data file"
```

---

### Task 9: Create `_data/downloads.yml`

**Files:**
- Create: `_data/downloads.yml`
- Create: `files/` directory with a placeholder

**Interfaces:**
- Produces: `site.data.downloads` — array of `{name, description, file}` objects, consumed by `_pages/downloads.md`

- [ ] **Step 1: Create files directory**

```bash
mkdir -p files
```

- [ ] **Step 2: Write the data file**

```yaml
# Downloadable Files
# Add files to the "files/" directory, then list them here.
# Example: put "resume.pdf" in files/, then add an entry below.

# - name: "Resume (PDF)"
#   description: "My latest resume in PDF format."
#   file: "files/resume.pdf"

# - name: "Certificate Example"
#   description: "Professional certification."
#   file: "files/certificate.pdf"
```

(File starts with commented-out examples so the page renders but shows "no files yet" — the user adds entries when ready.)

- [ ] **Step 3: Commit**

```bash
git add _data/downloads.yml files/
git commit -m "feat: add downloads data file and files directory"
```

---

## Phase 3: Create Include Components

### Task 10: Create `_includes/timeline.html`

**Files:**
- Create: `_includes/timeline.html`

**Interfaces:**
- Consumes: `include.type` parameter — `"experience"` or `"education"`, which selects `site.data.experience` or `site.data.education`
- Produces: HTML timeline markup

- [ ] **Step 1: Write the component**

```html
{% comment %}
  Timeline component
  Usage: {% include timeline.html type="experience" %}
         {% include timeline.html type="education" %}
  Data source: _data/experience.yml or _data/education.yml
{% endcomment %}

{% case include.type %}
  {% when "experience" %}
    {% assign entries = site.data.experience %}
    {% assign section_title = "💼 Work Experience" %}
  {% when "education" %}
    {% assign entries = site.data.education %}
    {% assign section_title = "📖 Education" %}
{% endcase %}

{% if entries %}
<h1>{{ section_title }}</h1>

<div class="timeline">
  {% for entry in entries %}
  <div class="timeline__item">
    <div class="timeline__dot"></div>
    <div class="timeline__content">
      <span class="timeline__period">{{ entry.period }}</span>
      {% if entry.role %}
        <h3 class="timeline__title">{{ entry.role }}</h3>
        <h4 class="timeline__subtitle">{{ entry.company }}</h4>
      {% else %}
        <h3 class="timeline__title">{{ entry.degree }}</h3>
        <h4 class="timeline__subtitle">{{ entry.school }}</h4>
      {% endif %}
      {% if entry.highlights %}
        <ul class="timeline__highlights">
          {% for h in entry.highlights %}
            <li>{{ h }}</li>
          {% endfor %}
        </ul>
      {% endif %}
      {% if entry.notes %}
        <p class="timeline__notes">
          {% for n in entry.notes %}
            {{ n }}{% unless forloop.last %}<br>{% endunless %}
          {% endfor %}
        </p>
      {% endif %}
    </div>
  </div>
  {% endfor %}
</div>
{% endif %}
```

- [ ] **Step 2: Commit**

```bash
git add _includes/timeline.html
git commit -m "feat: add timeline component"
```

---

### Task 11: Create `_includes/project-cards.html`

**Files:**
- Create: `_includes/project-cards.html`

**Interfaces:**
- Consumes: `site.data.projects` — array of `{name, client, summary, highlights[], image, tags[]}`
- Produces: HTML responsive card grid

- [ ] **Step 1: Write the component**

```html
{% comment %}
  Project Cards component
  Data source: _data/projects.yml
{% endcomment %}

{% assign projects = site.data.projects %}

{% if projects %}
<h1>📱 Project Experience</h1>

<div class="project-cards">
  {% for project in projects %}
  <div class="project-card">
    {% if project.image and project.image != "" %}
      <div class="project-card__image">
        <img src="{{ project.image }}" alt="{{ project.name }}" loading="lazy">
      </div>
    {% else %}
      <div class="project-card__image project-card__image--placeholder">
        <span>{{ project.name | truncate: 1, "" }}</span>
      </div>
    {% endif %}
    <div class="project-card__body">
      <h3 class="project-card__title">{{ project.name }}</h3>
      {% if project.client %}
        <span class="project-card__client">{{ project.client }}</span>
      {% endif %}
      <p class="project-card__summary">{{ project.summary }}</p>
      {% if project.highlights %}
        <ul class="project-card__highlights">
          {% for h in project.highlights %}
            <li>{{ h }}</li>
          {% endfor %}
        </ul>
      {% endif %}
      {% if project.tags %}
        <div class="project-card__tags">
          {% for tag in project.tags %}
            <span class="project-card__tag">{{ tag }}</span>
          {% endfor %}
        </div>
      {% endif %}
    </div>
  </div>
  {% endfor %}
</div>
{% endif %}
```

- [ ] **Step 2: Commit**

```bash
git add _includes/project-cards.html
git commit -m "feat: add project cards component"
```

---

### Task 12: Create `_includes/skills-cloud.html`

**Files:**
- Create: `_includes/skills-cloud.html`

**Interfaces:**
- Consumes: `site.data.skills` — array of `{category, items[]}`
- Produces: HTML skill tag cloud grouped by category

- [ ] **Step 1: Write the component**

```html
{% comment %}
  Skills Cloud component
  Data source: _data/skills.yml
{% endcomment %}

{% assign skills = site.data.skills %}

{% if skills %}
<h1>🛠️ Skills</h1>

<div class="skills-cloud">
  {% for group in skills %}
  <div class="skills-cloud__group">
    <h3 class="skills-cloud__category">{{ group.category }}</h3>
    <div class="skills-cloud__tags">
      {% for skill in group.items %}
        <span class="skills-cloud__tag">{{ skill }}</span>
      {% endfor %}
    </div>
  </div>
  {% endfor %}
</div>
{% endif %}
```

- [ ] **Step 2: Commit**

```bash
git add _includes/skills-cloud.html
git commit -m "feat: add skills cloud component"
```

---

## Phase 4: Update Existing Files

### Task 13: Update `_includes/author-profile.html` — remove academic social icons

**Files:**
- Modify: `_includes/author-profile.html`

- [ ] **Step 1: Strip academic-only social links**

The file currently lists 30+ social platforms with `{% if author.xxx %}` blocks. We need to remove the academic-specific ones.

Remove the following blocks from `_includes/author-profile.html`:

1. Lines 28-29: `employer` block
2. Lines 37-39: `researchgate` block
3. Lines 55-57: `xing` block
4. Lines 76-78: `lastfm` block
5. Lines 82-84: `foursquare` block
6. Lines 88-90: `youtube` block
7. Lines 91-93: `soundcloud` block
8. Lines 97-99: `flickr` block
9. Lines 103-105: `vine` block
10. Lines 106-108: `googlescholar` block
11. Lines 109-111: `pubmed` block
12. Lines 112-114: `orcid` block
13. Lines 115-117: `impactstory` block
14. Lines 118-120: `wikipedia` block
15. Lines 48-51: `google_plus` block
16. Lines 52-54: `linkedin` block → keep this one, just move it
17. Lines 49-51: `dblp` block
18. Lines 67-69: `bitbucket` block
19. Lines 73-75: `stackoverflow` block
20. Lines 79-81: `pinterest` block
21. Lines 70-72: `instagram` block
22. Lines 61-63: `tumblr` block
23. Lines 64-66: `codepen` block
24. Lines 58-60: `dribbble` block
25. Lines 85-87: `steam` block
26. Lines 94-96: `weibo` block
27. Lines 100-102: `facebook` block

Keep only: `location`, `email`, `github`, `linkedin`, `twitter`, `uri`. And keep the `description` block at the top.

The same removals must be applied to the `author__urls_sm` (small screen) section at the bottom.

Here's the exact replacement — replace the entire file:

```html
{% if page.author and site.data.authors[page.author] %}
  {% assign author = site.data.authors[page.author] %}{% else %}{% assign author = site.author %}
{% endif %}

<div itemscope itemtype="http://schema.org/Person" class="profile_box">

  <div class="author__avatar">
    <img src="{{ author.avatar }}" class="author__avatar" alt="{{ author.name }}">
  </div>

  <div class="author__content">
    <h3 class="author__name">{{ author.name }}</h3>
    {% if author.bio %}<p class="author__bio">{{ author.bio }}</p>{% endif %}
  </div>

  <div class="author__urls-wrapper">
    <ul class="author__urls social-icons">
      {% if site.description %}
        <li><div style="white-space: normal; margin-bottom: 1em;">{{ site.description }}</div></li>
      {% endif %}
      {% if author.location %}
        <li><i class="fa fa-fw fa-map-marker" aria-hidden="true"></i> {{ author.location }}</li>
      {% endif %}
      {% if author.email %}
        <li><a href="mailto:{{ author.email }}"><i class="fas fa-fw fa-envelope" aria-hidden="true"></i> Email</a></li>
      {% endif %}
      {% if author.github %}
        <li><a href="https://github.com/{{ author.github }}"><i class="fab fa-fw fa-github" aria-hidden="true"></i> Github</a></li>
      {% endif %}
      {% if author.linkedin %}
        <li><a href="https://www.linkedin.com/in/{{ author.linkedin }}"><i class="fab fa-fw fa-linkedin" aria-hidden="true"></i> LinkedIn</a></li>
      {% endif %}
      {% if author.twitter %}
        <li><a href="https://twitter.com/{{ author.twitter }}"><i class="fab fa-fw fa-twitter-square" aria-hidden="true"></i> Twitter</a></li>
      {% endif %}
      {% if author.uri %}
        <li><a href="{{ author.uri }}"><i class="fas fa-fw fa-link" aria-hidden="true"></i> Website</a></li>
      {% endif %}
    </ul>
    <div class="author__urls_sm">
      {% if author.email %}
        <a href="mailto:{{ author.email }}"><i class="fas fa-fw fa-envelope" aria-hidden="true"></i></a>
      {% endif %}
      {% if author.github %}
        <a href="https://github.com/{{ author.github }}"><i class="fab fa-fw fa-github" aria-hidden="true"></i></a>
      {% endif %}
      {% if author.linkedin %}
        <a href="https://www.linkedin.com/in/{{ author.linkedin }}"><i class="fab fa-fw fa-linkedin" aria-hidden="true"></i></a>
      {% endif %}
      {% if author.twitter %}
        <a href="https://twitter.com/{{ author.twitter }}"><i class="fab fa-fw fa-twitter-square" aria-hidden="true"></i></a>
      {% endif %}
      {% if author.uri %}
        <a href="{{ author.uri }}"><i class="fas fa-fw fa-link" aria-hidden="true"></i></a>
      {% endif %}
    </div>
  </div>
</div>
```

- [ ] **Step 2: Commit**

```bash
git add _includes/author-profile.html
git commit -m "refactor: trim author-profile to non-academic social links"
```

---

### Task 14: Update `_includes/scripts.html` — remove Google Scholar stats fetch

**Files:**
- Modify: `_includes/scripts.html`

- [ ] **Step 1: Replace scripts.html content**

Current content (lines 1-4):
```html
<script src="assets/js/main.min.js"></script>

{% include analytics.html %}
{% include fetch_google_scholar_stats.html %}
```

Replace with:
```html
<script src="assets/js/main.min.js"></script>

{% include analytics.html %}
```

- [ ] **Step 2: Verify no reference to the deleted file remains**

```bash
grep -r "fetch_google_scholar_stats" . --include="*.html" --include="*.md" --include="*.yml"
```

Expected: No matches.

- [ ] **Step 3: Commit**

```bash
git add _includes/scripts.html
git commit -m "fix: remove Google Scholar stats include from scripts"
```

---

## Phase 5: Create New Pages

### Task 15: Replace `_pages/about.md` with `_pages/home.md`

**Files:**
- Delete: `_pages/about.md`
- Create: `_pages/home.md`

**Interfaces:**
- Consumes: `site.data.experience`, `site.data.education`, `site.data.projects`, `site.data.skills` via includes
- Produces: Homepage at `/`

- [ ] **Step 1: Delete old about.md**

```bash
rm _pages/about.md
```

- [ ] **Step 2: Create home.md**

```markdown
---
permalink: /
title: ""
excerpt: ""
author_profile: true
redirect_from:
  - /about/
  - /about.html
---

<span class='anchor' id='about-me'></span>

I am an **Engineering Project Manager (EPM)** at **Sunny Optical Technology**, specializing in the full lifecycle management (NPI to Mass Production) of mobile camera modules for North American clients.

I hold a B.S. in **Cyberspace Security** from **Hangzhou Dianzi University** (2019-2023). My expertise lies in technical project management, cross-functional collaboration, and lean manufacturing. I am proficient in **Python and Data Analysis**, leveraging these skills to automate workflows and drive efficiency in complex manufacturing environments.

{% include timeline.html type="experience" %}

{% include timeline.html type="education" %}

{% include project-cards.html %}

{% include skills-cloud.html %}
```

- [ ] **Step 3: Commit**

```bash
git add _pages/home.md _pages/about.md
git commit -m "feat: replace about.md with modular home.md"
```

---

### Task 16: Create `_pages/contact.md`

**Files:**
- Create: `_pages/contact.md`

- [ ] **Step 1: Write contact.md**

```markdown
---
permalink: /contact/
title: "Contact Me"
excerpt: "Get in touch with Juary Wang"
author_profile: true
---

<span class='anchor' id='contact'></span>

# 📧 Contact

Feel free to reach out — I'm always open to discussing new opportunities, collaborations, or questions.

| Method | Detail |
|---|---|
| **Email** | [{{ site.author.email }}](mailto:{{ site.author.email }}) |
| **Location** | {{ site.author.location }} |

{% if site.author.github %}
- **GitHub**: [github.com/{{ site.author.github }}](https://github.com/{{ site.author.github }})
{% endif %}
{% if site.author.linkedin %}
- **LinkedIn**: [linkedin.com/in/{{ site.author.linkedin }}](https://www.linkedin.com/in/{{ site.author.linkedin }})
{% endif %}

---

*I typically respond within 24 hours on business days.*
```

- [ ] **Step 2: Commit**

```bash
git add _pages/contact.md
git commit -m "feat: add contact page"
```

---

### Task 17: Create `_pages/downloads.md`

**Files:**
- Create: `_pages/downloads.md`

- [ ] **Step 1: Write downloads.md**

```markdown
---
permalink: /downloads/
title: "Downloads"
excerpt: "Downloadable resources from Juary Wang"
author_profile: true
---

<span class='anchor' id='downloads'></span>

# 📥 Downloads

{% assign files = site.data.downloads %}

{% if files and files.size > 0 %}
  <div class="downloads-list">
  {% for item in files %}
    <div class="downloads-list__item">
      <h3>{{ item.name }}</h3>
      <p>{{ item.description }}</p>
      <a href="{{ item.file }}" class="downloads-list__link">
        <i class="fas fa-download" aria-hidden="true"></i> Download
      </a>
    </div>
  {% endfor %}
  </div>
{% else %}
  <p style="text-align: center; padding: 3em 0; color: #7a8288;">
    No files available yet. Check back soon!
  </p>
{% endif %}
```

- [ ] **Step 2: Commit**

```bash
git add _pages/downloads.md
git commit -m "feat: add downloads page"
```

---

## Phase 6: Add Styles for New Components

### Task 18: Add SCSS for timeline, project cards, skills cloud, and downloads

**Files:**
- Create: `_sass/_timeline.scss`
- Create: `_sass/_project-cards.scss`
- Create: `_sass/_skills-cloud.scss`
- Create: `_sass/_downloads.scss`
- Modify: `assets/css/main.scss`

- [ ] **Step 1: Create `_sass/_timeline.scss`**

```scss
/* ==========================================================================
   Timeline (experience & education)
   ========================================================================== */

.timeline {
  position: relative;
  padding-left: 2em;
  margin: 1.5em 0;

  &::before {
    content: '';
    position: absolute;
    left: 0;
    top: 0;
    bottom: 0;
    width: 2px;
    background: $border-color;
  }
}

.timeline__item {
  position: relative;
  margin-bottom: 2em;

  &:last-child {
    margin-bottom: 0;
  }
}

.timeline__dot {
  position: absolute;
  left: -2em;
  top: 0.4em;
  width: 12px;
  height: 12px;
  margin-left: -5px;
  border-radius: 50%;
  background: $info-color;
  border: 2px solid lighten($info-color, 20%);
}

.timeline__content {
  padding-left: 0.5em;
}

.timeline__period {
  display: inline-block;
  font-size: $type-size-7;
  color: $gray;
  background: $lighter-gray;
  padding: 0.15em 0.6em;
  border-radius: 3px;
  margin-bottom: 0.3em;
}

.timeline__title {
  margin: 0 0 0.1em;
  font-size: $type-size-5;
  color: $text-color;
}

.timeline__subtitle {
  margin: 0 0 0.5em;
  font-size: $type-size-6;
  font-weight: normal;
  color: $gray;
}

.timeline__highlights {
  margin: 0.5em 0 0;
  padding-left: 1.2em;
  font-size: $type-size-6;

  li {
    margin-bottom: 0.25em;
  }
}

.timeline__notes {
  margin: 0.3em 0 0;
  font-size: $type-size-6;
  color: $gray;
}
```

- [ ] **Step 2: Create `_sass/_project-cards.scss`**

```scss
/* ==========================================================================
   Project Cards
   ========================================================================== */

.project-cards {
  display: grid;
  grid-template-columns: 1fr;
  gap: 1.5em;
  margin: 1.5em 0;

  @include breakpoint($medium) {
    grid-template-columns: 1fr 1fr;
  }
}

.project-card {
  border: 1px solid $border-color;
  border-radius: $border-radius;
  overflow: hidden;
  background: #fff;
  transition: box-shadow 0.2s ease;

  &:hover {
    box-shadow: 0 2px 12px rgba(0, 0, 0, 0.1);
  }
}

.project-card__image {
  width: 100%;
  height: 140px;
  overflow: hidden;

  img {
    width: 100%;
    height: 100%;
    object-fit: cover;
  }

  &--placeholder {
    display: flex;
    align-items: center;
    justify-content: center;
    background: linear-gradient(135deg, $info-color 0%, lighten($info-color, 20%) 100%);

    span {
      font-size: 3em;
      font-weight: bold;
      color: rgba(255, 255, 255, 0.6);
    }
  }
}

.project-card__body {
  padding: 1em;
}

.project-card__title {
  margin: 0 0 0.2em;
  font-size: $type-size-5;
  color: $text-color;
}

.project-card__client {
  display: inline-block;
  font-size: $type-size-7;
  color: $info-color;
  background: transparentize($info-color, 0.9);
  padding: 0.1em 0.5em;
  border-radius: 3px;
  margin-bottom: 0.5em;
}

.project-card__summary {
  font-size: $type-size-6;
  color: $text-color;
  margin: 0.5em 0;
  line-height: 1.5;
}

.project-card__highlights {
  margin: 0.5em 0;
  padding-left: 1.2em;
  font-size: $type-size-6;
  line-height: 1.5;

  li {
    margin-bottom: 0.2em;
  }
}

.project-card__tags {
  display: flex;
  flex-wrap: wrap;
  gap: 0.4em;
  margin-top: 0.8em;
}

.project-card__tag {
  display: inline-block;
  font-size: $type-size-8;
  padding: 0.2em 0.6em;
  background: $lighter-gray;
  color: $gray;
  border-radius: 3px;
}
```

- [ ] **Step 3: Create `_sass/_skills-cloud.scss`**

```scss
/* ==========================================================================
   Skills Cloud
   ========================================================================== */

.skills-cloud {
  margin: 1.5em 0;
}

.skills-cloud__group {
  margin-bottom: 1.2em;
}

.skills-cloud__category {
  margin: 0 0 0.5em;
  font-size: $type-size-6;
  color: $gray;
  text-transform: uppercase;
  letter-spacing: 0.05em;
}

.skills-cloud__tags {
  display: flex;
  flex-wrap: wrap;
  gap: 0.5em;
}

.skills-cloud__tag {
  display: inline-block;
  font-size: $type-size-6;
  padding: 0.35em 0.8em;
  background: $info-color;
  color: #fff;
  border-radius: 20px;
  white-space: nowrap;
}
```

- [ ] **Step 4: Create `_sass/_downloads.scss`**

```scss
/* ==========================================================================
   Downloads
   ========================================================================== */

.downloads-list {
  margin: 1.5em 0;
}

.downloads-list__item {
  padding: 1.2em;
  border: 1px solid $border-color;
  border-radius: $border-radius;
  margin-bottom: 1em;

  h3 {
    margin: 0 0 0.3em;
    font-size: $type-size-5;
  }

  p {
    margin: 0 0 0.5em;
    font-size: $type-size-6;
    color: $gray;
  }
}

.downloads-list__link {
  display: inline-block;
  font-size: $type-size-6;
  color: $info-color;
  text-decoration: none;
  font-weight: bold;

  &:hover {
    text-decoration: underline;
  }
}
```

- [ ] **Step 5: Update `assets/css/main.scss`**

Replace the entire file content. We remove imports for deleted SCSS files, add imports for the four new components, and remove the unused `.paper-box` and `.badge` academic styles:

```scss
---
---

/*
 *    Minimal Mistakes Jekyll Theme
 *
 *  - Michael Rose
 *  - mademistakes.com
 *  - https://twitter.com/mmistakes
 *
*/

@import "vendor/breakpoint/breakpoint"; // media query mixins
@import "variables";
@import "mixins";
@import "vendor/susy/susy";

@import "reset";
@import "base";

@import "utilities";
@import "animations";
@import "masthead";
@import "navigation";

@import "page";
@import "sidebar";

@import "vendor/font-awesome/fontawesome";
@import "vendor/font-awesome/solid";
@import "vendor/font-awesome/brands";

@import "timeline";
@import "project-cards";
@import "skills-cloud";
@import "downloads";

$scroll_offset : 2em;
h1:before, .anchor:before {
    content: '';
    display: block;
    position: relative;
    width: 0;
    height: $scroll_offset;
    margin-top: -$scroll_offset;
}
```

Removed imports: `tables`, `buttons`, `notices`, `footer`, `syntax`, `forms`, `archive`, `vendor/magnific-popup/magnific-popup`, `print`.
Removed styles: `.paper-box` (academic publication display), `.badge` (conference badge) — not used in the new layout.

- [ ] **Step 6: Commit**

```bash
git add _sass/_timeline.scss _sass/_project-cards.scss _sass/_skills-cloud.scss _sass/_downloads.scss assets/css/main.scss
git commit -m "feat: add styles for timeline, project cards, skills cloud, and downloads"
```

---

## Phase 7: Integration & Verification

### Task 19: Full build test and fix any issues

- [ ] **Step 1: Build the site**

```bash
bundle exec jekyll build 2>&1
```

Expected: Build completes without errors. Check the output for any Liquid template errors or missing file warnings.

- [ ] **Step 2: Check for broken includes**

```bash
grep -r "{% include" _pages/ _includes/ _layouts/ | while read line; do
  file=$(echo "$line" | grep -oP '(?<=include )[^%} ]+' | head -1)
  if [ -n "$file" ]; then
    if [ ! -f "_includes/$file" ] && [ ! -f "_includes/$file.html" ]; then
      echo "MISSING: $file"
    fi
  fi
done
```

Expected: No output (no missing includes).

- [ ] **Step 3: Check all referenced data files exist**

```bash
for f in experience education projects skills navigation downloads; do
  if [ ! -f "_data/$f.yml" ]; then
    echo "MISSING: _data/$f.yml"
  fi
done
echo "Data file check complete."
```

Expected: `Data file check complete.` with no MISSING lines.

- [ ] **Step 4: Verify no references to deleted files**

```bash
grep -r "fetch_google_scholar_stats\|google_scholar_crawler\|googlescholar\|researchgate\|pubmed\|orcid\|dblp\|impactstory" _includes/ _layouts/ _pages/ _data/ _config.yml --include="*.html" --include="*.md" --include="*.yml" || echo "All clean"
```

Expected: `All clean` (or only false positives in this very command).

- [ ] **Step 5: Run Jekyll serve for visual check**

```bash
bundle exec jekyll serve --no-watch &
sleep 5
# Check that the server started
curl -s -o /dev/null -w "%{http_code}" http://127.0.0.1:4000/
```

Expected: HTTP `200`.

- [ ] **Step 6: Test all pages return 200**

```bash
for path in "/" "/contact/" "/downloads/"; do
  code=$(curl -s -o /dev/null -w "%{http_code}" "http://127.0.0.1:4000$path")
  echo "$path → $code"
  if [ "$code" != "200" ]; then
    echo "ERROR: $path returned $code"
  fi
done
```

Expected: All three paths return `200`.

- [ ] **Step 7: Stop the server and commit**

```bash
kill %1 2>/dev/null
git add -A
git commit -m "chore: final integration fixes and verification"
```

---

## Success Criteria Checklist

- [ ] `bundle exec jekyll build` runs without errors
- [ ] Homepage (`/`) renders: about text, experience timeline, education timeline, project cards, skills cloud
- [ ] Contact page (`/contact/`) accessible from navigation, renders correctly
- [ ] Downloads page (`/downloads/`) accessible from navigation, shows "no files yet" message
- [ ] No 404 errors on any page
- [ ] Sidebar shows only: avatar, name, bio, description, location, email, github, linkedin, twitter, website
- [ ] Google Scholar crawler directory and workflow completely removed
- [ ] No `fetch_google_scholar_stats` references remain
- [ ] `_config.yml` has no academic social fields (pubmed, orcid, dblp, researchgate, googlescholar, impactstory)
- [ ] `_data/` directory contains: navigation.yml, experience.yml, education.yml, projects.yml, skills.yml, downloads.yml
- [ ] `_pages/` directory contains: home.md, contact.md, downloads.md (no about.md)
- [ ] `_includes/` contains new: timeline.html, project-cards.html, skills-cloud.html
- [ ] Navigation bar shows: Homepage, Contact, Downloads
