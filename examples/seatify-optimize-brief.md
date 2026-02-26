---
app: Seatify
feature: Seating Optimization
style: feature-demo
output_file: seatify-optimize.html
---

# Design Language
- Background: #0a0a0a
- Surface: #141414
- Border: #2a2a2a
- Border light: #222222
- Accent: #d4a574 (warm gold/copper)
- Text: #ffffff
- Text dim: #999999
- Green (capacity/confirmed): #4ade80
- Group Work: #fde68a (light gold)
- Group Family: #22d3ee (cyan)
- Group Friends: #fca5a5 (pink)
- Heading font: Playfair Display (serif)
- Body font: Inter (sans-serif)
- Border radius: 8px
- BETA badge: green background, dark text, uppercase

# Layout
- Stage: 960x620px
- Nav bar: back link (< EVENTS), tabs (CANVAS / GUESTS / RSVPS), event name, BETA badge, avatar
- Canvas area: dark grid background (#0a0a0a with subtle grid lines at #1a1a1a)
- Right sidebar panel (240px wide): READINESS checklist + OPTIMIZE SEATING button

## Tables (simplified to 3 for animation)
- Table 1 (round): center ~(200, 300), radius 80px, capacity 8
  - SVG circle with #222 background stroke, green capacity arc overlay
  - Label: "Table 1" centered, capacity "N/8" below in green
- Table 2 (rectangular): position ~(480, 180), 160x100px, capacity 6
  - Rect with #222 border, rounded corners 6px
  - Label: "Table 2" centered, capacity "N/6" below
- Table 3 (rectangular): position ~(480, 400), 160x100px, capacity 6
  - Same styling as Table 2
  - Label: "Table 3" centered, capacity "N/6" below

## Guest Seats
- 30x30px circles
- Background: #d4a574 (gold/copper)
- Border: 2px solid [group color]
- Dark text initials (#0a0a0a) centered, font-size: 10px, font-weight: 600
- Positioned around table perimeters using trig (round) or edge distribution (rect)

## Sidebar Panel
- Top: "READINESS" label in small caps, gold
- Checklist items with green checkmarks (✓):
  - "12 confirmed guests"
  - "3 tables with capacity"
  - "8 relationships"
- Gold button: "⚡ OPTIMIZE SEATING" — full-width, #d4a574 background, dark text, 8px radius

# Animation Plan

## Style: Before → Action → After

### Before State (mixed seating)
12 guests across 3 tables in mixed groups:

**Table 1 (round, 5/8):** 5 guests around perimeter
- SM (Work/gold) at -90° (top)
- AJ (Friends/pink) at -18°
- LC (Friends/pink) at 54°
- BT (Work/gold) at 126°
- JW (Family/cyan) at 198°

**Table 2 (rect, 4/6):** 4 guests along edges
- CW (Family/cyan) top-left
- NM (Work/gold) top-right
- HR (Friends/pink) bottom-left
- IB (Work/gold) bottom-right

**Table 3 (rect, 3/6):** 3 guests along edges
- ED (Family/cyan) top-center
- ML (Friends/pink) bottom-left
- EW (Family/cyan) bottom-right

Capacity text: "5/8", "4/6", "3/6" in dim text
Table borders: #222 (neutral, no green progress)

### Action (optimize)
1. Animated cursor moves to the "OPTIMIZE SEATING" button
2. Cursor clicks — button flashes briefly
3. Processing overlay fades in over canvas: semi-transparent dark overlay with spinning loader icon and "Optimizing..." text
4. Overlay holds for 1.5s

### After State (optimized seating)
Guests slide to new positions, grouped by color:

**Table 1 (round, 5/8):** All Work group (gold borders)
- SM at -90° (top)
- BT at -18°
- NM at 54°
- IB at 126°
- OC at 198° (new guest appears)

**Table 2 (rect, 4/6):** All Family group (cyan borders)
- JW top-left
- CW top-right
- ED bottom-left
- EW bottom-right

**Table 3 (rect, 5/6):** All Friends group (pink borders)
- AJ top-left
- LC top-right
- HR center
- ML bottom-left
- DT bottom-right (new guest appears)

Visual changes:
- Table capacity arcs animate from 0 to proportional fill (green #4ade80)
- Capacity text updates: "5/8", "4/6", "5/6" in green
- Each table border shifts from neutral #222 to having a colored green capacity arc

Score badge appears:
- Centered on canvas, slightly above middle
- "✓ Optimized — Score: 94%"
- Green background (#4ade80), dark text, rounded pill shape
- Fades in with scale spring animation

### Timing
- 0–1.5s: Before state visible (hold)
- 1.5–2.5s: Cursor moves to button
- 2.5–2.8s: Button click flash
- 2.8–4.3s: Processing overlay (1.5s)
- 4.3–5.5s: Guests slide to new positions (staggered, spring easing)
- 5.5–6.0s: Capacity arcs animate, score badge fades in
- 6.0–8.0s: Hold final state
- 8.0–8.5s: Fade to reset
- Total loop: ~8.5s
- Easing: cubic-bezier(0.34, 1.56, 0.64, 1) for guest movement spring
- Easing: ease-out for overlay and badge
