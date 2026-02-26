---
name: css-animation
description: Generates self-contained HTML/CSS animations of app features for walkthroughs, demos, and onboarding. Researches the target app via Chrome, interviews the user, generates a structured brief and animation HTML file, then enters an iterative freeze-inspect-feedback review loop until the user approves. Use when the user says "css animation", "animate this feature", "create a css walkthrough", "animation walkthrough", or wants to create a CSS-based visual demo of an app feature.
---

# CSS Animation Walkthrough Skill

You create polished, self-contained HTML/CSS animations that mimic real app features. These are used for walkthroughs, onboarding demos, and marketing. They are NOT GIFs — they are resolution-independent, tiny file size, and easy to iterate on.

## Two Animation Styles

1. **Feature Demo** — Before → Action → After. Shows a single feature transformation (e.g., clicking "Optimize" and watching seats rearrange).
2. **Carousel** — Multi-view. Cycles through several app screens with cross-fade transitions (e.g., Dashboard → Guest List → Canvas → RSVPs).

Determine which style the user needs during the Interview phase.

## Phases

```
Phase 1: Research → Phase 2: Interview → Phase 3: Generate → Phase 4: Review Loop
```

---

## Phase 1: Research

Before generating anything, deeply understand the target app's visual system by navigating it in Chrome.

### Prerequisites
- Claude-in-Chrome MCP tools must be available
- The target app must be accessible in a browser (running locally or deployed)

### Step 1: Set up Chrome
1. Call `tabs_context_mcp` to check existing tabs
2. Create a new dedicated tab: `tabs_create_mcp`
3. Store this tab ID — use it exclusively for all subsequent browser operations

### Step 2: Navigate and explore
1. Navigate to the target app URL
2. If auth is required, ask the user to log in manually, then resume
3. **If targeting mobile or both:** Resize the browser to mobile viewport (390×844px) using `resize_window` before navigating to the feature. This ensures you research the app's actual mobile layout, not the desktop version squeezed down. If targeting **both**, research desktop first at full size, then resize to mobile and re-extract the mobile layout.
4. Navigate to the specific feature/page to animate
5. Take exploratory screenshots to understand the layout

### Step 3: Extract design language
Use `read_page`, `find`, and `javascript_tool` to extract:

- **Colors**: background, surface, border, accent/primary, text, text-dim
  ```js
  // Example: extract computed styles
  const body = getComputedStyle(document.body);
  JSON.stringify({
    bg: body.backgroundColor,
    color: body.color,
    fontFamily: body.fontFamily
  })
  ```
- **Fonts**: heading font-family, body font-family, font weights used
- **Spacing**: padding, margins, border-radius values
- **Component styles**: buttons, cards, badges, avatars, sidebar panels

Record these as CSS custom properties (e.g., `--bg: #0f0f0f`).

### Step 4: Map layout geometry
For the specific feature to animate:

- **Container dimensions**: width, height, position of major containers
- **Element positions**: absolute coordinates of key elements
- **Relationships**: which elements are inside which, z-index stacking
- **Circular elements**: center point and radius
- **Rectangular elements**: origin (top-left), width, height

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
- The final/result state after the feature action completes

These screenshots serve as visual reference for generation.

---

## Phase 2: Interview

Ask the user focused questions to define what to animate. Use AskUserQuestion for each — one at a time, with multiple choice options.

### Question 1: Animation style
```
"What style of animation should this be?"
- Feature Demo (Recommended): Before → Action → After. Shows one feature transformation.
- Carousel: Cycles through multiple app views with cross-fade transitions.
```

### Question 2: Target size
```
"What size should the animation target?"
- Desktop only (960×620px) — for landing pages, marketing, desktop walkthroughs
- Mobile only (360×640px) — for in-app tour tooltips, mobile onboarding
- Both — generates two files from the same brief, one per viewport
```

When **Mobile** or **Both** is selected:
- The mobile variant uses a **360×640px** stage (portrait, matching common phone viewports)
- Simplify the layout: fewer elements (8–12 max), larger relative sizing, no sidebars
- Drop elements that don't translate to small screens (wide toolbars, multi-column layouts)
- Prefer vertical stacking over horizontal layouts
- Increase font sizes relative to stage (minimum 11px body, 14px headings)
- File naming: `<app>-<feature>.html` (desktop), `<app>-<feature>-mobile.html` (mobile)

### Question 3: Feature identification
```
"What specific feature or flow should the animation show?"
- [Options based on research phase findings]
- Other (user describes)
```

### Question 4: The payoff moment
```
"What's the key moment — the visual 'wow' of this animation?"
- [Options relevant to the chosen feature]
- Other
```

### Question 5: Emphasis
```
"Is there anything specific to emphasize or avoid showing?"
- Show everything as-is
- Emphasize specific elements (user specifies)
- Hide certain elements (user specifies)
```

### Question 6: Output location
```
"Where should I save the animation files?"
- Current directory: [show cwd path]
- Desktop
- Other (user specifies path)
```

Stop after 3-6 questions — when you have enough context to generate. Don't over-interview.

---

## Phase 3: Generate

This phase produces two artifacts: a **brief** (markdown) and an **HTML animation** file.

### 3a: Generate the Brief

Write a structured markdown document capturing everything needed to generate or regenerate the animation. Save as `<app>-<feature>-brief.md` in the user's chosen directory.

**Brief format:**

```markdown
---
app: <App Name>
feature: <Feature Name>
style: feature-demo | carousel
target: desktop | mobile | both
output_file: <app>-<feature>.html
output_file_mobile: <app>-<feature>-mobile.html  # only when target is mobile or both
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
- Additional colors: <name>: <hex> (for groups, statuses, categories, etc.)

# Layout (Desktop)
- Stage: 960x620px
- [Container hierarchy description]
- [Element positions with exact pixel coordinates]
- [Circular elements: center point (x,y), radius R]
- [Rectangular containers: origin (x,y), width, height]
- [Item sizing: WxH px]

# Layout (Mobile)  <!-- only when target is mobile or both -->
- Stage: 360x640px
- [Simplified container hierarchy — no sidebars, single-column]
- [Reduced element count: 8–12 elements max]
- [Element positions recalculated for smaller stage]
- [Larger relative element sizing: e.g., 36x36px avatars instead of 30x30px]
- [Elements or sections omitted from desktop version and why]

# Animation Plan

## Style: Before → Action → After
(or: ## Style: Carousel with N scenes)

### Before State
- [Element positions, visual state, text content]
- [For circular layouts: N items, angular spacing, starting angle]

### Action (feature-demo only)
- [User action: cursor movement, button click]
- [Intermediate states: loading, processing]

### After State
- [New positions, trigonometrically calculated for circles]
- [Visual changes: colors, borders, labels, badges]
- [Text content changes]

### Scene N (carousel only)
- [What's visible in this scene]
- [Entry animations for elements]
- [Duration and transition to next scene]

### Timing
- Total loop: <N>s
- [Phase-by-phase durations]
- Easing: cubic-bezier(0.34, 1.56, 0.64, 1) for spring motion
- Easing: ease-in-out for fades
```

**Show the brief to the user** before generating HTML:
```
"Does this brief capture the animation correctly?"
- Yes, generate the animation
- Needs adjustments (user specifies)
```

Iterate on the brief until the user approves it, then proceed to 3b.

### 3b: Generate the HTML/CSS Animation

Using the brief as your spec, generate a self-contained HTML file. Save as `<app>-<feature>.html` in the user's chosen directory.

#### File Structure

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title><App> – <Feature> Animation</title>
<style>
  @import url('https://fonts.googleapis.com/css2?family=...');
  /* ALL CSS here — no external stylesheets */
</style>
</head>
<body>
  <div class="stage" id="stage">
    <!-- All visual elements -->
  </div>
  <script>
    // Minimal JS: setTimeout loop controller + class toggling ONLY
  </script>
</body>
</html>
```

#### Technical Rules (MANDATORY)

**1. Trigonometric positioning for circular layouts.**

NEVER eyeball positions on circles. ALWAYS calculate:
```
left = centerX + R * cos(angle_radians) - (elementWidth / 2)
top  = centerY + R * sin(angle_radians) - (elementHeight / 2)
```
Where:
- `R` = visible radius of container + (elementSize / 2), so the element's inner edge touches the container perimeter
- For N items evenly spaced: `angle_i = startAngle + i * (2 * PI / N)`, starting from `-PI/2` (top center)
- Convert degrees to radians: `radians = degrees * PI / 180`

Example for 7 items on a circle with center (140, 160) and R=95, items are 30x30px:
```
Item 0: angle = -90°  → left = 140 + 95*cos(-90°) - 15 = 125,  top = 160 + 95*sin(-90°) - 15 = 50
Item 1: angle = -38.6° → left = 140 + 95*cos(-38.6°) - 15 = 199, top = 160 + 95*sin(-38.6°) - 15 = 86
...etc for all N items
```

**2. Rectangular edge positioning.**

For items along rectangle edges, distribute evenly:
- Top/bottom edges: vary X at equal intervals, Y at edge ± (elementHeight / 2)
- X spacing: `containerLeft + containerWidth * (i + 1) / (numItemsOnEdge + 1)`
- Left/right edges: same logic, varying Y

**3. Self-contained HTML.** No external dependencies except Google Fonts `@import`. All CSS in `<style>`, all JS in `<script>`. Zero build step.

**4. CSS transitions for element movement.**
```css
.element {
  position: absolute;
  transition: left 1.2s cubic-bezier(0.34, 1.56, 0.64, 1),
              top 1.2s cubic-bezier(0.34, 1.56, 0.64, 1);
}
```

**5. JS only for loop control and class toggling.**

**CRITICAL: Never use inline styles** (`element.style.opacity = '1'`) in JS reset/update functions. Inline styles override CSS class rules and cause state bugs (e.g., a "hidden" element that stays visible because an inline style trumps the CSS class). Use class toggling exclusively.

```js
const stage = document.getElementById('stage');
function runCycle() {
  // Reset — class toggling only, no inline styles
  stage.classList.remove('clicking', 'processing', 'optimized', 'complete');
  // Reset text content to before-state values

  setTimeout(() => stage.classList.add('clicking'), 500);
  setTimeout(() => {
    stage.classList.remove('clicking');
    stage.classList.add('processing');
  }, 900);
  setTimeout(() => {
    stage.classList.remove('processing');
    stage.classList.add('optimized');
    // Update text content to after-state values
  }, 1500);
  setTimeout(() => stage.classList.add('complete'), 3000);
  setTimeout(runCycle, 6000); // Loop
}
setTimeout(runCycle, 300);
```

**Default to fast timing.** Users almost always want the animation faster than your first instinct. Start with snappy defaults:
- Before state hold: 0.3–0.5s (just long enough to register)
- Stagger between elements: 0.3–0.5s
- After state hold: 2–3s
- Total loop: 5–8s
- Timing will be tuned iteratively in Phase 4 — start fast and slow down only if asked.

**6. CSS @keyframes for non-interactive animations** — cursor movement, badge entrance, spinner rotation. Use the `animation` property on the relevant elements.

**7. Stagger transitions** with `transition-delay` on individual elements (0.04–0.08s apart) for sequential movement that feels natural.

**8. Stage dimensions by target.**
- **Desktop:** 960×620px, `border-radius: 12px`
- **Mobile:** 360×640px, `border-radius: 16px` (portrait orientation)
- Always match the app's design language (dark/light theme, colors). Adjust proportions based on the app's actual layout at that viewport.

**9. Hidden elements must use `visibility: hidden` alongside `opacity: 0`.** On dark backgrounds, JPEG screenshot compression creates visible artifacts for opacity-0 elements — they appear as faint ghosts. Always pair:
```css
.element-hidden {
  opacity: 0;
  visibility: hidden;
  transition: opacity 0.5s ease-out, visibility 0s 0.5s; /* visibility delays on hide */
}
.element-hidden.visible {
  opacity: 1;
  visibility: visible;
  transition: opacity 0.5s ease-out, visibility 0s 0s; /* visibility instant on show */
}
```
For elements that should be completely removed from flow when hidden (like placeholder text), use `display: none` via a parent class toggle:
```css
.stage.active .placeholder { display: none; }
```

**10. Verify content fits the stage.** After generating, inject JS to check that all content fits within the stage bounds before entering the review loop:
```js
var content = document.querySelector('.content');
var stage = document.getElementById('stage');
JSON.stringify({ stageH: stage.offsetHeight, contentH: content.scrollHeight, fits: content.scrollHeight <= stage.offsetHeight - content.offsetTop })
```
If content overflows, compact spacing (reduce padding, margins, font sizes) or increase stage height before proceeding to review. Don't waste review cycles on overflow bugs.

#### Carousel-Specific Rules

- Each scene uses `opacity` keyframes to show/hide (fade in → hold → fade out)
- Child elements within each scene use `animation-delay` for staggered entry
- Total loop duration = sum of all scene durations + transition gaps
- Use `@keyframes` with percentage-based timing for scene visibility
- Each scene is a container `div` with `position: absolute; inset: 0;`

#### Feature-Demo-Specific Rules

- Default CSS defines "before" positions for all elements
- After-state positions go under `.stage.optimized .element { left: ...; top: ...; }`
- JS class toggling (`stage.classList.add('optimized')`) triggers CSS transitions
- Include: animated cursor (CSS keyframes), click flash effect, processing spinner overlay
- Include: completion badge that animates in after the transformation
- Include: text content updates (counters, labels) via JS in the setTimeout chain

#### Mobile-Specific Rules

When generating mobile animations (360×640px stage):

- **Simplify aggressively.** Show the essence of the feature, not every UI element. A desktop animation might show a full toolbar + sidebar + canvas; the mobile version shows just the core interaction.
- **8–12 elements max.** Small stages get cluttered fast. Cut decorative elements first, then secondary UI.
- **No animated cursor.** Mobile is touch — replace cursor animations with a tap indicator (a brief circle pulse at the tap point).
- **Larger relative sizing.** Elements should be proportionally larger relative to the stage. If desktop uses 30×30px avatars on a 960px stage (3.1%), mobile should use 36×36px on 360px (10%).
- **Single-column layouts.** No sidebars. Stack content vertically.
- **Bottom-anchored actions.** Buttons and CTAs at the bottom of the stage, matching mobile app conventions.
- **Skip nav bars if tight on space.** The animation doesn't need to recreate the full app chrome — focus on the feature area.

When generating **both** variants from a single brief:
- Generate the desktop version first (`<app>-<feature>.html`)
- Then generate the mobile version (`<app>-<feature>-mobile.html`) as a separate file
- The mobile version is a purposeful redesign for the smaller stage, not a scaled-down copy
- Both files share the same design language (colors, fonts) but have independent layouts
- Review each variant independently in Phase 4

---

## Phase 4: Review Loop

This is the core quality mechanism. After generating the animation HTML, enter an iterative review loop until the user approves.

### Step 1: Serve the animation locally

1. Check if a Python HTTP server is already running:
   ```bash
   lsof -i :8765
   ```
2. If not running, start one in the output directory:
   ```bash
   python3 -m http.server 8765 --directory <output-directory> &
   ```
3. Use the Chrome tab from Phase 1 (or create a new one)
4. **If reviewing a mobile animation:** Resize the browser to mobile viewport (390×844px) using `resize_window` before navigating. This ensures you see the animation at the intended scale, not stretched across a desktop window.
5. Navigate to: `http://localhost:8765/<filename>.html?v=<timestamp>`

**ALWAYS append `?v=<Date.now()>` to bust the cache after every edit.**

**When reviewing both variants:** Review the desktop animation first at full browser size, then resize to mobile and review the mobile variant. Keep reviews separate — don't mix feedback between variants.

### Step 2: Freeze and inspect each key state

Inject JavaScript to stop the animation and set a specific state:

```js
// Stop ALL pending timers
var id = window.setTimeout(function(){}, 0);
while (id--) { window.clearTimeout(id); }

// Reset stage
var stage = document.getElementById('stage');
stage.classList.remove('clicking', 'processing', 'optimized', 'complete');

// Hide cursor
var cursor = document.querySelector('.cursor');
if (cursor) { cursor.style.animation = 'none'; cursor.style.opacity = '0'; }

// Hide badges/overlays
var badge = document.querySelector('.optimized-badge');
if (badge) { badge.style.display = 'none'; }
```

Then set the target state:
- **BEFORE state**: leave all classes removed (default CSS positions apply)
- **AFTER state**: `stage.classList.add('optimized');` and update text content
- **COMPLETE state**: `stage.classList.add('optimized', 'complete');` and show badge

Take a screenshot after setting each state. Present to the user.

### Key states to inspect

**Feature Demo:**
1. **Before state** — Are all elements visible? Positioned correctly on container perimeters? Colors match the app?
2. **After state** — Are elements on perimeters? Correct grouping? Labels and badges positioned correctly?
3. **Badge/overlay** — Does the completion badge obscure important elements?

**Carousel:**
1. **Each scene** — Layout matches the app? Elements properly styled? Text content correct?

### Step 3: Collect feedback

For EACH inspected state, use AskUserQuestion:

```
"How does the [BEFORE / AFTER / Scene N] state look?"
Options:
- Looks good
- Element positions are off (not on perimeters, wrong placement)
- Colors or styling need adjustment
- Layout or spacing needs changes
- Text content is wrong
- [Other]
```

Enable `multiSelect: true` if multiple issues are likely.

### Step 4: Fix and re-inspect

When the user reports an issue:

1. **Read the current HTML** to understand the CSS structure
2. **Identify the root cause:**
   - Positions off perimeter → recalculate with trig formulas
   - Colors wrong → update CSS custom properties
   - Layout shifted → check container positions and absolute coordinates
   - Text wrong → update HTML content
   - Badge obscuring elements → adjust badge position or z-index
3. **Apply the fix** using the Edit tool
4. **Update the brief** if the fix reveals a spec error
5. **Re-serve** with new cache-buster: `?v=<new-timestamp>`
6. **Re-freeze** at the affected state using the same JS injection
7. **Re-screenshot** and ask the user again

Repeat Steps 3-4 until the user says "Looks good" for every state.

### Step 5: Full playthrough and timing tuning

Once ALL frozen states are individually approved:

1. Reload the page (with cache-buster) to restart the animation loop
2. Let the user watch the full animation — do NOT freeze it
3. Ask:
   ```
   "How does the full animation look in motion?"
   Options:
   - Approved — looks great!
   - Timing needs adjustment
   - Transitions need work (easing feels wrong, stagger off)
   - Something else needs fixing
   ```
4. If **Approved** → done. Inform the user where files are saved (brief + HTML).
5. If **Timing needs adjustment** → enter the **Timing Tuning Loop** (below).
6. If other feedback → apply it, re-serve, re-ask.

#### Timing Tuning Loop

**Timing is always iterative.** Expect 2-4 rounds of timing adjustment — this is normal, not a sign of a bad first attempt. The goal is to converge quickly by asking specific questions.

When the user says timing needs adjustment, ask **specifically which phases**:

```
"What about the timing needs adjustment?"
Options (multiSelect: true):
- Initial state too long (before the action starts)
- Initial state too short (need more time to see the 'before')
- Action/stagger too slow (elements appear too slowly)
- Action/stagger too fast (can't follow what's happening)
- Final hold too long (loop feels sluggish)
- Final hold too short (not enough time to absorb the result)
```

Apply the changes, reload with cache-buster, and re-ask:

```
"How does the updated timing feel?"
Options:
- Approved — looks great!
- Still needs adjustment
```

If "Still needs adjustment" → ask the specific phase question again. Repeat until approved.

**Timing tuning tips:**
- Adjust in meaningful increments (halve or double, don't tweak by 50ms)
- If the user says "faster" without specifics, shorten the before-state hold and final hold first — the action stagger is usually fine
- Total loop under 6s feels snappy; over 12s feels sluggish
- The before-state rarely needs more than 0.5s — the viewer understands the "empty" state immediately

### Review Loop Principle

**Inspect static states first, then evaluate motion.**

Positioning and layout bugs are much easier to see in frozen frames. Timing and easing bugs only appear during playback. Fix the visual bugs first, then tune the motion.

---

## Error Handling

- **App requires login:** Stop and ask the user to log in manually in the Chrome tab. Resume research after they confirm.
- **Chrome tools unavailable:** Inform the user that Claude-in-Chrome is required for Phase 1 (Research). Offer to skip to Interview if they can describe the app verbally.
- **HTTP server port in use:** Try ports 8766, 8767, etc. Check with `lsof -i :<port>`.
- **Browser cache shows stale version:** Always cache-bust with `?v=<timestamp>`. If still stale, try a new Chrome tab.
- **User reports an issue you can't reproduce:** Ask the user to describe it in detail. Use `read_page` or `javascript_tool` to inspect the live DOM and computed styles.
- **Circular positioning looks wrong:** Double-check the formula. Common mistakes:
  - Forgetting to convert degrees to radians
  - Wrong center point (remember: center = origin + half-dimension)
  - Wrong radius (should be container visible radius + half element size)
  - Forgetting to subtract half element size from the calculated position

## Tips for High-Quality Animations

- **Match the app's personality.** Dark theme apps get dark animations. Playful apps get bouncy easing. Corporate apps get subtle transitions.
- **Less is more.** 14-20 moving elements is plenty. Don't try to recreate every pixel of the real app.
- **Spring easing sells it.** `cubic-bezier(0.34, 1.56, 0.64, 1)` makes elements feel physical and alive.
- **Stagger everything.** Sequential `transition-delay` (0.04–0.08s per element) looks far better than simultaneous movement.
- **Hold the payoff — but not too long.** After the transformation, hold the final state for 2-3 seconds. Longer than that and the loop feels sluggish. The viewer absorbs the result faster than you think.
- **Labels and badges appear last.** They narrate the result — let the visual change happen first.
- **Design language is king.** Getting the colors, fonts, and border-radius right makes a rough layout still feel "like the app." Getting the layout perfect with wrong colors feels off.
