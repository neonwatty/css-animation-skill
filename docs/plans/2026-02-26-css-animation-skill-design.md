# CSS Animation Walkthrough Skill — Design

## Summary

A Claude Code skill that generates self-contained HTML/CSS animations of app features for use in walkthroughs, demos, and onboarding. Differentiates from GIF-based walkthroughs by being resolution-independent, tiny file size, controllable, and easy to iterate on. Uses an iterative freeze-inspect-feedback review loop to ensure quality.

## Workflow

```
Phase 1: Research → Phase 2: Interview → Phase 3: Generate → Phase 4: Review Loop
```

### Phase 1: Research

Navigate the live app via Claude-in-Chrome to extract:
- **Design language**: colors, fonts, border styles, spacing, border-radius
- **Layout geometry**: element positions, dimensions, container hierarchy
- **Key screens**: screenshots of the feature in its various states
- **Interactive elements**: buttons, sidebar controls, modals relevant to the animation

Output: internal understanding of the app's visual system.

### Phase 2: Interview

Ask the user 3-6 focused questions via AskUserQuestion:
- What feature or flow to animate
- Animation style: **feature demo** (before → action → after) or **carousel** (multi-view)
- What the "payoff moment" is (the key visual transformation)
- What to emphasize or de-emphasize
- Where to save output files

### Phase 3: Generate

Produce two artifacts:

#### 3a. Brief (markdown)

A structured document capturing everything needed to generate or regenerate the animation:

```markdown
---
app: <App Name>
feature: <Feature Name>
style: feature-demo | carousel
output_file: <filename>.html
---

# Design Language
- Background, surface, border, accent colors
- Heading and body fonts
- Border radius, spacing conventions

# Layout
- Stage dimensions and structure
- Element positions with exact coordinates
- Table/container geometry (center points, radii, dimensions)
- Seat/item sizing

# Animation Plan
## Style: <Before → Action → After | Carousel>

### State descriptions
- Element positions per state (trigonometrically calculated for circles)
- Visual changes (colors, borders, labels)

### Timing
- Duration per phase
- Total loop length
- Easing curves
```

#### 3b. HTML/CSS Animation (self-contained)

- Single HTML file with embedded `<style>` block
- Google Fonts as only external dependency
- CSS transitions with class toggling via minimal JS
- JS setTimeout chains for animation loop controller
- No frameworks or libraries

### Phase 4: Review Loop

#### Step 1: Serve locally
- Start `python3 -m http.server <port>` in the output directory
- Navigate Chrome to `http://localhost:<port>/<file>?v=<timestamp>`

#### Step 2: Freeze and inspect each key state
- Inject JS to clear all timers and stop CSS animations
- For each key state (Before, Action, After / each carousel scene):
  - Set DOM to that state via JS class toggling
  - Take a screenshot via Chrome tools
  - Present to user

#### Step 3: Collect feedback via AskUserQuestion
- Per-state feedback with targeted options:
  - "Looks good"
  - Common issues (positioning off, colors wrong, layout needs changes)
  - Free-text for specific corrections

#### Step 4: Fix and re-inspect
- Apply fixes to HTML/CSS
- Update the brief if the fix reveals a spec issue
- Re-serve with cache-buster, re-freeze, re-screenshot
- Ask again for the affected state

#### Step 5: Full playthrough approval
- Once all frozen states approved, user watches the full animation loop
- Final feedback: timing, transitions, overall feel
- If approved → done. If not → iterate on specific feedback.

## Technical Guardrails

1. **Trigonometric positioning for circular layouts**: `left = centerX + R*cos(angle) - halfWidth`, `top = centerY + R*sin(angle) - halfHeight`. Never eyeball.
2. **Rectangular edge positioning**: distribute evenly along edges with equal spacing from corners.
3. **Self-contained HTML**: no external dependencies except Google Fonts.
4. **Local HTTP server**: always serve via `python3 -m http.server`, never `file://` URLs.
5. **Cache-busting**: append `?v=<timestamp>` on every reload.
6. **CSS transitions over JS animation**: prefer `transition` with class toggling. Use `cubic-bezier(0.34, 1.56, 0.64, 1)` for spring-like motion.
7. **Animation loop via setTimeout**: JS setTimeout chains for loop controller, CSS transitions for element movement.
8. **Default stage**: 960x620px, 12px border-radius, dark theme. Adjust based on app proportions.

## Invocation

**Trigger phrases**: "css animation", "animate this feature", "create a css walkthrough", "animation walkthrough"

**Inputs**: target app URL (required), feature to animate (can be discovered during interview)

**Outputs**:
1. `<brief>.md` — structured brief (user chooses save location)
2. `<animation>.html` — self-contained animation file (user chooses save location)
