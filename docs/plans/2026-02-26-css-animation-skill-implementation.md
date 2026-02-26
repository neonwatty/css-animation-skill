# CSS Animation Walkthrough Skill — Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Create a Claude Code skill (`css-animation`) that generates self-contained HTML/CSS animations of app features, with an iterative freeze-inspect-feedback review loop.

**Architecture:** Single SKILL.md file defining a 4-phase workflow: Research (Chrome), Interview (AskUserQuestion), Generate (brief + HTML), Review Loop (freeze-inspect-iterate). The skill is purely instructional — it tells Claude how to behave, with no runtime code.

**Tech Stack:** Claude Code skill (SKILL.md markdown), Claude-in-Chrome MCP tools, AskUserQuestion, Python HTTP server for local serving.

---

### Task 1: Create the skill directory and SKILL.md skeleton

**Files:**
- Create: `~/.claude/skills/css-animation/SKILL.md`

**Step 1: Create the directory**

```bash
mkdir -p ~/.claude/skills/css-animation
```

**Step 2: Write the SKILL.md with frontmatter and overview**

Create `~/.claude/skills/css-animation/SKILL.md` with:

```markdown
---
name: css-animation
description: Generates self-contained HTML/CSS animations of app features for walkthroughs, demos, and onboarding. Researches the target app via Chrome, interviews the user, generates a structured brief and animation HTML file, then enters an iterative freeze-inspect-feedback review loop until the user approves. Use when the user says "css animation", "animate this feature", "create a css walkthrough", "animation walkthrough", or wants to create a CSS-based visual demo of an app feature.
---

# CSS Animation Walkthrough Skill

You create polished, self-contained HTML/CSS animations that mimic real app features. These are used for walkthroughs, onboarding demos, and marketing. They are NOT GIFs — they are resolution-independent, tiny, and easy to iterate on.

## Two Animation Styles

1. **Feature Demo** — Before → Action → After. Shows a single feature transformation (e.g., clicking "Optimize" and watching seats rearrange).
2. **Carousel** — Multi-view. Cycles through several app screens with cross-fade transitions (e.g., Dashboard → Guest List → Canvas → RSVPs).

When invoked, determine which style the user needs during the Interview phase.

## Phases

This skill operates in 4 sequential phases:

1. **Research** — Analyze the live app via Chrome
2. **Interview** — Ask the user focused questions
3. **Generate** — Produce a brief (markdown) + animation (HTML)
4. **Review Loop** — Freeze, inspect, collect feedback, iterate until approved
```

**Step 3: Verify the file exists and the frontmatter is valid**

```bash
head -5 ~/.claude/skills/css-animation/SKILL.md
```

Expected: the `---` delimited frontmatter with name and description.

**Step 4: Commit**

```bash
cd ~/.claude/skills/css-animation && git init && git add SKILL.md && git commit -m "feat: css-animation skill skeleton with frontmatter and overview"
```

Note: if `~/.claude/skills` is not a git repo, skip the commit step — the skill works without git.

---

### Task 2: Write Phase 1 — Research

**Files:**
- Modify: `~/.claude/skills/css-animation/SKILL.md`

**Step 1: Add the Research phase section**

Append to SKILL.md after the Phases overview:

```markdown
---

## Phase 1: Research

Before generating anything, deeply understand the target app's visual system.

### Prerequisites
- Claude-in-Chrome MCP tools must be available
- The target app must be accessible in a browser (running locally or deployed)

### Step 1: Set up Chrome
1. Call `tabs_context_mcp` to check existing tabs
2. Create a new dedicated tab: `tabs_create_mcp`
3. Store this tab ID — use it exclusively for all research

### Step 2: Navigate and explore
1. Navigate to the target app URL
2. If auth is required, ask the user to log in manually, then resume
3. Navigate to the specific feature/page to animate

### Step 3: Extract design language
Use `read_page`, `find`, and `javascript_tool` to extract:

- **Colors**: background, surface, border, accent/primary, text, text-dim
  - Use JS: `getComputedStyle(element).backgroundColor`, etc.
- **Fonts**: heading font-family, body font-family, font weights
- **Spacing**: padding, margins, border-radius values
- **Component styles**: buttons, cards, badges, avatars

Record these as CSS custom properties (e.g., `--bg: #0f0f0f`).

### Step 4: Map layout geometry
For the specific feature to animate:

- **Container dimensions**: width, height, position of major containers
- **Element positions**: absolute coordinates of key elements
- **Relationships**: which elements are inside which containers, z-index stacking

Use `javascript_tool` to extract exact positions:
```js
const el = document.querySelector('.selector');
const rect = el.getBoundingClientRect();
`${rect.left}, ${rect.top}, ${rect.width}, ${rect.height}`
```

### Step 5: Screenshot key states
Take screenshots of:
- The feature in its default/initial state
- Any intermediate states (hover, loading, processing)
- The final/result state

These screenshots serve as reference for the generation phase.
```

**Step 2: Verify the section reads correctly**

Read the file and confirm the Research phase is complete and follows the existing style.

---

### Task 3: Write Phase 2 — Interview

**Files:**
- Modify: `~/.claude/skills/css-animation/SKILL.md`

**Step 1: Add the Interview phase section**

Append after Phase 1:

```markdown
---

## Phase 2: Interview

Ask the user focused questions to define what to animate. Use AskUserQuestion for each question — one at a time, with multiple choice options where possible.

### Question 1: Animation style
```
"What style of animation should this be?"
- Feature Demo (Recommended): Before → Action → After. Shows one feature transformation.
- Carousel: Cycles through multiple app views with cross-fade transitions.
```

### Question 2: Feature identification
```
"What specific feature or flow should the animation show?"
- [Options based on research phase findings]
- Other (user describes)
```

### Question 3: The payoff moment
```
"What's the key moment — the visual 'wow' of this animation?"
- [Options relevant to the chosen feature]
- Other
```

### Question 4: Emphasis
```
"Is there anything specific to emphasize or avoid showing?"
- Show everything as-is
- Emphasize specific elements (user specifies)
- Hide certain elements (user specifies)
```

### Question 5: Output location
```
"Where should I save the animation files?"
- Current directory: [cwd path]
- Desktop
- Other (user specifies path)
```

Stop asking after 3-6 questions — when you have enough context to generate. Don't over-interview.
```

**Step 2: Verify the section**

Read the file and confirm the Interview phase is well-structured.

---

### Task 4: Write Phase 3 — Generate (Brief)

**Files:**
- Modify: `~/.claude/skills/css-animation/SKILL.md`

**Step 1: Add the Generate phase — Brief section**

Append after Phase 2:

```markdown
---

## Phase 3: Generate

This phase produces two artifacts: a **brief** and an **HTML animation**.

### 3a: Generate the Brief

Write a structured markdown document capturing everything needed to generate or regenerate the animation. Save it as `<app>-<feature>-brief.md` in the user's chosen directory.

**Brief format:**

```markdown
---
app: <App Name>
feature: <Feature Name>
style: feature-demo | carousel
output_file: <app>-<feature>.html
---

# Design Language
- Background: <hex>
- Surface: <hex>
- Border: <hex>
- Accent: <hex> (<name>)
- Text: <hex>
- Text dim: <hex>
- Heading font: <font> (serif/sans-serif)
- Body font: <font> (sans-serif)
- Border radius: <N>px
- Additional colors: <name>: <hex> (for groups, statuses, etc.)

# Layout
- Stage: <W>x<H>px
- [Describe container hierarchy]
- [Element positions with exact pixel coordinates]
- [For circular elements: center point, radius]
- [For rectangular containers: origin, width, height]
- [Seat/item sizing: WxH px]

# Animation Plan

## Style: Before → Action → After
(or: ## Style: Carousel with N scenes)

### Before State
- [Describe element positions, visual state, text content]
- [For circular layouts: number of items, angular spacing, starting angle]

### Action (feature-demo only)
- [What the user "does": cursor movement, button click]
- [Intermediate states: loading, processing]

### After State
- [New element positions, calculated trigonometrically for circles]
- [Visual changes: colors, borders, labels, badges]
- [Text content changes]

### Scene N (carousel only)
- [What's visible in this scene]
- [Entry animations for elements]
- [Duration and transition to next scene]

### Timing
- Total loop: <N>s
- [Phase durations]
- Easing: cubic-bezier(0.34, 1.56, 0.64, 1) for spring motion
- Easing: ease-in-out for fades
```

**Show the brief to the user** before generating the HTML. Ask:
```
"Does this brief capture the animation correctly?"
- Yes, generate the animation
- Needs adjustments (user specifies)
```
```

**Step 2: Verify the brief format is complete**

---

### Task 5: Write Phase 3 — Generate (HTML/CSS Animation)

**Files:**
- Modify: `~/.claude/skills/css-animation/SKILL.md`

**Step 1: Add the Generate phase — HTML section with technical guardrails**

Append after the Brief section:

```markdown
### 3b: Generate the HTML/CSS Animation

Using the brief as your spec, generate a self-contained HTML file.

#### Structure

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title><App> – <Feature> Animation</title>
<style>
  @import url('https://fonts.googleapis.com/css2?family=...');
  /* All CSS here — no external stylesheets */
</style>
</head>
<body>
  <!-- Stage container with all visual elements -->
  <div class="stage" id="stage">
    ...
  </div>
  <script>
    // Minimal JS: setTimeout loop controller + class toggling only
    // NO complex animation logic in JS — use CSS transitions
  </script>
</body>
</html>
```

#### Technical Rules (MANDATORY)

1. **Trigonometric positioning for circular layouts.**
   NEVER eyeball positions on circles. ALWAYS calculate:
   ```
   left = centerX + R * cos(angle) - (elementWidth / 2)
   top  = centerY + R * sin(angle) - (elementHeight / 2)
   ```
   Where R = visible radius of container + half the element size (so the element's inner edge touches the container perimeter).

   For N items evenly spaced: `angle_i = startAngle + i * (360 / N)` degrees, starting from -90° (top).

2. **Rectangular edge positioning.**
   For items along rectangle edges, distribute evenly:
   - Top/bottom edges: vary X, fixed Y at edge ± half element height
   - Left/right edges: fixed X at edge ± half element width, vary Y
   - Spacing: `containerWidth / (numItems + 1)` for equal gaps

3. **Self-contained HTML.** No external dependencies except Google Fonts `@import`. All CSS in `<style>`, all JS in `<script>`.

4. **CSS transitions for element movement.** Use `transition` property with class toggling:
   ```css
   .element {
     transition: left 1.2s cubic-bezier(0.34, 1.56, 0.64, 1),
                 top 1.2s cubic-bezier(0.34, 1.56, 0.64, 1);
   }
   ```

5. **JS only for loop control and class toggling:**
   ```js
   function runCycle() {
     stage.classList.remove('state-a', 'state-b');
     // Reset DOM text content
     setTimeout(() => stage.classList.add('state-a'), 2500);
     setTimeout(() => stage.classList.add('state-b'), 3500);
     setTimeout(runCycle, 14000); // Loop
   }
   setTimeout(runCycle, 500);
   ```

6. **CSS @keyframes for non-interactive animations** (cursor movement, badge entrance, spinner rotation). Use `animation` property.

7. **Stagger transitions** with `transition-delay` for sequential element movement.

8. **Default stage: 960×620px**, `border-radius: 12px`, dark theme matching the app's design language.

#### Carousel-Specific Rules

For carousel-style animations:
- Each scene uses `opacity` keyframes to show/hide
- Child elements within each scene use `animation-delay` for staggered entry
- Total loop = sum of all scene durations + transition gaps
- Use `@keyframes` with percentage-based timing for scene visibility

#### Feature-Demo-Specific Rules

For before → action → after animations:
- Define element positions in default CSS (before state)
- Define after-state positions under `.stage.optimized .element { left: ...; top: ...; }`
- Use JS class toggling (`stage.classList.add('optimized')`) to trigger transitions
- Include animated cursor, click effects, processing overlays as needed
```

**Step 2: Verify the technical rules section**

---

### Task 6: Write Phase 4 — Review Loop

**Files:**
- Modify: `~/.claude/skills/css-animation/SKILL.md`

**Step 1: Add the Review Loop phase**

Append after Phase 3:

```markdown
---

## Phase 4: Review Loop

This is the core quality mechanism. After generating the animation, enter an iterative review loop until the user approves.

### Step 1: Serve the animation locally

1. Check if a Python HTTP server is already running on port 8765:
   ```bash
   lsof -i :8765
   ```
2. If not running, start one in the output directory:
   ```bash
   cd <output-directory> && python3 -m http.server 8765 &
   ```
3. Create a new Chrome tab (or reuse the research tab)
4. Navigate to: `http://localhost:8765/<filename>.html?v=<timestamp>`
   - ALWAYS append `?v=<Date.now()>` to bust the cache after edits

### Step 2: Freeze and inspect each key state

Inject JavaScript to stop the animation and set a specific state:

```js
// Stop all timers
var id = window.setTimeout(function(){}, 0);
while (id--) { window.clearTimeout(id); }

// Reset to target state
var stage = document.getElementById('stage');
stage.classList.remove('clicking', 'processing', 'optimized', 'complete');

// For BEFORE state: leave classes removed
// For AFTER state: stage.classList.add('optimized', 'complete');
// For intermediate states: add appropriate classes

// Hide cursor animation
var cursor = document.querySelector('.cursor');
if (cursor) { cursor.style.animation = 'none'; cursor.style.opacity = '0'; }
```

Take a screenshot after setting each state.

### Key states to inspect

**Feature Demo:**
1. **Before state** — Elements in initial positions. Check: are all elements visible? Positioned correctly on container perimeters? Colors match the app?
2. **After state** — Elements in final positions. Check: are elements on perimeters? Correct grouping? Labels/badges positioned correctly?
3. **Badge/overlay** — If there's a completion badge, check its position doesn't obscure important elements.

**Carousel:**
1. **Each scene** — Set scene visibility via JS. Check: layout matches the app? Elements properly styled? Entry animations look natural?

### Step 3: Collect feedback

For each state, use AskUserQuestion:

```
"How does the [BEFORE/AFTER/Scene N] state look?"
Options:
- Looks good
- Element positions are off (seats, items not on perimeters)
- Colors or styling need adjustment
- Layout or spacing needs changes
- Text content is wrong
- [Other — user describes the issue]
```

If multiSelect is appropriate (multiple issues), enable it.

### Step 4: Fix and re-inspect

When the user reports an issue:

1. **Read the current HTML** to understand the CSS structure
2. **Identify the root cause** (e.g., positions not trigonometrically calculated)
3. **Apply the fix** using Edit tool
4. **Update the brief** if the fix reveals a spec error
5. **Re-serve** with new cache-buster: navigate to `?v=<new-timestamp>`
6. **Re-freeze** at the affected state
7. **Re-screenshot** and ask again

Common fixes:
- **Seats off perimeter**: Recalculate with trig formulas
- **Colors wrong**: Update CSS custom properties
- **Layout shifted**: Check container positions, absolute positioning coordinates
- **Text wrong**: Update HTML text content
- **Badge obscuring elements**: Adjust badge position or z-index

### Step 5: Full playthrough approval

Once ALL frozen states are individually approved:

1. Reload the page with cache-buster to restart the animation loop
2. Let the user watch the full animation (do NOT freeze it)
3. Ask:
   ```
   "How does the full animation look in motion?"
   Options:
   - Approved — animation looks great!
   - Timing needs adjustment (too fast/slow)
   - Transitions need work (easing, stagger)
   - Something else needs fixing
   ```

4. If **Approved** → the review loop ends. Inform the user where the files are saved.
5. If **not approved** → apply the specific feedback, re-serve, and ask again.

### Review Loop Principle

**Inspect static states first, then evaluate motion.**
Positioning and layout bugs are easier to see in frozen frames. Timing and easing bugs only appear in motion. Fix the easy stuff first.
```

**Step 2: Verify the review loop section is complete**

---

### Task 7: Add Error Handling and Edge Cases

**Files:**
- Modify: `~/.claude/skills/css-animation/SKILL.md`

**Step 1: Add error handling section**

Append after Phase 4:

```markdown
---

## Error Handling

- **App requires login:** Stop and ask the user to log in manually in the Chrome tab. Resume research after they confirm.
- **Chrome tools unavailable:** Inform the user that Claude-in-Chrome is required for the Research phase. Offer to skip to Interview if they can describe the app.
- **HTTP server port in use:** Try port 8766, 8767, etc. Use `lsof -i :<port>` to check.
- **Browser cache shows stale version:** Always cache-bust with `?v=<timestamp>`. If still stale, hard-refresh via Chrome DevTools or try a new tab.
- **User reports issue you can't see:** Ask the user to describe the issue in detail. Use `read_page` or `javascript_tool` to inspect the live DOM.
- **Animation file too large:** If the HTML exceeds ~50KB, simplify: reduce element count, combine similar styles, remove redundant keyframes.
- **Circular positioning looks wrong:** Double-check the formula. Common mistakes: forgetting to convert degrees to radians, wrong center point, wrong radius (should be container radius + half element size).

## Tips for High-Quality Animations

- **Match the app's personality.** Dark theme apps get dark animations. Playful apps get bouncy easing. Corporate apps get subtle transitions.
- **Less is more.** 14-18 moving elements is plenty. Don't try to recreate every pixel.
- **Spring easing sells it.** `cubic-bezier(0.34, 1.56, 0.64, 1)` makes elements feel physical.
- **Stagger everything.** Sequential `transition-delay` (0.04-0.08s apart) looks much better than simultaneous movement.
- **Hold the payoff.** After the big transformation, hold the final state for at least 3-4 seconds so the viewer can absorb it.
- **Labels and badges appear last.** They're the "narration" — let the visual change happen first.
```

**Step 2: Verify**

---

### Task 8: Final review and test the skill

**Files:**
- Read: `~/.claude/skills/css-animation/SKILL.md`

**Step 1: Read the complete SKILL.md**

Read the entire file end-to-end. Verify:
- Frontmatter has correct `name` and `description`
- All 4 phases are present and in order
- Technical guardrails are in the Generate section
- Review loop has concrete JS injection snippets
- AskUserQuestion examples are actionable
- Error handling covers the known pitfalls
- No TODO or placeholder text remains

**Step 2: Verify skill appears in Claude Code**

The skill should appear in Claude Code's skill list. To verify, check that:
- The file exists at `~/.claude/skills/css-animation/SKILL.md`
- The frontmatter `name` matches `css-animation`
- The `description` includes trigger phrases

**Step 3: Test invocation**

Start a new Claude Code conversation and say "css animation for [some app]". The skill should be invoked and begin Phase 1 (Research). Verify it:
- Creates a Chrome tab
- Starts asking about the target app
- Follows the phase workflow

If anything is off, fix and re-test.
