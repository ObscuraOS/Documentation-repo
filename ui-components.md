# UI Components

Obscura OS features a distinctive cyberpunk-inspired design language. This document describes the visual system and component architecture used throughout the application.

---

## Design Language

### Visual Identity

Obscura OS draws inspiration from cyberpunk aesthetics and retro-futuristic operating systems:

- **Theme**: Dark, immersive, high-contrast
- **Mood**: Technical, secure, professional yet distinctive
- **Era**: Near-future cyberpunk with CRT/terminal influences

### Color Palette

```
PRIMARY COLORS
├── Background Dark:    #071019 (Deep Navy)
├── Background Alt:     #0b0d12 (Near Black)
├── Accent Gold:        #FFD86B (Primary Actions)
└── Accent Teal:        #7FFFD4 (Success/Info)

TEXT COLORS
├── Primary Text:       #FFFFFF (White)
├── Secondary Text:     #B0B0B0 (Gray)
├── Muted Text:         #666666 (Dark Gray)
└── Accent Text:        #FFD86B (Highlighted)

STATUS COLORS
├── Success:            #4ADE80 (Green)
├── Warning:            #FACC15 (Yellow)
├── Error:              #EF4444 (Red)
└── Info:               #7FFFD4 (Teal)

BORDER/GLOW
├── Border Default:     rgba(255, 216, 107, 0.2)
├── Border Hover:       rgba(255, 216, 107, 0.4)
├── Glow Effect:        0 0 20px rgba(255, 216, 107, 0.3)
└── Panel Glass:        rgba(0, 0, 0, 0.6)
```

### Typography

```
FONT FAMILIES
├── Headings:     'Orbitron', sans-serif
├── Body:         'Inter', sans-serif
├── Monospace:    'Roboto Mono', 'Share Tech Mono', monospace
└── Data Display: 'JetBrains Mono', monospace

SIZE SCALE
├── xs:   0.75rem  (12px)
├── sm:   0.875rem (14px)
├── base: 1rem     (16px)
├── lg:   1.125rem (18px)
├── xl:   1.25rem  (20px)
├── 2xl:  1.5rem   (24px)
└── 3xl:  1.875rem (30px)
```

---

## Core Components

### Module Window

The primary container for all application modules. Features draggable positioning, minimize/maximize/close controls, and persistent state.

```
┌─────────────────────────────────────────────────────────────┐
│ ┌─ Header ─────────────────────────────────────────────┐   │
│ │ [Icon] Module Name              [_] [□] [×]          │   │
│ └──────────────────────────────────────────────────────┘   │
│ ┌─ Content Area ───────────────────────────────────────┐   │
│ │                                                       │   │
│ │                                                       │   │
│ │              Module-specific content                  │   │
│ │                                                       │   │
│ │                                                       │   │
│ └──────────────────────────────────────────────────────┘   │
│ ┌─ Footer (optional) ──────────────────────────────────┐   │
│ │ Status: Active    │    [ Action Button ]             │   │
│ └──────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

**Properties:**
- `title`: Module display name
- `icon`: Lucide icon component
- `onClose`: Close handler
- `onMinimize`: Minimize handler
- `accentColor`: Theme color override
- `initialPosition`: Starting x/y coordinates
- `zIndex`: Stacking order

### Panel

A glass-morphism container for grouping related content within modules.

```css
.panel {
  background: rgba(0, 0, 0, 0.4);
  border: 1px solid rgba(255, 216, 107, 0.2);
  border-radius: 12px;
  backdrop-filter: blur(10px);
}
```

**Variants:**
- `default`: Standard glass panel
- `outlined`: Prominent border, no fill
- `solid`: Opaque background
- `highlighted`: Accent border glow

### Card

Compact content container for individual items like tokens, transactions, or settings.

```
┌─────────────────────────────────┐
│ [Icon]  Title          [Badge] │
│         Subtitle               │
│ ─────────────────────────────  │
│ Additional content or metrics  │
└─────────────────────────────────┘
```

### Status Bar

System-wide status display fixed to the bottom of the viewport.

```
┌──────────────────────────────────────────────────────────────────────────┐
│ [Logo] OBSCURA OS  │  Network: Mainnet  │  Block: 234,567,890  │  [Gear] │
└──────────────────────────────────────────────────────────────────────────┘
```

**Displays:**
- Application branding
- Network connection status
- Current block height
- Sound toggle
- Settings access
- User authentication status

### Taskbar

Module launcher and quick-access toolbar.

```
┌────────────────────────────────────────────────────────────────────┐
│ [Wallet] [Monitor] [Browser] [Vault] [Swap] [Rug] [Storage] [+]   │
└────────────────────────────────────────────────────────────────────┘
```

**Features:**
- Icon buttons for each module
- Active module indicator (glow)
- Minimized module indicators
- Keyboard shortcuts (Ctrl+1-7)

---

## Interactive Components

### Button

Primary action component with multiple variants.

**Variants:**
```tsx
// Primary (gold, filled)
<Button variant="primary">Connect Wallet</Button>

// Secondary (outlined)
<Button variant="secondary">Cancel</Button>

// Danger (red)
<Button variant="danger">Delete</Button>

// Ghost (text only)
<Button variant="ghost">Learn More</Button>
```

**States:**
- Default
- Hover (glow effect)
- Active (pressed)
- Disabled (reduced opacity)
- Loading (spinner)

### Input

Text input with cyberpunk styling.

```
┌─────────────────────────────────────┐
│ [Icon] Placeholder text...         │
└─────────────────────────────────────┘
```

**Features:**
- Optional leading icon
- Character counter
- Validation states (error/success)
- Monospace variant for addresses/hashes

### Toggle

Binary on/off switch for settings and modes.

```
[ OFF ]  ══════●  [ ON ]
```

### Dropdown / Select

Selection component for options lists.

```
┌─────────────────────────────────────┐
│ Selected Option                  ▼  │
├─────────────────────────────────────┤
│ Option 1                            │
│ Option 2                      [✓]   │
│ Option 3                            │
└─────────────────────────────────────┘
```

---

## Feedback Components

### Toast Notification

Temporary notifications for user feedback.

```
┌─────────────────────────────────────────────────┐
│ [✓] Transaction confirmed successfully      [×] │
└─────────────────────────────────────────────────┘
```

**Types:**
- Success (green accent)
- Error (red accent)
- Warning (yellow accent)
- Info (teal accent)

### Badge

Small status indicators and counts.

```tsx
<Badge variant="success">Active</Badge>
<Badge variant="warning">3 Pending</Badge>
<Badge variant="danger">High Risk</Badge>
```

### Progress

Visual progress indicators.

```
Score: 85/100
████████████████████░░░░░░░
```

**Variants:**
- Linear bar
- Circular/ring
- Segmented steps

### Skeleton

Loading placeholder for content.

```
┌─────────────────────────────────────┐
│ ████████████████████████████████    │
│ ██████████████████                  │
│ ██████████████████████████          │
└─────────────────────────────────────┘
```

---

## Data Display Components

### Data Table

Structured data presentation for tokens, transactions, etc.

```
┌──────────────────────────────────────────────────────────────┐
│ Token          │ Balance        │ Value          │ Actions  │
├──────────────────────────────────────────────────────────────┤
│ SOL            │ 12.5           │ $2,500         │ [Scan]   │
│ USDC           │ 1,000          │ $1,000         │ [Scan]   │
│ BONK           │ 1,000,000      │ $25            │ [Scan]   │
└──────────────────────────────────────────────────────────────┘
```

### Hash Display

Truncated hash with copy functionality.

```tsx
<HashDisplay 
  value="7xKXtg2CW87d97TXJSDpbD5jBkheTqA83TZRuJosgHT9" 
  truncate={true}
  copyable={true}
/>
// Displays: 7xKXtg...sHT9 [📋]
```

### Score Display

Visual score representation with color coding.

```tsx
<ScoreDisplay 
  value={85} 
  max={100}
  label="Privacy Score"
  thresholds={{ danger: 30, warning: 60, success: 80 }}
/>
```

---

## Effects & Animations

### Glow Effects

Applied to interactive elements and active states:

```css
.glow-gold {
  box-shadow: 0 0 20px rgba(255, 216, 107, 0.3);
}

.glow-teal {
  box-shadow: 0 0 20px rgba(127, 255, 212, 0.3);
}
```

### Scanline Overlay

Optional CRT effect for retro aesthetics:

```css
.scanlines::before {
  content: '';
  position: absolute;
  inset: 0;
  background: repeating-linear-gradient(
    0deg,
    rgba(0, 0, 0, 0.1) 0px,
    rgba(0, 0, 0, 0.1) 1px,
    transparent 1px,
    transparent 2px
  );
  pointer-events: none;
}
```

### Transitions

Standard animation timings:

```css
--transition-fast: 150ms ease-out;
--transition-normal: 250ms ease-out;
--transition-slow: 400ms ease-out;
```

---

## Responsive Design

### Breakpoints

```css
--mobile: 640px;
--tablet: 768px;
--desktop: 1024px;
--wide: 1280px;
```

### Mobile Adaptations

On mobile devices (< 768px):
- Modules display full-screen instead of windowed
- Taskbar converts to bottom navigation
- Touch-optimized hit targets (min 44px)
- Swipe gestures for navigation
- Simplified animations for performance

---

## Accessibility

### Keyboard Navigation

- `Tab`: Focus next interactive element
- `Shift+Tab`: Focus previous element
- `Enter/Space`: Activate buttons/toggles
- `Escape`: Close modals/dropdowns
- `Ctrl+1-7`: Quick module access

### Screen Reader Support

- Semantic HTML structure
- ARIA labels on interactive elements
- Status announcements for dynamic content
- Focus management in modals

### Color Contrast

All text meets WCAG AA contrast requirements:
- Normal text: 4.5:1 minimum
- Large text: 3:1 minimum
- Interactive elements: Clear focus indicators
