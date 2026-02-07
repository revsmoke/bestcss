# CSS Theme Studio — Agent Swarm Build Prompt

## PROJECT OVERVIEW

Build an HTML5/CSS/Vanilla JavaScript single-page application that serves as a comprehensive, interactive CSS theme builder. The user sees every styleable HTML5 element, adjusts its appearance through intuitive controls, and exports a production-ready HTML boilerplate + CSS file reflecting all their choices. The application itself should demonstrate modern CSS mastery — it is both the tool and the showcase.

## SHARED CONVENTIONS & CONSTRAINTS

### Tech Stack
- **HTML5, CSS, Vanilla JavaScript only** — no frameworks, no build tools, no dependencies
- Target: latest stable Chrome, Safari, Edge as of February 2026
- CSS features must be Baseline available or on the public release track for calendar year 2026
- Use modern CSS conventions: nesting, `@layer`, `@scope`, container queries, `color-mix()`, `oklch()`, relative color syntax, `@property`, `@starting-style`, view transitions, scroll-driven animations, anchor positioning — where appropriate and where they serve the user experience

### Naming & Architecture
- CSS methodology: Use native CSS nesting with `@layer` for organization
- Layers (in cascade order): `reset`, `tokens`, `base`, `components`, `utilities`, `user-overrides`
- JavaScript: ES modules pattern, single global `ThemeStudio` namespace
- All state lives in a single reactive store: `ThemeStudio.state`
- File structure: `index.html`, `studio.css`, `studio.js` (may be split into ES modules)

### Interface Contract: The Style Event Bus
All components communicate through a shared event system:

```javascript
// Any control that changes a style emits:
ThemeStudio.bus.emit('style:change', {
  layer: string,        // which @layer this belongs to
  selector: string,     // CSS selector
  property: string,     // CSS property name  
  value: string,        // CSS value
  specificity: string,  // 'element' | 'class' | 'id' | 'inline' | 'layer' | 'scope'
  animation?: {         // optional, for animated properties
    type: 'transition' | 'animation',
    duration: string,
    timing: string,
    delay?: string,
    iterationCount?: string
  }
});

// The live preview and export engine both subscribe to these events.
```

### State Shape
```javascript
ThemeStudio.state = {
  tokens: {},        // design tokens (colors, spacing, typography scales)
  layers: {          // organized by @layer
    reset: {},
    tokens: {},
    base: {},
    components: {},
    utilities: {},
    'user-overrides': {}
  },
  animations: [],    // @keyframes definitions
  mediaQueries: [],  // responsive breakpoints
  exportFormat: 'nested' // 'nested' | 'flat' for CSS output preference
};
```

---

## WORKSTREAMS

### Workstream 1: Foundation — Design Token System & HTML Element Catalog

**Description:**
Build the structural foundation: a complete catalog of every styleable HTML5 element organized by category, plus a design token system (CSS custom properties) for colors, typography, spacing, borders, shadows, and motion.

**Inputs:** None (this is the root dependency)

**Outputs:**
- Complete HTML document with every styleable element, organized in navigable sections
- Design token CSS custom property system with sensible defaults
- The reactive state store (`ThemeStudio.state`) and event bus (`ThemeStudio.bus`)

**Element Categories to Include:**
1. **Typography:** `h1`–`h6`, `p`, `blockquote`, `pre`, `code`, `kbd`, `samp`, `var`, `abbr`, `cite`, `dfn`, `em`, `strong`, `small`, `s`, `mark`, `sub`, `sup`, `ruby`/`rt`/`rp`, `bdi`/`bdo`, `time`, `data`
2. **Lists:** `ul`, `ol`, `li`, `dl`, `dt`, `dd`, `menu`
3. **Tables:** `table`, `thead`, `tbody`, `tfoot`, `tr`, `th`, `td`, `caption`, `colgroup`/`col`
4. **Forms:** `form`, `fieldset`, `legend`, `input` (all types), `textarea`, `select`/`option`/`optgroup`, `button`, `label`, `output`, `progress`, `meter`, `datalist`
5. **Media & Embedded:** `img`, `figure`/`figcaption`, `picture`, `video`, `audio`, `canvas`, `svg`, `iframe`, `object`, `embed`
6. **Sectioning & Structure:** `header`, `nav`, `main`, `section`, `article`, `aside`, `footer`, `div`, `span`, `hr`, `br`
7. **Interactive & Disclosure:** `details`/`summary`, `dialog`, `popover` elements
8. **Links & Navigation:** `a` (all states), `nav` patterns

**Acceptance Criteria:**
- [ ] Every styleable HTML5 element appears on the page with representative content
- [ ] Elements are organized into collapsible/navigable sections
- [ ] Design tokens are defined as CSS custom properties in the `:root`
- [ ] The state store is reactive (changes propagate to subscribers)
- [ ] The event bus supports `on`, `off`, and `emit`
- [ ] Navigating between sections is smooth (scroll-driven or tabbed)

---

### Workstream 2: Interactive Style Controls

**Description:**
Build the control panel UI that lets users adjust every CSS property for every element. Controls should be contextually appropriate: color pickers for colors, sliders for numeric values, dropdowns for enumerated values, text inputs for custom values. Critically, controls must respect and teach the CSS cascade — users should be able to set styles at different specificity levels.

**Inputs:** Foundation (element catalog, state store, event bus, token system)

**Outputs:**
- A control panel that appears alongside each element or in a persistent sidebar
- Controls emit `style:change` events per the interface contract
- Cascade/specificity selector on each control group

**Control Types Needed:**
- **Color:** Native picker + OKLCH/HSL sliders + `color-mix()` compositor + contrast checker
- **Typography:** Font family (with preview), size (with `clamp()` builder), weight, line-height, letter-spacing, `text-wrap: balance|pretty`
- **Spacing:** Visual box model editor (margin, padding, gap) with unit toggle (px/rem/em/%)
- **Layout:** Display mode selector, flexbox/grid configurators with visual feedback
- **Borders & Outlines:** Width, style, color, radius (per-corner)
- **Shadows:** `box-shadow` and `text-shadow` builder with layering (multiple shadows)
- **Filters & Effects:** `backdrop-filter`, `filter`, `mix-blend-mode`, `opacity`
- **Scroll & Overflow:** `overflow`, `scroll-behavior`, `overscroll-behavior`, `scrollbar-color`/`scrollbar-width`

**Cascade Controls (critical differentiator):**
- Each control group has a specificity selector: element, class, ID, layer, scope
- Show computed value with cascade resolution visualization
- Allow `!important` toggle with educational warning
- Show which layer a rule lives in and allow moving between layers
- Demonstrate `@scope` boundaries visually

**Acceptance Criteria:**
- [ ] Every major CSS property category has appropriate interactive controls
- [ ] Controls emit events per the interface contract
- [ ] Changing a control instantly updates the live preview
- [ ] Users can set the same property at different specificity levels and see cascade resolution
- [ ] Color controls support modern color spaces (oklch, lab, lch)
- [ ] Spacing controls show a visual box model diagram
- [ ] Typography controls include a `clamp()` responsive size builder

---

### Workstream 3: Animation & Transition Studio

**Description:**
Build the animation and transition authoring experience. Users should be able to apply CSS transitions to any property, build `@keyframes` animations with a visual timeline, configure scroll-driven animations, and use `@starting-style` for entry animations. Include a curated library of preset animations they can customize.

**Inputs:** Foundation (element catalog, state store, event bus)

**Outputs:**
- Transition controls for any animatable property
- A keyframe animation builder (visual timeline)
- Scroll-driven animation configurator
- View transition configurator
- Animation preset library
- All animation data stored in `ThemeStudio.state.animations`

**Features:**
- **Transitions:** Property, duration, timing function (with cubic-bezier visual editor), delay
- **Keyframe Animations:** Visual timeline editor, add/remove keyframes, per-keyframe property editing
- **Scroll-Driven Animations:** `animation-timeline: scroll()` and `view()` configurator
- **Entry/Exit Animations:** `@starting-style` for appear animations, `transition-behavior: allow-discrete` for display/overlay transitions
- **Timing Functions:** Visual cubic-bezier curve editor, `linear()` function with easing graph, spring physics previewer
- **Presets:** Fade, slide, scale, rotate, bounce, shake, flip — all customizable
- **Preview:** Play/pause, slow-motion, step-through controls

**Acceptance Criteria:**
- [ ] Users can add transitions to any animatable property
- [ ] Keyframe animation builder produces valid `@keyframes` CSS
- [ ] Cubic-bezier editor is visual and interactive (drag control points)
- [ ] Scroll-driven animations work in the preview
- [ ] `@starting-style` entry animations demonstrate correctly
- [ ] At least 8 animation presets are included and customizable
- [ ] Animation preview has playback controls

---

### Workstream 4: CSS Functions & Modern Selectors Showcase

**Description:**
Build an interactive showcase/playground for CSS functions and modern selectors. This is both educational and practical — users learn what's available and can incorporate these into their theme. Each function/selector should have a live, interactive demo.

**Inputs:** Foundation (element catalog, state store, event bus)

**Outputs:**
- Interactive demos for each CSS function
- Selector playground with match highlighting
- Integration with the control panel (users can use functions in their theme values)

**CSS Functions to Demonstrate:**
- **Color:** `color-mix()`, `oklch()`, `light-dark()`, relative color syntax (`from`)
- **Math:** `calc()`, `min()`, `max()`, `clamp()`, `round()`, `mod()`, `rem()`, `abs()`, `sign()`
- **Shape & Layout:** `minmax()`, `fit-content()`, `repeat()` with `auto-fill`/`auto-fit`
- **Transform:** `translate()`, `rotate()`, `scale()`, `skew()`, `perspective()`, `matrix()`
- **Filter:** `blur()`, `brightness()`, `contrast()`, `grayscale()`, `hue-rotate()`, `saturate()`
- **Gradient:** `linear-gradient()`, `radial-gradient()`, `conic-gradient()`, `repeating-*` variants
- **Other:** `env()`, `attr()`, `url()`, `var()` with fallbacks, `counter()`, `image-set()`

**Modern Selectors to Demonstrate:**
- `:has()`, `:is()`, `:where()`, `:not()` with complex arguments
- `:nth-child(of S)` syntax
- `::highlight()`, `::backdrop`, `::marker`, `::selection`, `::placeholder`, `::file-selector-button`
- `:user-valid`, `:user-invalid` (form validation)
- `:popover-open`, `:modal`
- `@scope` boundaries, `&` nesting selector
- `:focus-visible` vs `:focus`, `:focus-within`
- `@container` queries with named containers

**Acceptance Criteria:**
- [ ] Every listed CSS function has a live interactive demo
- [ ] Every listed selector has a demo showing what it matches (highlight matched elements)
- [ ] Users can copy function syntax into their theme values
- [ ] Selector demos show specificity weight
- [ ] Demos clearly label browser support status

---

### Workstream 5: Export Engine

**Description:**
Build the export system that generates a clean, production-ready HTML boilerplate and CSS file from the user's configuration. The exported CSS should be well-organized, use modern conventions (nesting, layers, custom properties), and include helpful comments.

**Inputs:** State store (all style data), token system, animation data

**Outputs:**
- Generated HTML5 boilerplate file with semantic markup
- Generated CSS file with all user styles organized by `@layer`
- Download functionality (individual files or zip)
- Import functionality (load a previously exported theme)
- Live CSS preview panel showing the generated code

**Export Features:**
- **CSS Output Options:**
  - Nested (modern) or flat (legacy compatibility) syntax
  - With or without `@layer` organization
  - Minified or formatted
  - With or without comments
- **HTML Boilerplate:**
  - Clean semantic HTML5 template
  - Proper `<meta>` tags, Open Graph basics
  - CSS file linked
  - Includes only the elements the user has styled (optional: include all)
- **Live Preview:**
  - Split-pane code editor showing generated CSS in real-time
  - Syntax highlighting for the CSS output
  - Copy-to-clipboard for individual rules or the full file
- **Import/Load:**
  - Parse a CSS file and load its values back into the controls
  - Save/load theme presets (localStorage)

**Acceptance Criteria:**
- [ ] Export produces valid CSS that renders identically to the live preview
- [ ] CSS uses `@layer` organization by default
- [ ] CSS uses native nesting by default
- [ ] HTML boilerplate is valid HTML5 (passes W3C validator)
- [ ] Download works (both individual files and as a zip)
- [ ] Importing a previously exported theme restores the UI state
- [ ] Theme presets can be saved and loaded from localStorage
- [ ] Generated code has syntax highlighting in the preview

---

### Workstream 6: Visual Polish & Experience

**Description:**
This is the "make it world-class" workstream. Own the overall UI/UX: the application shell, navigation, responsive layout, onboarding, keyboard shortcuts, accessibility, and visual delight. This agent should think like a designer, not just a developer.

**Inputs:** All other workstreams (this is the integration and polish layer)

**Outputs:**
- Application shell with navigation, theming, and responsive layout
- Onboarding flow or guided tour
- Keyboard shortcuts and command palette
- Accessibility audit and fixes
- Visual flourishes: smooth transitions, micro-interactions, satisfying feedback
- Dark/light mode for the studio itself (independent of the theme being built)

**Ideas to Explore:**
- Command palette (Cmd+K) for quick access to any control or element
- Breadcrumb showing current cascade context (e.g., `@layer base > section > h1`)
- Side-by-side or overlay comparison of before/after
- Undo/redo with visual timeline
- "Randomize" button that generates a harmonious random theme
- Accessibility score/checker for the theme being built (contrast ratios, etc.)
- Responsive preview at different breakpoints
- Print stylesheet preview
- Share themes via URL (state encoded in hash/query)

**Acceptance Criteria:**
- [ ] Application is responsive and works on tablet-sized screens and above
- [ ] Navigation between sections is clear and fast
- [ ] The studio itself has dark and light mode
- [ ] Keyboard shortcuts exist for common actions
- [ ] Focus management and ARIA roles are correct
- [ ] Undo/redo works for at least the last 50 actions
- [ ] The overall experience feels polished, cohesive, and delightful

---

## DEPENDENCY GRAPH

```
                    ┌───────────────────────┐
                    │   Workstream 1:       │
                    │   Foundation          │
                    │   (tokens, elements,  │
                    │    state, event bus)  │
                    └──────────┬────────────┘
                               │
              ┌────────────────┼────────────────┐
              │                │                │
              ▼                ▼                ▼
   ┌──────────────────┐ ┌────────────┐ ┌──────────────────┐
   │  Workstream 2:   │ │ WS 3:      │ │  Workstream 4:   │
   │  Style Controls  │ │ Animation  │ │  Functions &     │
   │                  │ │ Studio     │ │  Selectors       │
   └────────┬─────────┘ └─────┬──────┘ └────────┬─────────┘
            │                 │                  │
            └────────────────┬┘──────────────────┘
                             │
                             ▼
                  ┌──────────────────────┐
                  │  Workstream 5:       │
                  │  Export Engine       │
                  │  (needs all style    │
                  │   data to export)    │
                  └──────────┬───────────┘
                             │
                             ▼
                  ┌──────────────────────┐
                  │  Workstream 6:       │
                  │  Visual Polish &     │
                  │  Experience          │
                  │  (integration layer) │
                  └──────────────────────┘
```

**Parallelization opportunities:**
- Workstreams 2, 3, and 4 can all run in parallel once Workstream 1 is complete
- Workstream 5 can begin scaffolding its architecture in parallel with 2–4, but needs their output for full implementation
- Workstream 6 can begin the app shell and navigation in parallel, but final polish requires all others

---

## INTEGRATION PLAN

### Assembly Order
1. Workstream 1 delivers the foundation — all other agents receive this as their starting context
2. Workstreams 2, 3, 4 develop against the foundation independently
3. Integration checkpoint: merge all components, verify event bus communication works end-to-end
4. Workstream 5 wires up the export engine to the live state
5. Workstream 6 does final assembly, polish, and cross-cutting concerns

### Integration Testing Strategy
After merging all workstreams:
- Change a style via controls → verify live preview updates → verify export output matches
- Build an animation → verify it plays in preview → verify it exports correctly
- Set styles at different cascade levels → verify resolution is correct in preview and export
- Import a previously exported theme → verify all controls restore to correct state
- Test all interactive demos in the CSS functions showcase
- Keyboard-navigate the entire application
- Test at viewport widths: 768px, 1024px, 1440px, 1920px

---

## GLOBAL ACCEPTANCE CRITERIA

These apply to the fully assembled, integrated application:

- [ ] **Works end-to-end:** User can adjust styles → see live preview → export valid CSS + HTML
- [ ] **Cascade correctness:** Setting the same property at different specificity levels resolves correctly per CSS spec
- [ ] **Animation fidelity:** All animations preview correctly and export valid CSS
- [ ] **Export round-trip:** Export → import → all controls restore → re-export produces identical CSS
- [ ] **Cross-browser:** Works in latest Chrome, Safari, and Edge
- [ ] **No build tools:** Opens directly in a browser from the file system (`file://` protocol) or a simple static server
- [ ] **Performance:** Smooth 60fps interactions, no jank on style changes
- [ ] **Modern CSS:** Uses nesting, `@layer`, `@scope`, `@property`, container queries, and other modern features appropriately throughout
- [ ] **Self-documenting:** The generated CSS includes comments explaining the layer structure and token system
- [ ] **Accessible:** Meets WCAG 2.1 AA for the studio interface itself
- [ ] **Delightful:** A designer or developer would enjoy using this tool and learn something from it
