---
app: Seatify
feature: RSVP Tracking
style: feature-demo
output_file: seatify-rsvp.html
---

# Design Language
- Background: #0a0a0a
- Surface: #141414
- Border: #2a2a2a
- Accent: #d4a574 (warm gold/copper)
- Text: #ffffff
- Text dim: #999999
- Green (confirmed): #4caf50
- Orange (pending): #e8943a
- Red (declined): #e05555
- Heading font: Playfair Display (serif)
- Body font: Inter (sans-serif)
- Border radius: 8px
- Section numbering: italic gold numbers (01, 02, etc.)
- BETA badge: green background, dark text, uppercase

# Layout
- Stage: 960x620px
- Nav bar: back link (< EVENTS), tabs (CANVAS / GUESTS / RSVPS), event name, BETA badge, avatar
- Sub-tabs: RESPONSES / INVITATIONS / SETTINGS
- RSVP Link section: bordered card with URL, Copy button, QR icon
- Stats row: 4 stat cards in a row (Response Rate %, Total Attending, Pending, Declined)
  - Each card: ~200x100px, bordered, large number on top, label below, detail text below label
  - Response Rate: gold/copper number
  - Total Attending: green number
  - Pending: orange number
  - Declined: red number
- Below stats: sub-tabs (Overview, Pending (2), Seating Preferences)
- Recent Responses section: bordered card with response entries

# Animation Plan

## Style: Before → Responses Roll In → Final State

### Before State (empty/early)
- RSVPS tab active in nav
- RESPONSES sub-tab active
- RSVP Link card visible with URL and Copy button
- Stats row showing starting values:
  - Response Rate: 0% (gold)
  - Total Attending: 0 (green)
  - Pending: 18 (orange)
  - Declined: 0 (red)
- Recent Responses section: "No responses yet."

### Action (responses rolling in)
- Guest responses appear one by one in the Recent Responses feed
- Each response slides in from the right with a fade
- As each response appears:
  - Response Rate counter increments
  - Total Attending counter increments (for confirmed)
  - Pending counter decrements
- 6 responses animate in, staggered at ~1s intervals:
  1. James Wilson — Confirmed (Family)
  2. Charlotte White — Confirmed (Family)
  3. Benjamin Taylor — Confirmed (Work)
  4. Harper Reed — Confirmed (Friends)
  5. Ava Johnson — Confirmed (Friends)
  6. Daniel Thompson — Confirmed (Friends)

### After State (final)
- Stats show final values:
  - Response Rate: 89% (animates via counting up)
  - Total Attending: 16 (green)
  - Pending: 2 (orange)
  - Declined: 0 (red)
- Recent Responses feed shows 6 most recent entries
- A subtle "pulse" glow on the Response Rate card to celebrate

### Response Entry Format
Each entry in the Recent Responses feed:
- Left: colored avatar circle with initials (color by group)
- Middle: guest name, organization, "Confirmed" badge in green
- Right: timestamp "Just now" or "2m ago"

### Timing
- 0–2s: Initial empty state visible
- 2–8s: 6 responses animate in (1s apart), counters climb
- 8–9s: Final stat values settle, Response Rate pulses
- 9–14s: Hold final state
- 14–16s: Fade/reset to beginning
- Total loop: 16s
- Easing: cubic-bezier(0.34, 1.56, 0.64, 1) for counter number spring
- Easing: ease-out for slide-in entries
