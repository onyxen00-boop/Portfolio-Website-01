# coldpixels.co — MASTER TASK LIST
## Every Task · Every File · Every Fix · In Exact Order
## Status: [✅ Done] [🔄 In Progress] [⬜ Todo]

---

## HOW TO USE THIS LIST

1. Give this file + `COLDPIXELS_COMPLETE_FIX_PROMPT.md` to your AI (Cursor, Claude, etc.)
2. Say: "Complete Task 1. When done, tell me. I will then say 'next' for Task 2."
3. Verify each task works before moving on
4. Each task is self-contained — one AI context window

---

## PHASE 0 — FOUNDATION (Do These First)
```
These 4 tasks must be done before anything else.
Without them, nothing works.
```

### TASK 0.1 — Install Dependencies
```
npm install framer-motion gsap @gsap/react react-hot-toast swr firebase react-hook-form zod react-markdown
npm install -D @types/node

Verify: npm run build shows no missing module errors
```

### TASK 0.2 — Environment Variables
```
Create file: .env.local in project root
Content:
  NEXT_PUBLIC_FIREBASE_API_KEY=
  NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN=
  NEXT_PUBLIC_FIREBASE_PROJECT_ID=
  NEXT_PUBLIC_FIREBASE_STORAGE_BUCKET=
  NEXT_PUBLIC_FIREBASE_MESSAGING_SENDER_ID=
  NEXT_PUBLIC_FIREBASE_APP_ID=

Also create: .env.local.example (same content, for documentation)

Fill in values from Firebase Console → Project Settings → Web App
```

### TASK 0.3 — Replace globals.css
```
Replace entire content of: app/globals.css
With: the globals.css from outputs/app/globals.css

CRITICAL FIXES in this file:
  - cursor: none on html + html * (fixes default cursor showing over custom cursor)
  - All keyframe animations defined
  - All utility classes (.grok-card, .border-sweep, .shimmer-text, etc.)
  - CSS custom properties (--font-display, --bg-primary, etc.)
  - [data-reveal] / [data-reveal-left] animation system
  - noise-overlay, cross-grid, dot-grid utility classes

Verify: Run npm run dev, open browser, default arrow cursor should be GONE
```

### TASK 0.4 — Replace lib/firebase.ts and lib/firestore.ts
```
Replace: src/lib/firebase.ts
  - Remove getAuth (not needed)
  - Remove databaseURL (not needed)
  - Prevent duplicate initialization with getApps() check

Replace: src/lib/firestore.ts
  - Fix collection name: 'config' → 'portfolio_config'
  - Fix document ID: 'default' → 'main'
  - Add getSocialLinks() function (was missing entirely)
  - Add getStats() function (was missing entirely)
  - Add listenToStats() real-time listener
  - Add listenToConfig() real-time listener
  - Add submitContactForm() that actually saves to Firestore
  - Add voteFAQ(), incrementProjectViews()
  - All functions use correct field: visible == true

Verify: No TypeScript errors. Firebase connects without console errors.
```

---

## PHASE 1 — CURSOR SYSTEM
```
Fix the broken cursor before everything else so you can
actually interact with the site while building it.
```

### TASK 1.1 — SmoothCursor Component
```
Create: src/components/cursor/SmoothCursor.tsx
Content: Use outputs/src/components/cursor/SmoothCursor.tsx

Features:
  - Spring physics (framer-motion useSpring, damping:20 stiffness:400)
  - Two layers: fast dot (exact position) + slow ring (springs behind)
  - Sprinkle particles: 5 shapes (star, circle, cross, diamond, ring)
  - Particles emit on mouse move (throttled: max 25/sec)
  - Burst of 12 particles on click
  - Cursor state detection: default | link | button | text | image | drag
  - Ring changes size per state (32px default, 56px button, 80px image)
  - Label appears inside ring for image ("VIEW") and drag ("DRAG")
  - Auto-disabled on touch devices (pointer: coarse)
  - Canvas RAF loop for particles
  - All cleanup on unmount

IMPORTANT: Delete old SparkCursor.tsx or rename it. Use SmoothCursor everywhere.
```

### TASK 1.2 — Add Cursor to Layout
```
Edit: app/layout.tsx
  - Import SmoothCursor from '@/components/cursor/SmoothCursor'
  - Mount it OUTSIDE SmoothScrollProvider, at the very end of <body>
  - This ensures cursor is always on top and never clipped by scroll containers

Also verify globals.css has:
  @media (pointer: fine) {
    html, html *, html *::before, html *::after { cursor: none !important; }
  }

Verify: Open browser, move mouse → custom cursor shows, native cursor GONE.
        Hover a link → ring shrinks. Hover a button → ring grows. Click → particles burst.
```

---

## PHASE 2 — LAYOUT SHELL (Navbar, Footer, Transitions)

### TASK 2.1 — Root layout.tsx
```
Replace: app/layout.tsx
Use: outputs/app/layout.tsx

This wraps EVERY page with:
  ColdPixelsIntro → PageTransition → AnnouncementBar →
  ScrollProgress → Navbar → children → Footer → BackToTop →
  Toaster → SmoothCursor

Fonts loaded: Bebas Neue, DM Mono, Playfair Display, Instrument Serif
All as CSS variables: --font-display, --font-mono, --font-serif, --font-body
```

### TASK 2.2 — Navbar (Firebase-connected)
```
Rewrite: src/components/layout/Navbar.tsx

Firebase fields used (all real-time via useConfig):
  config.owner_name            → shown nowhere in nav but drives logo
  config.availability_label    → shown next to availability dot
  config.availability_status   → drives pulsing dot (white dot = active)
  config.owner_email           → shown in mobile menu

From social_links collection (via useSocialLinks):
  socialLinks[].platform, socialLinks[].url  → mobile menu social links (REAL URLs)

Features:
  - Transparent → frosted glass + border on scroll (>60px, GSAP quickTo)
  - Logo left, nav links center (desktop), hamburger right (mobile)
  - Active link: framer-motion layoutId underline slides to active page
  - Availability dot with ping animation next to "GET IN TOUCH" button
  - Mobile menu: full screen overlay, links animate in with stagger
  - Mobile menu shows: nav links (large font-display) + email + social links
  - ESC key closes mobile menu
  - usePathname() auto-closes menu on route change

Verify: Links navigate correctly. Mobile menu opens/closes. Scroll → glass effect.
```

### TASK 2.3 — Footer (Firebase-connected)
```
Rewrite: src/components/layout/Footer.tsx

Firebase fields used:
  config.footer_tagline         → giant CTA "Let's build something remarkable."
  config.owner_bio_short        → brand column bio text
  config.owner_email            → mailto: link (REAL email from Firebase)
  config.owner_location         → contact column
  config.availability_label     → availability dot label
  config.footer_copy            → bottom bar copyright text
  config.copyright_year         → bottom bar year
  config.calendly_url           → "Book a call ↗" button (shown only if set)
  socialLinks[].platform + .url → Social column, real clickable URLs

Layout (4 columns on lg, 2 on sm, 1 on mobile):
  Col 1: Brand (logo, bio, availability)
  Col 2: Navigate (internal links)
  Col 3: Social (from Firebase, real URLs opening _blank)
  Col 4: Contact (email, location, calendly)

Verify: Change owner_email in Firebase → footer email updates within 60 seconds.
        Click a social link → correct URL opens in new tab.
```

### TASK 2.4 — PageTransition (Pixel Shatter)
```
Rewrite: src/components/layout/PageTransition.tsx

Current version is boring opacity fade — replace with:
  - 8×5 grid of tiles (40 tiles total)
  - On route change: tiles slam in from top (translateY -100% → 0%), stagger random
  - Brief hold (100ms)
  - Then tiles scatter out to bottom (translateY 100%), stagger random
  - Behind tiles during hold: bg-[#080808] fully covers page
  - Entry: tiles are NOT shown (already departed)
  - Uses usePathname() to detect route change
  - GSAP timeline with stagger: { amount: 0.3, from: 'random' }
  - Pointer-events: all during transition, none after

Verify: Navigate between pages → see tile shatter animation.
```

### TASK 2.5 — ScrollProgress, BackToTop, AnnouncementBar
```
ScrollProgress:
  - Fixed 2px line at top (z-index 9990)
  - White background, scaleX transforms with scroll
  - useScroll() hook → scrollYProgress → transform-origin left

BackToTop:
  - Appears when scrollY > 400px
  - Fixed bottom-right, framer-motion AnimatePresence fade
  - Click → window.scrollTo({ top: 0, behavior: 'smooth' })
  - Border sweep animation on hover

AnnouncementBar:
  - Reads config.announcement_bar_enabled from Firebase
  - If false: renders null
  - If true: shows config.announcement_bar_text + link to config.announcement_bar_link
  - Can be dismissed (sessionStorage flag)
  - Sits above navbar (not sticky, part of page flow)

Verify: Scroll down → progress bar fills. ScrollY>400 → back-to-top appears.
        Toggle announcement_bar_enabled in Firebase → bar appears/disappears.
```

### TASK 2.6 — ColdPixelsIntro (Loading Screen)
```
Keep existing ColdPixelsIntro.tsx but verify:
  - sessionStorage.getItem('cp_intro_done') → if set, skip to 150ms fade
  - Content is visibility:hidden during intro, then revealed
  - Sets sessionStorage.setItem('cp_intro_done', '1') on complete
  - prefers-reduced-motion: skip entire sequence, just onComplete()
  - Cleanup: all GSAP timelines killed, all RAF cancelled

Also verify: Page content loads BEHIND the intro (Firebase listeners start immediately)
```

---

## PHASE 3 — ALL FIREBASE HOOKS (Real-time)

### TASK 3.1 — useConfig (Real-time)
```
Rewrite: src/hooks/useConfig.ts

Pattern:
  - useState(MOCK_CONFIG) for immediate data
  - useEffect → listenToConfig(onSnapshot) → setConfig({ ...MOCK_CONFIG, ...data })
  - 3s timeout fallback (shows mock if Firebase is slow)
  - Returns { config, loading }
  - MOCK_CONFIG has all required fields with sensible defaults

Critical: config.owner_email from this hook powers:
  - Navbar "GET IN TOUCH" link
  - Footer email
  - Contact page "Email" field
  - Autofill Compose TO: field
  - Mobile menu email

Verify: Change owner_email in Firebase Console → all 5 locations update within 3s.
```

### TASK 3.2 — useSocialLinks (Real-time)
```
Rewrite: src/hooks/useSocialLinks.ts

Pattern:
  - Reads from social_links COLLECTION (NOT from config document)
  - getSocialLinks() queries: visible==true, orderBy order asc
  - Falls back to MOCK social links if empty
  - Returns { socialLinks, loading }

Each social link document must have: platform, url, icon, label, order, visible
url must be FULL URL (https://github.com/xxx not just "github")

Appears in: Navbar mobile menu, Footer Social column, About page, Contact page
All appearances: <a href={link.url} target="_blank" rel="noopener noreferrer">

Verify: Add a social link in Firebase with url="https://github.com/xxx"
        → Footer Social column shows it as clickable link opening correct URL.
```

### TASK 3.3 — useStats (Real-time)
```
Rewrite: src/hooks/useStats.ts

Pattern:
  - Real-time: useEffect → listenToStats(onSnapshot) → setStats(data)
  - Falls back to MOCK_STATS if collection empty
  - Each stat: { value(number), suffix(string), label(string), icon, order, visible }

Appears in: Home StatsCounter section, About page stats grid

Verify: Change stat value in Firebase → counter updates on page within 2s.
```

### TASK 3.4 — useProjects (SWR + slug lookup)
```
Rewrite: src/hooks/useProjects.ts

Two exports:
  useProjects(opts?) → list with optional { featured, category } filters
  useProject(slug)   → single project for detail page

Both fall back to MOCK_PROJECTS if Firestore empty.
Mock data has realistic content including:
  cover_image_url: 'https://images.unsplash.com/...' (real working images)
  live_url, github_url → real placeholder URLs
  tags, tech_stack → arrays

Verify: /work page shows projects. /work/[slug] loads detail.
```

### TASK 3.5 — Remaining Hooks
```
Verify/fix these hooks all follow the same pattern:

useSkills.ts     → getSkills() → visible==true, orderBy order
useExperience.ts → getExperience() → visible==true, orderBy order
useTestimonials.ts → getTestimonials() → visible==true && approved==true
useFAQs.ts       → getFAQs() → visible==true, orderBy order
useServices.ts   → getServices() → visible==true, orderBy order
useAwards.ts     → getAwards() → visible==true, orderBy order

All return mock data if Firestore returns empty.
All use useSWR with { revalidateOnFocus: false, dedupingInterval: 60000 }.
```

---

## PHASE 4 — ANIMATION SYSTEM

### TASK 4.1 — Reveal on Scroll (Global)
```
Create: src/components/animations/RevealOnScroll.tsx

IntersectionObserver wrapper:
  - Watches [data-reveal] and [data-reveal-left] elements
  - On intersect: adds 'revealed' class
  - CSS handles the transition (defined in globals.css)
  - staggerDelay prop: multiplied by element index within the observer
  - once: true (don't re-animate on scroll back up)
  - threshold: 0.1

Mount in layout.tsx as a client component that runs globally.

Also add useReveal() hook:
  - Attaches observer to all [data-reveal] elements on mount
  - Cleans up on unmount
  - Works across page transitions (re-runs on pathname change)
```

### TASK 4.2 — TextAnimations (6 modes)
```
Verify/enhance: src/components/animations/TextAnimations.tsx

All 6 modes must work:
  scramble:   chars scramble with random chars → settle left→right
  wave:       chars fly up from below with spring bounce
  blur:       words blur clear from left
  split:      chars split vertically (top/bottom halves)
  liquid:     chars slide in from left with elastic spring
  typewriter: chars revealed one by one with blinking cursor

All modes:
  - Check prefers-reduced-motion → skip to final state instantly
  - GSAP ref-based (no useState for animation values)
  - Cleanup: kill GSAP tweens and RAF on unmount
  - trigger: 'onLoad' | 'onScroll'
  - IntersectionObserver for onScroll (observe, animate, unobserve)
```

### TASK 4.3 — ScrollVelocityMarquee (RAF-based)
```
Verify/enhance: src/components/animations/ScrollVelocityMarquee.tsx

CRITICAL: Zero useState during animation. All via refs + direct DOM.

3 rows from Firebase: config.marquee_items (array) split into rows
  Row 1: fill text, dir right, speed 2
  Row 2: outline text, dir left, speed 1.5
  Row 3: ghost text, dir right, speed 2.5

On scroll: velocity increases proportional to scroll speed, decays 0.88/frame
Skew: Math.min(Math.abs(velocity) * 0.003, 6) degrees applied via skewX
Separator: <span style={{color:'#FF6B2C'}}>◆</span> between items
Font: Bebas Neue, clamp(64px, 9vw, 120px)

Appears on: Home page after hero
```

### TASK 4.4 — MagneticWrapper
```
Verify: src/components/animations/MagneticWrapper.tsx

Wraps any child. On hover: element magnetically follows cursor.
Strength: 0.3 (30% of distance)
Spring back on mouse leave (transition 0.4s cubic-bezier(0.34,1.56,0.64,1))
Distance threshold: 80px
Uses requestAnimationFrame for smooth tracking
Apply to: "GET IN TOUCH" button, primary CTAs, logo

Verify: Hover near (not on) the button → button pulls toward cursor.
```

### TASK 4.5 — CountUp Animation
```
Verify: src/components/animations/CountUp.tsx

Props: end(number), duration(number, default 2), suffix(string)
On scroll enter: GSAP animate from 0 → end over duration seconds
ease: 'power2.out'
Formatted output: Math.floor(current) + suffix
prefers-reduced-motion: just show final value immediately

Used in: StatsCounter section, About stats grid
```

### TASK 4.6 — GSAP ScrollTrigger (Global Setup)
```
Create: src/hooks/useScrollReveal.ts

Hook that:
  - Imports { ScrollTrigger } from 'gsap/ScrollTrigger'
  - gsap.registerPlugin(ScrollTrigger) on mount
  - Takes a ref and animates children on scroll enter
  - Returns context (gsap.context) for cleanup

Usage example (add to each section):
  const sectionRef = useRef<HTMLElement>(null);
  useScrollReveal(sectionRef);

Also create: src/components/animations/ParallaxLayer.tsx
  - Wraps children, applies GSAP ScrollTrigger scrub parallax
  - speed prop: 0.1 to 0.9 (how fast relative to scroll)
  - Used for: hero background, decorative text, floating elements
```

### TASK 4.7 — Ripple Component
```
Verify: src/components/animations/Ripple.tsx

Pure CSS — no JavaScript needed for the animation.
  - numCircles: 5 (default)
  - Each: position absolute, centered, border only (no fill), border-radius 50%
  - Size: mainCircleSize + (i * mainCircleSize * 0.4)
  - Animation: rippleExpand keyframe, scale 0.3→3, opacity 0.8→0
  - Delay: i * (duration / numCircles) seconds
  - prefers-reduced-motion: show static rings, opacity 0.2

Used behind: Hero CTA button, Contact hero heading, CTABanner
```

### TASK 4.8 — AnimatedBeam
```
Verify: src/components/animations/AnimatedBeam.tsx

SVG paths between DOM nodes using:
  - ResizeObserver to track container + node positions
  - Quadratic bezier curves (controlled by curvature prop)
  - Two SVG layers: ghost track (opacity 0.12) + animated traveler
  - Traveler: strokeDasharray="60 10000", animated dashoffset
  - All colors: white/rgba(255,255,255,*) — no color variants
  - Cleanup: ResizeObserver.disconnect() on unmount

Used on: Home page BeamInteractions section (data from Firebase beam_nodes + beam_connections)
```

---

## PHASE 5 — ALL PAGES (Firebase-Connected, Long & Impressive)

### TASK 5.1 — Home Page (app/page.tsx)
```
The home page must have ALL these sections in order:

1. HeroSection (100svh)
   Firebase: config.hero_headline_word1/2/3, config.hero_subheadline
   Firebase: config.hero_cta_primary_label/url, socialLinks (vertical bar)
   Firebase: stats[] (3 stats with CountUp)
   Animations: TextAnimations (scramble on word1, wave on word2, liquid on word3)
   Background: ParticleField (Three.js, dynamic import ssr:false)
               Fallback on mobile: animated gradient orbs (pure CSS)
   MagneticWrapper on CTA buttons

2. KineticMarquee
   Firebase: config.marquee_items array
   ScrollVelocityMarquee (3 rows, velocity on scroll)

3. FeaturedProjects
   Firebase: useProjects({ featured: true })
   Each card: cover_image_url, title, category, tags, short_description
   Click: navigate to /work/[slug]
   Hover: image scale + border sweep + corner brackets light up
   Grid: asymmetric 2-col (7/12 + 5/12 alternating) on lg

4. StatsCounter (white section — inverted colors)
   Firebase: useStats() real-time
   Background: #F5F5F5, text: #080808
   CountUp animation on scroll enter
   Navbar inverts when this section is in viewport

5. TestimonialsSection (if config.show_testimonials)
   Firebase: useTestimonials() (approved==true)
   Framer Motion drag carousel
   Each card: avatar, quote, name, role, company, star rating

6. CTABanner
   Firebase: config.footer_tagline (or dedicated cta_text field)
   Huge text, MagneticWrapper on button, Ripple behind button
   "LET'S BUILD SOMETHING →" linking to /contact

Verify: Every section shows Firebase data. Changing data in Firebase → page updates.
```

### TASK 5.2 — About Page (app/about/page.tsx)
```
6 sections, all Firebase-connected:

1. Hero (config.owner_name, owner_title, owner_bio_short, availability_*, location, timezone)
   Right side: stats grid from useStats() (value, suffix, label)

2. Biography (config.owner_bio_long — split by \n\n into paragraphs)

3. Skills (useSkills() — category: Design | Development | Tools)
   3 columns, each with animated fill bars (framer-motion whileInView)
   Bar width: skill.proficiency%

4. Experience Timeline (useExperience())
   company, role, type, start_date, end_date, description, achievements[]
   GSAP ScrollTrigger stagger reveal per entry

5. Awards Grid (useAwards(), only if config.show_awards)
   title, organization, year, url (REAL clickable link)

6. Connect / Social Links (useSocialLinks())
   4-col grid, each social link is <a href={link.url} target="_blank">
   url shows below platform name so user sees it's connected

All sections: data-reveal attributes for GSAP scroll reveal.
Verify: Every field shows from Firebase. Social links open correct URLs.
```

### TASK 5.3 — Work Page (app/work/page.tsx)
```
Firebase-connected project grid with filtering.

Hero:
  Decorative large "WORK" text (white 2% opacity)
  "N Projects Built With Precision" (N = projects.length)
  TextAnimations scramble on hero text

Filter Bar (sticky below navbar):
  Pills: All + unique categories from project data
  View toggle: Grid | List
  Active: bg-white text-black
  Animation: GSAP pill indicator slides to active

Project Grid (useProjects()):
  cover_image_url → <Image> with grayscale filter (hover removes)
  title, category, year, short_description, tags[]
  featured projects: larger card (md:col-span-2)
  AnimatePresence on filter change (layout animations)

Each card on hover:
  translateY(-6px), border brightens
  CornerBorder corners lighten
  .shine-sweep sweeps across
  Thumbnail preview (image.cover_image_url)

List view:
  72px rows with columns: title | category | year | →
  Floating thumbnail near cursor on hover

Empty state: "Add projects in Firebase → projects collection"

Verify: Filter by category works. Cards link to /work/[slug].
```

### TASK 5.4 — Work Detail Page (app/work/[slug]/page.tsx)
```
Reads from Firebase by slug. Increments view_count on load.

Full-bleed hero image (cover_image_url, 60vh height)
Gradient overlay (transparent → #080808)

Two-column layout (lg):
  Left (65%):
    category label, title (Bebas Neue clamp(40,5vw,96px))
    subtitle (Playfair Display italic)
    tags[] chips
    Challenge section
    Approach section
    Outcome section
    Gallery (project.gallery[] images)
  
  Right sidebar (35%, sticky):
    Client, Year, Role, Duration
    live_url → "VISIT LIVE SITE ↗" button (REAL link, opens _blank)
    github_url → "VIEW CODE ↗" button (REAL link, opens _blank)
    tech_stack[] chips

Back button: "← BACK TO WORK" (top-left)
Next/Prev project navigation (bottom)

Verify: project.live_url shows as real clickable button.
        project.github_url shows as real clickable button.
        Both open in new tab.
```

### TASK 5.5 — Contact Page (app/contact/page.tsx)
```
The most important page — contact form ACTUALLY saves to Firestore.

HERO:
  "LET'S BUILD SOMETHING REMARKABLE"
  Ripple animation behind heading
  Shows config.response_time, config.owner_email

TAB TOGGLE: "Contact Form" | "Autofill Compose"

CONTACT FORM TAB:
  Fields: name, email, project_type (chips from Firebase), budget (chips from Firebase), timeline (select), message
  On submit: await submitContactForm(data) → saves to contact_submissions collection
  Success: shows "✓ Message Received" state
  Error: toast notification
  Validation: react-hook-form + zod schema

AUTOFILL COMPOSE TAB:
  TO: shows config.owner_email live from Firebase (changes when you change in Firebase)
  Fields: user's reply email, project type chips, project brief textarea
  Method toggle: Gmail | Mail App
  Launch button: builds URL and opens Gmail or mailto
  Live preview panel: shows To, From, Subject, body preview

RIGHT SIDEBAR (info panel):
  Email (config.owner_email) → mailto: link
  Phone (config.owner_phone) → tel: link
  Location (config.owner_location)
  Response time (config.response_time)
  Social links (useSocialLinks()) → all real URLs
  Calendly button (config.calendly_url, shown only if set)

Verify: Submit form → document appears in Firestore contact_submissions.
        Change config.owner_email in Firebase → Autofill TO: updates.
        Social links in sidebar are real URLs.
```

### TASK 5.6 — Services Page (app/services/page.tsx)
```
Firebase-connected accordion of services.

Hero: "SERVICES" Bebas Neue with TextAnimations

Services list (useServices()):
  Each service row:
    Index number (01, 02, 03...) + name (large font-display)
    tagline (DM Mono, smaller)
    "POPULAR" badge if service.popular
    starting_price (if set)
    Click → expands accordion

Expanded content (AnimatePresence height animation):
  description (Instrument Serif, large)
  features[] column: ✓ What's Included
  deliverables[] column: → Deliverables
  timeline + starting_price column + "GET A QUOTE →" button

Process section (below services):
  useProcessSteps() or from Firebase process_steps collection
  5 numbered steps, GSAP ScrollTrigger pin
  "01 — Discovery" style numbering

CTA at bottom linking to /contact

Verify: All services expand correctly. Features list shows from Firebase.
```

### TASK 5.7 — FAQ Page (app/faq/page.tsx)
```
Firebase-connected searchable FAQ.

Hero with decorative large "FAQ" text

Sticky search bar (below navbar):
  Filters in real-time as user types
  Placeholder cycles through FAQ questions (typewriter)
  Clears on button click

Category filter pills (unique categories from FAQ data)

FAQ Accordion (useFAQs()):
  question (DM Mono, white/80)
  Expanding answer (AnimatePresence, height animation)
  category badge
  Helpful voting: "Yes (N) / No (N)"
    Click: voteFAQ(id, helpful) → increments Firestore counter
    Session flag: sessionStorage prevents double-voting
    Counter shows live counts from Firebase

Empty + no-match states with Firebase setup hint

CTA: "Still have questions? Ask me directly →" (links to /contact)

Verify: Vote on FAQ → Firestore helpful_count increments.
        Search → filters in real-time.
        All FAQ data from Firebase.
```

### TASK 5.8 — Features Page (app/features/page.tsx)
```
Showcase page for what makes the studio unique.

Background: #050505 + dot-grid + subtle scanlines

Hero: Left-aligned "FEATURES" with blur TextAnimation
Status badge: "● SYSTEM ONLINE" (DM Mono)

Bento grid (12-col CSS grid):
  Large cards (8/12): main features with AnimatedBeam section
  Medium cards (6/12): feature highlights
  Small cards (4/12): compact feature items
  Full-width card (12/12): spotlight feature

Each bento card:
  .grok-card style, CornerBorder
  GSAP ScanLine fires on scroll enter
  Icon (Lucide, 32px), title (Bebas Neue 22px), description (DM Mono 12px)
  Random stagger delay 0–400ms

AnimatedBeam section inside one of the large cards:
  section="features", reads from Firebase beam_nodes (section=='features')

Dot-grid parallax: 0.3x scroll speed (GSAP ScrollTrigger scrub)

Verify: Bento cards reveal on scroll. AnimatedBeam animates.
```

---

## PHASE 6 — MICRO-INTERACTIONS & POLISH

### TASK 6.1 — Button Micro-interactions
```
Enhance: src/components/ui/CPButton.tsx

All buttons:
  active:scale-95 on click (spring back)
  .shine-sweep hover effect
  CornerBorder (corner brackets)
  MagneticWrapper on primary CTAs

Primary button (variant="primary"):
  Background: white, text: black
  Hover: subtle glow ring (box-shadow 0 0 0 1px rgba(255,255,255,0.3))
  .border-sweep on hover

Ghost button (variant="ghost"):
  Border: rgba(255,255,255,0.2)
  Hover: border brightens + sweep shine

Icon button:
  Ripple pulse on hover (single ring)

Loading state:
  3 thinking-dot divs + "SENDING..." text
  Spinner replaces arrow icon

All: prefers-reduced-motion: no scale, no shine, just color change
```

### TASK 6.2 — Card Hover Effects
```
All project cards / service cards / FAQ items:
  translateY(-4px to -8px) on hover
  Border brightens (rgba(255,255,255,0.06 → 0.20))
  Corner brackets expand (width: 12px → 16px)
  .shine-sweep fires once on hover enter
  Transition: all 0.25s cubic-bezier(0.33, 1, 0.68, 1)

Image cards:
  image.scale 1.0 → 1.05 on hover (overflow:hidden on container)
  grayscale(20%) → grayscale(0%) on hover

Text reveal on hover:
  "VIEW CASE STUDY →" overlay on project cards
  Framer Motion: y: 20 → 0, opacity: 0 → 1

Tags/chips on hover:
  border-color brightens
  background: rgba(255,255,255,0 → 0.04)
```

### TASK 6.3 — Link Underline Animations
```
All nav links and inline text links:
  Custom underline via ::after pseudo element
  scaleX: 0 → 1 on hover (left to right)
  transform-origin: left
  height: 1px, background: white
  transition: transform 0.3s ease

Active nav link:
  framer-motion layoutId underline (slides between links)

Social links in footer/sidebar:
  Arrow "↗" translateX(2px) translateY(-2px) on hover
  opacity: 0.3 → 1 on hover
```

### TASK 6.4 — Form Micro-interactions
```
Contact form inputs:
  border-bottom brightens on focus (rgba(255,255,255,0.10 → 0.55))
  glow pulse on focus (box-shadow)
  Error: border-bottom becomes red/50%, red label appears with slide-in animation
  Valid: subtle green check icon fades in

Project type chips:
  spring scale (1 → 1.04) on click
  bg transitions: transparent → white (150ms)
  Text transitions: white/40 → black (150ms)
  Deselect: spring back

Submit button:
  Loading: width expands slightly, dots animate
  Success: checkmark draws in (SVG path animation)
  Error: shake animation (GSAP: x oscillates ±5px, 4 times)
```

### TASK 6.5 — Scroll-Triggered Parallax
```
Add to: Home hero, About hero, Work hero

Background elements (decorative large text, gradient orbs):
  GSAP ScrollTrigger scrub: 1
  y: 0 → -80px as section scrolls out

Foreground content:
  GSAP ScrollTrigger: y: 0 → -30px (slower than background = parallax)

Image cards in gallery:
  alternate: even cards scroll up 20px, odd cards scroll down 20px

Hero headline words:
  stagger: each word different scroll speed
  word1: y: 0 → -20px
  word2: y: 0 → -10px
  word3: y: 0 → -5px (barely moves — depth effect)
```

---

## PHASE 7 — FIREBASE FIRESTORE SETUP

### TASK 7.1 — Deploy Security Rules
```
Replace entire content of firestore.rules with:

rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /{document=**} {
      allow read: if true;
      allow write: if false;
    }
    match /contact_submissions/{id} {
      allow create: if
        request.resource.data.name is string &&
        request.resource.data.name.size() >= 2 &&
        request.resource.data.email is string &&
        request.resource.data.email.matches('[^@]+@[^@]+\\.[^@]+') &&
        request.resource.data.message is string &&
        request.resource.data.message.size() >= 10;
    }
    match /faqs/{id} {
      allow update: if
        request.resource.data.diff(resource.data)
          .affectedKeys().hasOnly(['helpful_count', 'not_helpful_count']);
    }
    match /projects/{id} {
      allow update: if
        request.resource.data.diff(resource.data)
          .affectedKeys().hasOnly(['view_count']);
    }
  }
}

Deploy: firebase deploy --only firestore:rules
```

### TASK 7.2 — Create All Collections in Firebase Console
```
Create these collections + seed documents (see FIREBASE_SETUP_GUIDE.md for exact fields):

Collection: portfolio_config
  Document: main (create manually, all fields listed in guide)

Collection: social_links
  Documents: 1 per social platform (GitHub, Twitter, LinkedIn, Dribbble)
  Critical field: url = FULL URL including https://

Collection: projects
  Documents: at least 3 projects with:
  cover_image_url: use real Unsplash URLs
  live_url: real URL (can be placeholder like https://example.com)
  featured: true for at least 2
  visible: true for all
  order: 1, 2, 3...

Collection: stats (4 documents)
Collection: skills (8+ documents, different categories)
Collection: experience (2+ documents)
Collection: faqs (5+ documents)
Collection: services (3+ documents)
Collection: awards (optional, 2+ documents)
Collection: process_steps (5 documents for the process section)
```

### TASK 7.3 — Add Composite Indexes
```
Go to Firebase Console → Firestore → Indexes → Create Index:

Index 1 (projects):
  Collection: projects
  Fields: visible (Ascending) + order (Ascending)

Index 2 (projects + featured):
  Collection: projects
  Fields: visible (Ascending) + featured (Ascending) + order (Ascending)

Index 3 (social_links):
  Collection: social_links
  Fields: visible (Ascending) + order (Ascending)

Index 4 (stats):
  Collection: stats
  Fields: visible (Ascending) + order (Ascending)

Index 5 (faqs):
  Collection: faqs
  Fields: visible (Ascending) + order (Ascending)

Index 6 (skills):
  Collection: skills
  Fields: visible (Ascending) + order (Ascending)

Index 7 (testimonials):
  Collection: testimonials
  Fields: visible (Ascending) + approved (Ascending) + order (Ascending)

NOTE: Firestore will show index errors in console when queries need them.
      Just click the link in the error message to auto-create.
```

---

## PHASE 8 — RESPONSIVENESS

### TASK 8.1 — Mobile Breakpoint Audit
```
Test every page at: 375px, 414px, 768px, 1024px, 1440px

Check per breakpoint:
  <640px (mobile):
    Navbar: logo + hamburger only
    All grids: 1 column
    Font sizes: at clamp minimum
    Three.js: disabled (static CSS fallback)
    Custom cursor: disabled (touch device)
    Section padding: 20px horizontal
    Contact: tabs (not side-by-side)
    Services: accordion only (no split panel)
    No glass effect (flat dark card)

  640-1024px (tablet):
    Stats: 2×2 grid
    Projects: 2 equal columns
    Footer: 2 columns
    About skills: stacked if not enough space

  1024px+ (desktop):
    Full layouts
    Glass panels enabled
    Split panels enabled
    Vertical social bar in hero

Fix any overflow: hidden on page-level containers.
No horizontal scroll at any breakpoint.
```

### TASK 8.2 — Image Optimization
```
All images must use Next.js <Image> component:
  - sizes prop set correctly per layout
  - priority={true} on above-fold images
  - placeholder="blur" with blurDataURL for skeleton effect

next.config.mjs must allow external image domains:
  images: {
    remotePatterns: [
      { protocol: 'https', hostname: 'images.unsplash.com' },
      { protocol: 'https', hostname: 'firebasestorage.googleapis.com' },
      { protocol: 'https', hostname: '*.cloudinary.com' },
    ]
  }
```

---

## PHASE 9 — FINAL QA

### TASK 9.1 — Build Verification
```
Run: npm run build

Must complete with:
  - Zero TypeScript errors
  - Zero ESLint errors that block build
  - All pages pre-rendered or marked dynamic correctly
  - No "hydration mismatch" warnings in dev

Common fixes needed:
  - Add 'use client' to any component using useState/useEffect
  - Add suppressHydrationWarning to elements showing live times
  - Ensure Three.js always uses dynamic import with ssr:false
  - Ensure cursor uses useEffect to check pointer:coarse (not SSR)
```

### TASK 9.2 — Firebase Connection Verification
```
Open the site with devtools open. Verify:
  1. No Firebase errors in console
  2. Network tab: Firestore WebSocket connected
  3. Change owner_email in Firebase → watch it update on Contact page within 3s
  4. Submit contact form → see document in Firestore console
  5. Vote on FAQ → see helpful_count increment in Firestore console
  6. Add a new social link in Firebase → appears in Footer/Navbar within 60s
  7. Set announcement_bar_enabled: true → announcement bar appears
```

### TASK 9.3 — Animation QA
```
Verify each animation type works:
  [ ] Custom cursor: smooth spring, particles on move, burst on click
  [ ] Page transition: pixel tiles animate between routes
  [ ] Hero text: scramble/wave/liquid on page load
  [ ] Scroll reveal: sections animate in as you scroll
  [ ] ScrollVelocityMarquee: speeds up/skews on fast scroll
  [ ] CountUp: numbers animate on scroll into view
  [ ] AnimatedBeam: SVG lines animate on Home and Features
  [ ] Ripple: rings expand behind hero CTA
  [ ] Project cards: hover effects, corner brackets
  [ ] Form chips: spring animation on select/deselect
  [ ] Page load: ColdPixelsIntro on first visit, skips on return
  [ ] Back-to-top: appears at 400px scroll, smooth scroll on click
  [ ] Scroll progress: bar fills as you scroll
  [ ] Mobile menu: stagger entrance animation
```

### TASK 9.4 — Performance Check
```
lighthouse audit: aim for:
  Performance: 85+
  Accessibility: 90+
  Best Practices: 90+
  SEO: 95+

Key optimizations:
  - Three.js: dynamic import ssr:false (don't server-render)
  - GSAP ScrollTrigger: only imported where needed
  - SWR: dedupingInterval 60000 (don't re-fetch every render)
  - onSnapshot listeners: always clean up (return unsub from useEffect)
  - Fonts: next/font/google (preloads, swaps, avoids layout shift)
  - Images: next/image (automatic optimization, WebP)
  - prefers-reduced-motion: respected by all animations
```

---

## SUMMARY — FILE COUNT
```
Total files to create or rewrite: 42

PHASE 0 (Foundation):    4 tasks
PHASE 1 (Cursor):        2 tasks → 1 new file
PHASE 2 (Layout):        6 tasks → 6 rewrites
PHASE 3 (Hooks):         5 tasks → 7 rewrites
PHASE 4 (Animations):    8 tasks → 4 rewrites + 2 new files
PHASE 5 (Pages):         8 tasks → 8 complete rewrites
PHASE 6 (Polish):        5 tasks → enhancements to existing files
PHASE 7 (Firebase):      3 tasks → Firestore setup (no code files)
PHASE 8 (Responsive):    2 tasks → fixes across all pages
PHASE 9 (QA):            4 tasks → verification only
```

---

## QUICK REFERENCE — FIREBASE FIELD → WHERE IT SHOWS

| Firebase Field                      | Shows On                                          |
|-------------------------------------|---------------------------------------------------|
| config.owner_email                  | Navbar, Footer, Contact (3 places), Autofill TO:  |
| config.owner_name                   | Navbar logo text, About, Footer                   |
| config.owner_title                  | About page h1                                     |
| config.owner_bio_short              | About hero, Footer brand column                   |
| config.owner_bio_long               | About biography section (split by \\n\\n)           |
| config.owner_location               | Contact sidebar, About hero, Footer               |
| config.availability_label           | Navbar, About hero, Footer                        |
| config.hero_headline_word1/2/3      | Home hero (3 animated lines)                      |
| config.hero_subheadline             | Home hero paragraph                               |
| config.footer_tagline               | Footer CTA belt (large text)                      |
| config.show_awards                  | About awards section: visible or hidden           |
| config.show_testimonials            | Home testimonials: visible or hidden              |
| config.announcement_bar_enabled     | Announcement bar: shows or hidden                 |
| config.announcement_bar_text        | Announcement bar message                          |
| config.contact_form_project_types   | Contact form chips + Autofill compose chips       |
| config.contact_form_budget_ranges   | Contact form budget chips                         |
| config.response_time                | Contact page info + footer copy                   |
| config.calendly_url                 | Contact sidebar + Footer "Book a call" button     |
| social_links[].url                  | Footer Social col, About Social, Contact sidebar  |
| social_links[].platform             | All above                                         |
| projects[].cover_image_url          | Work grid, Work detail hero                       |
| projects[].live_url                 | Work detail "VISIT LIVE SITE ↗" button            |
| projects[].github_url               | Work detail "VIEW CODE ↗" button                  |
| skills[].proficiency                | About skills animated bars (%)                    |
| faqs[].helpful_count                | FAQ accordion voting display                      |
| stats[].value + suffix              | Home StatsCounter, About stats grid               |
| awards[].url                        | About awards "View ↗" link                        |
| services[].features[]               | Services accordion expanded content               |
| services[].starting_price           | Services row + expanded content                   |