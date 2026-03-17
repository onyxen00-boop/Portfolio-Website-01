# coldpixels.co — COMPLETE FIX & REBUILD PROMPT
## Every Bug Fixed · Every Firebase Field Live · Every Page Long & Impressive
## Read Every Word — Do Not Skip Sections — Do Not Leave TODOs

---

## 🚨 DIAGNOSIS — WHY EVERYTHING IS BROKEN

After analyzing the uploaded zip, here are the exact problems to fix:

```
PROBLEM 1: All app/ pages (home, about, work, contact, services) are STATIC HTML
           They don't use Firebase hooks. Everything is hardcoded dummy data.
           FIX: Rewrite every page as 'use client', use Firebase hooks, show real data.

PROBLEM 2: Firebase collection name MISMATCH
           firestore.ts reads from 'config/default'
           But the correct collection/document is: 'portfolio_config' / 'main'
           FIX: Standardize ALL collection names (list below).

PROBLEM 3: NO getSocialLinks() function exists in firestore.ts
           Social links are embedded inside Config object which doesn't have them.
           FIX: Add getSocialLinks() that reads from 'social_links' collection.
           Add useSocialLinks() hook that reads from 'social_links' collection directly.

PROBLEM 4: NO getStats() function exists in firestore.ts
           Stats never load from Firebase.
           FIX: Add getStats() reading from 'stats' collection.

PROBLEM 5: Contact form has TODO comment — it never saves anything anywhere
           FIX: Save submissions to Firestore 'contact_submissions' collection.
           Also add email autofill compose feature.

PROBLEM 6: Spline integration — remove completely. No Spline anywhere.
           FIX: Delete all Spline imports/references/URLs. Replace with CSS animations.

PROBLEM 7: Page transition is just opacity fade — boring and doesn't look "loading"
           FIX: Replace with a full-screen pixel shatter transition (GSAP).

PROBLEM 8: Social links in footer and navbar are HARDCODED href="#"
           FIX: Pull from Firebase social_links collection and display real URLs.

PROBLEM 9: app/layout.tsx does NOT wrap children with Navbar, Footer, or PageTransition
           FIX: Add these to the root layout so they appear on every page.

PROBLEM 10: Images/links in projects not clickable or opening real pages
            FIX: Project live_url and github_url from Firebase displayed as real links.

PROBLEM 11: .env.local — Firebase config already uses NEXT_PUBLIC_ env vars correctly.
            But there is no .env.local.example file with clear instructions.
            FIX: Create .env.local.example with all required keys.
```

---

## ✅ STANDARDIZED FIREBASE COLLECTION NAMES

**CRITICAL: Use these exact collection/document names everywhere in code:**

```
portfolio_config / main          ← site config, owner info, hero text, toggles
projects / {auto-id}             ← portfolio projects
social_links / {auto-id}         ← GitHub, Twitter, LinkedIn, Dribbble etc.
stats / {auto-id}                ← project count, client count, years etc.
skills / {auto-id}               ← design/dev/tool skills with proficiency %
experience / {auto-id}           ← work history timeline
testimonials / {auto-id}         ← client quotes
faqs / {auto-id}                 ← FAQ questions and answers
services / {auto-id}             ← services offered
awards / {auto-id}               ← recognition and awards
process_steps / {auto-id}        ← process: discovery → design → dev → launch
contact_submissions / {auto-id}  ← form submissions (write-only for users)
```

---

## 📁 EVERY FILE TO CREATE OR REWRITE

### FILE 1: .env.local.example (create in project root)
```
# Firebase Configuration — copy this to .env.local and fill in your values
# Get these from: Firebase Console → Project Settings → Your apps → Web app

NEXT_PUBLIC_FIREBASE_API_KEY=
NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN=
NEXT_PUBLIC_FIREBASE_PROJECT_ID=
NEXT_PUBLIC_FIREBASE_STORAGE_BUCKET=
NEXT_PUBLIC_FIREBASE_MESSAGING_SENDER_ID=
NEXT_PUBLIC_FIREBASE_APP_ID=

# HOW TO FILL THIS:
# 1. Go to https://console.firebase.google.com
# 2. Click your project
# 3. Click the gear icon ⚙️ → Project Settings
# 4. Scroll down to "Your apps" → Web app (</> icon)
# 5. Copy each value from the firebaseConfig object
```

---

### FILE 2: src/lib/firebase.ts — COMPLETE REWRITE
```typescript
// IMPORTANT: All config comes from .env.local — NEVER hardcode values here
import { initializeApp, getApps } from 'firebase/app';
import { getFirestore } from 'firebase/firestore';

const firebaseConfig = {
  apiKey:            process.env.NEXT_PUBLIC_FIREBASE_API_KEY!,
  authDomain:        process.env.NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN!,
  projectId:         process.env.NEXT_PUBLIC_FIREBASE_PROJECT_ID!,
  storageBucket:     process.env.NEXT_PUBLIC_FIREBASE_STORAGE_BUCKET!,
  messagingSenderId: process.env.NEXT_PUBLIC_FIREBASE_MESSAGING_SENDER_ID!,
  appId:             process.env.NEXT_PUBLIC_FIREBASE_APP_ID!,
};

// Prevent duplicate initialization on hot reload
const app = getApps().length === 0 ? initializeApp(firebaseConfig) : getApps()[0];
export const db = getFirestore(app);
export default app;
```

---

### FILE 3: src/lib/firestore.ts — COMPLETE REWRITE

```typescript
import { db } from './firebase';
import {
  collection, doc, getDoc, getDocs, addDoc, updateDoc,
  query, where, orderBy, onSnapshot, increment, serverTimestamp,
  QueryConstraint,
} from 'firebase/firestore';

// ─── COLLECTION NAMES ──────────────────────────────────────────────────────
export const COLLECTIONS = {
  CONFIG:        'portfolio_config',  // doc id: 'main'
  PROJECTS:      'projects',
  SOCIAL_LINKS:  'social_links',
  STATS:         'stats',
  SKILLS:        'skills',
  EXPERIENCE:    'experience',
  TESTIMONIALS:  'testimonials',
  FAQS:          'faqs',
  SERVICES:      'services',
  AWARDS:        'awards',
  PROCESS:       'process_steps',
  SUBMISSIONS:   'contact_submissions',
} as const;

// ─── CONFIG ────────────────────────────────────────────────────────────────
export async function getConfig() {
  try {
    const ref = doc(db, COLLECTIONS.CONFIG, 'main');
    const snap = await getDoc(ref);
    return snap.exists() ? snap.data() : null;
  } catch (e) { console.error('[firebase] getConfig:', e); return null; }
}

export function listenToConfig(callback: (data: any) => void) {
  const ref = doc(db, COLLECTIONS.CONFIG, 'main');
  return onSnapshot(ref, (snap) => {
    if (snap.exists()) callback(snap.data());
  }, (e) => console.error('[firebase] listenToConfig:', e));
}

// ─── PROJECTS ──────────────────────────────────────────────────────────────
export async function getProjects(opts?: { featured?: boolean; category?: string }) {
  try {
    const constraints: QueryConstraint[] = [where('visible', '==', true)];
    if (opts?.featured !== undefined) constraints.push(where('featured', '==', opts.featured));
    if (opts?.category) constraints.push(where('category', '==', opts.category));
    constraints.push(orderBy('order', 'asc'));
    const q = query(collection(db, COLLECTIONS.PROJECTS), ...constraints);
    const snap = await getDocs(q);
    return snap.docs.map(d => ({ id: d.id, ...d.data() }));
  } catch (e) { console.error('[firebase] getProjects:', e); return []; }
}

export async function getProjectBySlug(slug: string) {
  try {
    const q = query(collection(db, COLLECTIONS.PROJECTS), where('slug', '==', slug));
    const snap = await getDocs(q);
    if (snap.empty) return null;
    const d = snap.docs[0];
    return { id: d.id, ...d.data() };
  } catch (e) { console.error('[firebase] getProjectBySlug:', e); return null; }
}

export async function incrementProjectViews(id: string) {
  try {
    await updateDoc(doc(db, COLLECTIONS.PROJECTS, id), { view_count: increment(1) });
  } catch {}
}

// ─── SOCIAL LINKS ──────────────────────────────────────────────────────────
// Social links are stored in their OWN collection (NOT inside config)
// Each document: { platform, url, icon, label, order, visible }
export async function getSocialLinks() {
  try {
    const q = query(
      collection(db, COLLECTIONS.SOCIAL_LINKS),
      where('visible', '==', true),
      orderBy('order', 'asc')
    );
    const snap = await getDocs(q);
    return snap.docs.map(d => ({ id: d.id, ...d.data() }));
  } catch (e) { console.error('[firebase] getSocialLinks:', e); return []; }
}

// ─── STATS ─────────────────────────────────────────────────────────────────
// Each document: { value, suffix, label, icon, order, visible }
export async function getStats() {
  try {
    const q = query(
      collection(db, COLLECTIONS.STATS),
      where('visible', '==', true),
      orderBy('order', 'asc')
    );
    const snap = await getDocs(q);
    return snap.docs.map(d => ({ id: d.id, ...d.data() }));
  } catch (e) { console.error('[firebase] getStats:', e); return []; }
}

export function listenToStats(callback: (data: any[]) => void) {
  const q = query(
    collection(db, COLLECTIONS.STATS),
    where('visible', '==', true),
    orderBy('order', 'asc')
  );
  return onSnapshot(q, (snap) => {
    callback(snap.docs.map(d => ({ id: d.id, ...d.data() })));
  });
}

// ─── SKILLS ────────────────────────────────────────────────────────────────
// Each document: { name, category, proficiency(0-100), years_experience, order, visible }
export async function getSkills() {
  try {
    const q = query(
      collection(db, COLLECTIONS.SKILLS),
      where('visible', '==', true),
      orderBy('order', 'asc')
    );
    const snap = await getDocs(q);
    return snap.docs.map(d => ({ id: d.id, ...d.data() }));
  } catch (e) { console.error('[firebase] getSkills:', e); return []; }
}

// ─── EXPERIENCE ────────────────────────────────────────────────────────────
// Each document: { company, role, type, start_date, end_date, description, achievements[], order, visible }
export async function getExperience() {
  try {
    const q = query(
      collection(db, COLLECTIONS.EXPERIENCE),
      where('visible', '==', true),
      orderBy('order', 'asc')
    );
    const snap = await getDocs(q);
    return snap.docs.map(d => ({ id: d.id, ...d.data() }));
  } catch (e) { console.error('[firebase] getExperience:', e); return []; }
}

// ─── TESTIMONIALS ──────────────────────────────────────────────────────────
// Each document: { name, role, company, avatar_url, quote, rating, approved, order, visible }
export async function getTestimonials() {
  try {
    const q = query(
      collection(db, COLLECTIONS.TESTIMONIALS),
      where('visible', '==', true),
      where('approved', '==', true),
      orderBy('order', 'asc')
    );
    const snap = await getDocs(q);
    return snap.docs.map(d => ({ id: d.id, ...d.data() }));
  } catch (e) { console.error('[firebase] getTestimonials:', e); return []; }
}

// ─── FAQs ──────────────────────────────────────────────────────────────────
// Each document: { question, answer, category, helpful_count, not_helpful_count, order, visible }
export async function getFAQs() {
  try {
    const q = query(
      collection(db, COLLECTIONS.FAQS),
      where('visible', '==', true),
      orderBy('order', 'asc')
    );
    const snap = await getDocs(q);
    return snap.docs.map(d => ({ id: d.id, ...d.data() }));
  } catch (e) { console.error('[firebase] getFAQs:', e); return []; }
}

export async function voteFAQ(id: string, helpful: boolean) {
  try {
    const field = helpful ? 'helpful_count' : 'not_helpful_count';
    await updateDoc(doc(db, COLLECTIONS.FAQS, id), { [field]: increment(1) });
  } catch {}
}

// ─── SERVICES ──────────────────────────────────────────────────────────────
// Each document: { name, tagline, description, features[], deliverables[], timeline, starting_price, popular, order, visible }
export async function getServices() {
  try {
    const q = query(
      collection(db, COLLECTIONS.SERVICES),
      where('visible', '==', true),
      orderBy('order', 'asc')
    );
    const snap = await getDocs(q);
    return snap.docs.map(d => ({ id: d.id, ...d.data() }));
  } catch (e) { console.error('[firebase] getServices:', e); return []; }
}

// ─── AWARDS ────────────────────────────────────────────────────────────────
// Each document: { title, organization, year, category, url, order, visible }
export async function getAwards() {
  try {
    const q = query(
      collection(db, COLLECTIONS.AWARDS),
      where('visible', '==', true),
      orderBy('order', 'asc')
    );
    const snap = await getDocs(q);
    return snap.docs.map(d => ({ id: d.id, ...d.data() }));
  } catch (e) { console.error('[firebase] getAwards:', e); return []; }
}

// ─── PROCESS STEPS ─────────────────────────────────────────────────────────
// Each document: { step_number, title, description, deliverables[], order, visible }
export async function getProcessSteps() {
  try {
    const q = query(
      collection(db, COLLECTIONS.PROCESS),
      where('visible', '==', true),
      orderBy('order', 'asc')
    );
    const snap = await getDocs(q);
    return snap.docs.map(d => ({ id: d.id, ...d.data() }));
  } catch (e) { console.error('[firebase] getProcessSteps:', e); return []; }
}

// ─── CONTACT FORM ──────────────────────────────────────────────────────────
// Saves to contact_submissions collection
// Security rule: create-only for users, read-only for admin
export async function submitContactForm(data: {
  name: string;
  email: string;
  project_type: string;
  budget?: string;
  message: string;
  timeline?: string;
}) {
  // Validate before writing
  if (!data.name || data.name.length < 2) throw new Error('Name too short');
  if (!data.email || !data.email.includes('@')) throw new Error('Invalid email');
  if (!data.message || data.message.length < 10) throw new Error('Message too short');

  const ref = await addDoc(collection(db, COLLECTIONS.SUBMISSIONS), {
    ...data,
    created_at: serverTimestamp(),
    status: 'new',
    source: 'website_contact_form',
  });
  return ref.id;
}
```

---

### FILE 4: src/hooks/useSocialLinks.ts — COMPLETE REWRITE
```typescript
'use client';
import useSWR from 'swr';
import { getSocialLinks } from '@/lib/firestore';

const MOCK = [
  { id: '1', platform: 'GitHub',   url: 'https://github.com',   icon: 'github',   label: 'GitHub',   order: 1, visible: true },
  { id: '2', platform: 'Twitter',  url: 'https://twitter.com',  icon: 'twitter',  label: 'Twitter',  order: 2, visible: true },
  { id: '3', platform: 'LinkedIn', url: 'https://linkedin.com', icon: 'linkedin', label: 'LinkedIn', order: 3, visible: true },
  { id: '4', platform: 'Dribbble', url: 'https://dribbble.com', icon: 'dribbble', label: 'Dribbble', order: 4, visible: true },
];

export function useSocialLinks() {
  const { data, error, isLoading } = useSWR(
    'social_links',
    async () => {
      const result = await getSocialLinks();
      return result.length > 0 ? result : MOCK;
    },
    { fallbackData: MOCK, revalidateOnFocus: false, dedupingInterval: 60000 }
  );
  return { socialLinks: data ?? MOCK, loading: isLoading, error: !!error };
}
```

---

### FILE 5: src/hooks/useStats.ts — COMPLETE REWRITE
```typescript
'use client';
import { useState, useEffect } from 'react';
import { listenToStats } from '@/lib/firestore';

const MOCK = [
  { id: '1', value: 50,  suffix: '+', label: 'Projects Completed',  icon: 'briefcase', order: 1, visible: true },
  { id: '2', value: 30,  suffix: '+', label: 'Happy Clients',       icon: 'users',     order: 2, visible: true },
  { id: '3', value: 5,   suffix: '+', label: 'Years Experience',    icon: 'calendar',  order: 3, visible: true },
  { id: '4', value: 100, suffix: '%', label: 'Client Satisfaction', icon: 'star',      order: 4, visible: true },
];

export function useStats() {
  const [stats, setStats] = useState(MOCK);
  useEffect(() => {
    const unsub = listenToStats((data) => {
      if (data.length > 0) setStats(data as any);
    });
    return () => unsub();
  }, []);
  return { stats };
}
```

---

### FILE 6: src/hooks/useConfig.ts — COMPLETE REWRITE
```typescript
'use client';
import { useState, useEffect } from 'react';
import { listenToConfig } from '@/lib/firestore';

export const MOCK_CONFIG = {
  // Owner info
  owner_name: 'coldpixels.co',
  owner_title: 'Digital Designer & Developer',
  owner_bio_short: 'I design and build premium digital products.',
  owner_bio_long: 'coldpixels.co is a digital craft studio focused on precision, performance, and pixel-perfect execution. I work with ambitious brands to create digital experiences that resonate.',
  owner_email: 'hello@coldpixels.co',
  owner_phone: '+1 (555) 000-0000',
  owner_location: 'New York, NY',
  owner_timezone: 'EST',
  // Availability
  availability_status: 'open',        // 'open' | 'busy' | 'closed'
  availability_label: 'Available for new projects',
  // Hero
  hero_headline_word1: 'DESIGN',
  hero_headline_word2: 'DEVELOP',
  hero_headline_word3: 'DELIVER',
  hero_subheadline: 'Precision in every pixel. Excellence in every interaction.',
  hero_cta_primary_label: 'View My Work',
  hero_cta_primary_url: '/work',
  hero_cta_secondary_label: 'Get In Touch',
  cv_url: '/cv.pdf',
  // About page
  about_headline: 'Building the web with cold precision.',
  // Marquee
  marquee_items: ['Design Systems', 'Motion Design', 'Web Development', 'Brand Identity', 'UI/UX', 'Creative Direction'],
  // Footer
  footer_tagline: "Let's build something remarkable.",
  footer_copy: 'Crafted with precision by coldpixels.co',
  copyright_year: new Date().getFullYear(),
  // Feature toggles
  show_testimonials: true,
  show_awards: true,
  announcement_bar_enabled: false,
  announcement_bar_text: 'Now accepting new projects for 2025',
  announcement_bar_link: '/contact',
  // Contact
  contact_form_project_types: ['UI/UX Design', 'Web Development', 'Brand Identity', 'Motion Design', 'Full-Stack App', 'Creative Direction', 'Something Else'],
  contact_form_budget_ranges: ['< $5K', '$5K–$15K', '$15K–$30K', '$30K–$60K', '$60K+'],
  response_time: 'Usually responds within 24 hours',
  // Calendly
  calendly_url: '',
};

export function useConfig() {
  const [config, setConfig] = useState<typeof MOCK_CONFIG>(MOCK_CONFIG);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    const unsub = listenToConfig((data) => {
      if (data) setConfig({ ...MOCK_CONFIG, ...data });
      setLoading(false);
    });
    // Also handle no Firebase connection
    const timeout = setTimeout(() => setLoading(false), 3000);
    return () => { unsub(); clearTimeout(timeout); };
  }, []);

  return { config, loading };
}
```

---

### FILE 7: src/hooks/useProjects.ts — COMPLETE REWRITE
```typescript
'use client';
import useSWR from 'swr';
import { getProjects, getProjectBySlug } from '@/lib/firestore';

export const MOCK_PROJECTS = [
  {
    id: '1', slug: 'digital-transformation',
    title: 'Digital Transformation Suite',
    subtitle: 'Enterprise platform redesign',
    short_description: 'Complete redesign of a legacy enterprise dashboard serving 50,000+ users.',
    category: 'Web Development', tags: ['React', 'Next.js', 'TypeScript'],
    cover_image_url: 'https://images.unsplash.com/photo-1555066931-4365d14bab8c?w=1200&q=80',
    thumbnail_url: 'https://images.unsplash.com/photo-1555066931-4365d14bab8c?w=800&q=80',
    client: 'TechCorp Inc', year: 2024, role: 'Lead Designer & Developer',
    duration: '4 months', featured: true, featured_order: 1,
    visible: true, order: 1, view_count: 0,
    challenge: 'The existing platform was built in 2012 and could not handle modern UX expectations.',
    approach: 'We audited 200+ user sessions, rebuilt the component library, and migrated to Next.js.',
    outcome: 'Page load time dropped 60%. User satisfaction score rose from 3.2 to 4.8.',
    live_url: 'https://example.com',
    github_url: '',
    tech_stack: ['React', 'Next.js', 'TypeScript', 'Tailwind', 'Firebase'],
    gallery: [],
  },
  {
    id: '2', slug: 'brand-identity-system',
    title: 'Brand Identity System',
    subtitle: 'Complete visual identity',
    short_description: 'Full rebrand for a creative agency including logo, type, and digital guidelines.',
    category: 'Brand Identity', tags: ['Figma', 'Illustrator', 'Motion'],
    cover_image_url: 'https://images.unsplash.com/photo-1561070791-2526d30994b5?w=1200&q=80',
    thumbnail_url: 'https://images.unsplash.com/photo-1561070791-2526d30994b5?w=800&q=80',
    client: 'Studio Creative', year: 2024, role: 'Brand Designer',
    duration: '6 weeks', featured: true, featured_order: 2,
    visible: true, order: 2, view_count: 0,
    challenge: 'The agency had no consistent brand voice across digital and print.',
    approach: 'Built a modular brand system with clear usage rules and an 80-page brand book.',
    outcome: 'Brand recognition doubled in 3 months. Won two design awards.',
    live_url: 'https://example.com',
    github_url: '',
    tech_stack: ['Figma', 'Illustrator', 'After Effects'],
    gallery: [],
  },
];

export function useProjects(opts?: { featured?: boolean; category?: string }) {
  const key = ['projects', opts?.featured, opts?.category];
  const { data, error, isLoading } = useSWR(key, async () => {
    const result = await getProjects(opts as any);
    return result.length > 0 ? result : MOCK_PROJECTS;
  }, { fallbackData: MOCK_PROJECTS, revalidateOnFocus: false, dedupingInterval: 60000 });
  return { projects: data ?? MOCK_PROJECTS, loading: isLoading, error: !!error };
}

export function useProject(slug: string) {
  const { data, error, isLoading } = useSWR(
    slug ? ['project', slug] : null,
    async () => {
      const result = await getProjectBySlug(slug);
      return result ?? MOCK_PROJECTS.find(p => p.slug === slug) ?? null;
    },
    { revalidateOnFocus: false }
  );
  return { project: data, loading: isLoading, error: !!error };
}
```

---

### FILE 8: src/components/layout/PageTransition.tsx — COOL PIXEL SHATTER

**DELETE THE OLD VERSION. Write this complete file:**

```tsx
'use client';

import { useEffect, useRef } from 'react';
import { usePathname } from 'next/navigation';
import gsap from 'gsap';

const COLS = 8;
const ROWS = 5;

export function PageTransition() {
  const pathname = usePathname();
  const prevPath = useRef(pathname);
  const tiles = useRef<HTMLDivElement[]>([]);
  const container = useRef<HTMLDivElement>(null);
  const isFirstRender = useRef(true);

  useEffect(() => {
    if (isFirstRender.current) {
      isFirstRender.current = false;
      // Entry animation on first load
      gsap.fromTo(
        tiles.current,
        { opacity: 1, scale: 1 },
        {
          opacity: 0, scale: 0.6,
          duration: 0.4,
          stagger: { amount: 0.3, from: 'center' },
          ease: 'power2.in',
          onComplete: () => {
            if (container.current) container.current.style.pointerEvents = 'none';
          }
        }
      );
      return;
    }

    if (pathname !== prevPath.current) {
      prevPath.current = pathname;
      if (container.current) container.current.style.pointerEvents = 'all';

      // Tiles slam in from top, hold, then shatter out
      gsap.timeline()
        .fromTo(tiles.current, 
          { y: '-100%', opacity: 1, scale: 1 },
          { y: '0%', duration: 0.5, stagger: { amount: 0.2, from: 'random' }, ease: 'power3.out' }
        )
        .to({}, { duration: 0.1 }) // brief hold
        .to(tiles.current, {
          y: '100%', opacity: 0, scale: 0.5,
          duration: 0.5,
          stagger: { amount: 0.25, from: 'random' },
          ease: 'power3.in',
          onComplete: () => {
            if (container.current) container.current.style.pointerEvents = 'none';
            // Reset for next use
            gsap.set(tiles.current, { y: '-100%', opacity: 1, scale: 1 });
          }
        });
    }
  }, [pathname]);

  const tileCount = COLS * ROWS;
  return (
    <div
      ref={container}
      className="fixed inset-0 z-[99998] pointer-events-none"
      aria-hidden
    >
      <div
        className="w-full h-full grid"
        style={{ gridTemplateColumns: `repeat(${COLS}, 1fr)`, gridTemplateRows: `repeat(${ROWS}, 1fr)` }}
      >
        {Array.from({ length: tileCount }).map((_, i) => (
          <div
            key={i}
            ref={(el) => { if (el) tiles.current[i] = el; }}
            className="bg-[#080808]"
            style={{ transform: 'translateY(-100%)' }}
          />
        ))}
      </div>
    </div>
  );
}
```

---

### FILE 9: app/layout.tsx — COMPLETE REWRITE

**This wraps ALL pages with Navbar, Footer, PageTransition, SmoothScroll:**

```tsx
import type { Metadata, Viewport } from 'next';
import {
  Bebas_Neue, DM_Mono, Playfair_Display, Instrument_Serif
} from 'next/font/google';
import './globals.css';
import { Navbar } from '@/components/layout/Navbar';
import { Footer } from '@/components/layout/Footer';
import { PageTransition } from '@/components/layout/PageTransition';
import { AnnouncementBar } from '@/components/layout/AnnouncementBar';
import { ScrollProgress } from '@/components/layout/ScrollProgress';
import { BackToTop } from '@/components/layout/BackToTop';
import { SmoothScrollProvider } from '@/components/layout/SmoothScrollProvider';
import { ColdPixelsIntro } from '@/components/layout/ColdPixelsIntro';
import { Toaster } from 'react-hot-toast';

const bebasNeue = Bebas_Neue({ subsets: ['latin'], weight: '400', variable: '--font-display' });
const dmMono = DM_Mono({ subsets: ['latin'], weight: ['400', '500'], variable: '--font-mono' });
const playfair = Playfair_Display({ subsets: ['latin'], style: ['normal', 'italic'], variable: '--font-serif' });
const instrumentSerif = Instrument_Serif({ subsets: ['latin'], weight: '400', variable: '--font-body' });

export const metadata: Metadata = {
  title: { default: 'coldpixels.co — Digital Craft Studio', template: '%s | coldpixels.co' },
  description: 'Premium digital design & development. Building beautiful digital experiences with precision.',
  openGraph: {
    type: 'website', siteName: 'coldpixels.co',
    url: 'https://coldpixels.co',
  },
};

export const viewport: Viewport = {
  width: 'device-width', initialScale: 1,
  themeColor: '#080808',
};

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en" suppressHydrationWarning>
      <body className={`${bebasNeue.variable} ${dmMono.variable} ${playfair.variable} ${instrumentSerif.variable} bg-[#080808] text-white antialiased`}>
        <SmoothScrollProvider>
          <ColdPixelsIntro />
          <PageTransition />
          <AnnouncementBar />
          <ScrollProgress />
          <Navbar />
          <main id="main-content">{children}</main>
          <Footer />
          <BackToTop />
          <Toaster position="bottom-right" toastOptions={{ style: { background: '#111', color: '#fff', border: '1px solid rgba(255,255,255,0.1)' } }} />
        </SmoothScrollProvider>
      </body>
    </html>
  );
}
```

---

### FILE 10: app/page.tsx — COMPLETE REWRITE (Home)

**Remove navigation from this file — it's in layout.tsx now**

```tsx
import { HeroSection } from '@/components/sections/HeroSection';
import { KineticMarquee } from '@/components/sections/KineticMarquee';
import { FeaturedProjects } from '@/components/sections/FeaturedProjects';
import { StatsCounter } from '@/components/sections/StatsCounter';
import { TestimonialsSection } from '@/components/sections/TestimonialsSection';
import { CTABanner } from '@/components/sections/CTABanner';
import type { Metadata } from 'next';

export const metadata: Metadata = {
  title: 'coldpixels.co — Digital Craft Studio',
};

export default function Home() {
  return (
    <>
      <HeroSection />
      <KineticMarquee />
      <FeaturedProjects />
      <StatsCounter />
      <TestimonialsSection />
      <CTABanner />
    </>
  );
}
```

---

### FILE 11: app/about/page.tsx — COMPLETE REWRITE (Firebase-connected)

**This page is LONG — 6 full sections. Every field pulls from Firebase.**

```tsx
'use client';

import { useEffect, useRef } from 'react';
import gsap from 'gsap';
import { ScrollTrigger } from 'gsap/ScrollTrigger';
import { useConfig } from '@/hooks/useConfig';
import { useSkills } from '@/hooks/useSkills';
import { useExperience } from '@/hooks/useExperience';
import { useAwards } from '@/hooks/useAwards';
import { useSocialLinks } from '@/hooks/useSocialLinks';
import { useStats } from '@/hooks/useStats';
import { motion } from 'framer-motion';

gsap.registerPlugin(ScrollTrigger);

// ─── SECTION 1: HERO ───────────────────────────────────────────────────────
// Shows: config.owner_name, config.owner_title, config.owner_bio_short
// Shows: config.availability_status (as pulsing dot + label)
// Shows: config.owner_location, config.owner_timezone

// ─── SECTION 2: BIOGRAPHY (long form) ─────────────────────────────────────
// Shows: config.owner_bio_long (full markdown)
// Stats inline: useStats() → value + suffix + label

// ─── SECTION 3: SKILLS ─────────────────────────────────────────────────────
// Shows: useSkills() → 3 tabs (Design, Development, Tools)
// Each skill: name + animated fill bar (proficiency%) + years_experience

// ─── SECTION 4: EXPERIENCE TIMELINE ───────────────────────────────────────
// Shows: useExperience() → company, role, type, start_date, end_date, achievements[]

// ─── SECTION 5: AWARDS ─────────────────────────────────────────────────────
// Shows: useAwards() → title, organization, year (only if config.show_awards)

// ─── SECTION 6: SOCIAL LINKS ───────────────────────────────────────────────
// Shows: useSocialLinks() → platform, url (as real clickable links)

export default function AboutPage() {
  const { config } = useConfig();
  const { skills } = useSkills();
  const { experience } = useExperience();
  const { awards } = useAwards();
  const { socialLinks } = useSocialLinks();
  const { stats } = useStats();

  const sectionRef = useRef<HTMLDivElement>(null);

  useEffect(() => {
    gsap.registerPlugin(ScrollTrigger);
    // Animate every [data-reveal] element into view
    const elements = document.querySelectorAll('[data-reveal]');
    elements.forEach((el) => {
      gsap.fromTo(el, 
        { y: 40, opacity: 0 },
        { y: 0, opacity: 1, duration: 0.8, ease: 'power3.out',
          scrollTrigger: { trigger: el, start: 'top 88%', once: true } }
      );
    });
    return () => { ScrollTrigger.getAll().forEach(t => t.kill()); };
  }, []);

  const designSkills = skills.filter((s: any) => s.category === 'Design');
  const devSkills    = skills.filter((s: any) => s.category === 'Development');
  const toolSkills   = skills.filter((s: any) => s.category === 'Tools');

  return (
    <div className="min-h-screen">

      {/* ── SECTION 1: HERO ─────────────────────────────── */}
      <section className="relative pt-40 pb-24 px-6 md:px-12 lg:px-20 border-b border-white/6 overflow-hidden">
        {/* Large decorative text */}
        <div className="absolute inset-0 flex items-center justify-center pointer-events-none select-none overflow-hidden">
          <span className="font-display text-[20vw] text-white/[0.025] leading-none tracking-tighter">ABOUT</span>
        </div>
        <div className="relative max-w-7xl mx-auto">
          <div className="grid lg:grid-cols-2 gap-16 items-center">
            <div>
              {/* Availability dot */}
              <div className="flex items-center gap-2 mb-6">
                <span className="relative flex h-2 w-2">
                  <span className="animate-ping absolute inline-flex h-full w-full rounded-full bg-white opacity-40"></span>
                  <span className="relative inline-flex rounded-full h-2 w-2 bg-white"></span>
                </span>
                {/* This comes from Firebase: config.availability_label */}
                <span className="text-xs font-mono text-white/40 tracking-widest uppercase">
                  {config.availability_label}
                </span>
              </div>
              
              {/* Name — from Firebase: config.owner_name */}
              <p className="text-xs font-mono text-white/30 tracking-[0.2em] uppercase mb-3">
                {config.owner_name}
              </p>
              
              {/* Title — from Firebase: config.owner_title */}
              <h1 className="font-display text-[clamp(40px,6vw,80px)] leading-[0.92] tracking-tight mb-8">
                {config.owner_title ?? 'Digital Designer & Developer'}
              </h1>
              
              {/* Short bio — from Firebase: config.owner_bio_short */}
              <p className="text-lg text-white/60 font-body leading-relaxed max-w-lg mb-12">
                {config.owner_bio_short}
              </p>
              
              {/* Location + timezone — from Firebase */}
              <div className="flex gap-8 text-sm font-mono text-white/30">
                <span>📍 {config.owner_location}</span>
                <span>🕐 {config.owner_timezone}</span>
              </div>
            </div>
            
            {/* Right — Stats from Firebase useStats() */}
            <div className="grid grid-cols-2 gap-px bg-white/6 border border-white/6">
              {stats.map((stat: any, i: number) => (
                <div key={stat.id} className="bg-[#080808] p-8">
                  {/* value + suffix — from Firebase: stat.value, stat.suffix */}
                  <div className="font-display text-5xl mb-2">{stat.value}{stat.suffix}</div>
                  {/* label — from Firebase: stat.label */}
                  <div className="text-xs font-mono text-white/40 uppercase tracking-widest">{stat.label}</div>
                </div>
              ))}
            </div>
          </div>
        </div>
      </section>

      {/* ── SECTION 2: BIOGRAPHY ────────────────────────── */}
      <section className="py-32 px-6 md:px-12 lg:px-20 border-b border-white/6">
        <div className="max-w-7xl mx-auto grid lg:grid-cols-[1fr,2fr] gap-20">
          <div data-reveal>
            <p className="text-xs font-mono text-white/30 tracking-[0.2em] uppercase mb-4">01 — Story</p>
            <h2 className="font-display text-[clamp(32px,5vw,64px)] leading-[0.95]">The person behind the pixels</h2>
          </div>
          <div data-reveal>
            {/* Long bio — from Firebase: config.owner_bio_long */}
            <div className="text-white/70 font-body text-lg leading-relaxed space-y-6">
              {(config.owner_bio_long ?? '').split('\n\n').map((para: string, i: number) => (
                <p key={i}>{para}</p>
              ))}
            </div>
          </div>
        </div>
      </section>

      {/* ── SECTION 3: SKILLS ───────────────────────────── */}
      <section className="py-32 px-6 md:px-12 lg:px-20 border-b border-white/6">
        <div className="max-w-7xl mx-auto">
          <div className="mb-16" data-reveal>
            <p className="text-xs font-mono text-white/30 tracking-[0.2em] uppercase mb-4">02 — Expertise</p>
            <h2 className="font-display text-[clamp(32px,5vw,64px)] leading-[0.95]">Skills & Tools</h2>
          </div>
          
          {/* Skills come from Firebase useSkills() — category: Design | Development | Tools */}
          <div className="grid md:grid-cols-3 gap-12">
            {[
              { label: 'Design', items: designSkills },
              { label: 'Development', items: devSkills },
              { label: 'Tools', items: toolSkills },
            ].map((group) => (
              <div key={group.label} data-reveal>
                <h3 className="text-xs font-mono text-white/30 tracking-[0.2em] uppercase mb-8 border-b border-white/6 pb-4">
                  {group.label}
                </h3>
                <div className="space-y-6">
                  {group.items.map((skill: any) => (
                    <div key={skill.id}>
                      <div className="flex justify-between mb-2">
                        {/* name — from Firebase: skill.name */}
                        <span className="text-sm font-mono text-white/80">{skill.name}</span>
                        {/* proficiency — from Firebase: skill.proficiency (0-100) */}
                        <span className="text-xs font-mono text-white/30">{skill.proficiency}%</span>
                      </div>
                      <div className="h-px bg-white/8 relative overflow-hidden">
                        <motion.div
                          className="absolute inset-y-0 left-0 bg-white"
                          initial={{ width: 0 }}
                          whileInView={{ width: `${skill.proficiency}%` }}
                          viewport={{ once: true }}
                          transition={{ duration: 1.2, ease: [0.33, 1, 0.68, 1], delay: 0.2 }}
                        />
                      </div>
                      {/* years_experience — from Firebase: skill.years_experience */}
                      {skill.years_experience && (
                        <p className="text-xs text-white/20 mt-1 font-mono">{skill.years_experience} yrs</p>
                      )}
                    </div>
                  ))}
                  {group.items.length === 0 && (
                    <p className="text-sm text-white/20 font-mono">Add {group.label.toLowerCase()} skills in Firebase</p>
                  )}
                </div>
              </div>
            ))}
          </div>
        </div>
      </section>

      {/* ── SECTION 4: EXPERIENCE ───────────────────────── */}
      <section className="py-32 px-6 md:px-12 lg:px-20 border-b border-white/6">
        <div className="max-w-7xl mx-auto">
          <div className="mb-16" data-reveal>
            <p className="text-xs font-mono text-white/30 tracking-[0.2em] uppercase mb-4">03 — History</p>
            <h2 className="font-display text-[clamp(32px,5vw,64px)] leading-[0.95]">Work Experience</h2>
          </div>
          <div className="relative">
            {/* Vertical timeline line */}
            <div className="absolute left-0 top-0 bottom-0 w-px bg-white/8 hidden md:block" />
            <div className="space-y-16">
              {/* Each entry comes from Firebase useExperience() */}
              {experience.map((exp: any, i: number) => (
                <div key={exp.id} className="md:pl-12 relative" data-reveal>
                  {/* Timeline dot */}
                  <div className="absolute left-[-4px] top-2 hidden md:block">
                    <div className="w-2 h-2 rounded-full bg-white/20 border border-white/10" />
                  </div>
                  <div className="flex flex-col md:flex-row md:items-start md:justify-between gap-4 mb-4">
                    <div>
                      {/* role — from Firebase: exp.role */}
                      <h3 className="text-xl font-display tracking-tight">{exp.role}</h3>
                      {/* company — from Firebase: exp.company */}
                      <p className="text-white/60 font-mono text-sm mt-1">{exp.company}</p>
                    </div>
                    <div className="text-right">
                      {/* start_date, end_date — from Firebase */}
                      <p className="text-xs font-mono text-white/30 tracking-widest whitespace-nowrap">
                        {exp.start_date} — {exp.end_date ?? 'Present'}
                      </p>
                      {/* type — from Firebase: exp.type */}
                      <span className="inline-block mt-1 text-[10px] font-mono text-white/20 border border-white/8 px-2 py-0.5 rounded-sm uppercase tracking-wider">
                        {exp.type}
                      </span>
                    </div>
                  </div>
                  {/* description — from Firebase: exp.description */}
                  <p className="text-white/50 font-body leading-relaxed mb-4">{exp.description}</p>
                  {/* achievements[] — from Firebase: exp.achievements */}
                  {exp.achievements?.length > 0 && (
                    <ul className="space-y-2">
                      {exp.achievements.map((a: string, j: number) => (
                        <li key={j} className="flex gap-3 text-sm text-white/40 font-body">
                          <span className="text-white/20 mt-1">→</span>
                          <span>{a}</span>
                        </li>
                      ))}
                    </ul>
                  )}
                </div>
              ))}
              {experience.length === 0 && (
                <p className="text-white/20 font-mono text-sm md:pl-12">Add experience entries in Firebase → experience collection</p>
              )}
            </div>
          </div>
        </div>
      </section>

      {/* ── SECTION 5: AWARDS ─────────────────────────────── */}
      {config.show_awards && (
        <section className="py-32 px-6 md:px-12 lg:px-20 border-b border-white/6">
          <div className="max-w-7xl mx-auto">
            <div className="mb-16" data-reveal>
              <p className="text-xs font-mono text-white/30 tracking-[0.2em] uppercase mb-4">04 — Recognition</p>
              <h2 className="font-display text-[clamp(32px,5vw,64px)] leading-[0.95]">Awards & Recognition</h2>
            </div>
            <div className="grid md:grid-cols-2 lg:grid-cols-3 gap-px bg-white/6 border border-white/6">
              {/* Awards come from Firebase useAwards() */}
              {awards.map((award: any) => (
                <div key={award.id} className="bg-[#080808] p-8 group hover:bg-white/2 transition-colors" data-reveal>
                  {/* year — from Firebase: award.year */}
                  <div className="text-xs font-mono text-white/20 mb-4">{award.year}</div>
                  {/* title — from Firebase: award.title */}
                  <h3 className="font-display text-xl mb-2">{award.title}</h3>
                  {/* organization — from Firebase: award.organization */}
                  <p className="text-sm text-white/40 font-mono">{award.organization}</p>
                  {/* url — from Firebase: award.url — clickable if present */}
                  {award.url && (
                    <a href={award.url} target="_blank" rel="noopener noreferrer"
                      className="inline-block mt-4 text-xs font-mono text-white/20 hover:text-white/60 transition-colors">
                      View ↗
                    </a>
                  )}
                </div>
              ))}
              {awards.length === 0 && (
                <div className="bg-[#080808] p-8 col-span-3">
                  <p className="text-white/20 font-mono text-sm">Add awards in Firebase → awards collection. Set config.show_awards = true to show this section.</p>
                </div>
              )}
            </div>
          </div>
        </section>
      )}

      {/* ── SECTION 6: CONNECT (Social Links) ─────────────── */}
      <section className="py-32 px-6 md:px-12 lg:px-20">
        <div className="max-w-7xl mx-auto">
          <div className="mb-16" data-reveal>
            <p className="text-xs font-mono text-white/30 tracking-[0.2em] uppercase mb-4">05 — Connect</p>
            <h2 className="font-display text-[clamp(32px,5vw,64px)] leading-[0.95]">Find Me Online</h2>
          </div>
          {/* Social links — from Firebase: social_links collection */}
          {/* Each link: platform, url, icon, label */}
          {/* url is a REAL clickable link — opens in new tab */}
          <div className="grid sm:grid-cols-2 lg:grid-cols-4 gap-px bg-white/6 border border-white/6">
            {socialLinks.map((link: any) => (
              <a
                key={link.id}
                href={link.url}          {/* REAL URL from Firebase */}
                target="_blank"
                rel="noopener noreferrer"
                className="bg-[#080808] p-8 group hover:bg-white/2 transition-all duration-300 block"
                data-reveal
              >
                <p className="text-xs font-mono text-white/20 tracking-widest uppercase mb-4">
                  {link.platform}
                </p>
                <p className="font-display text-2xl group-hover:translate-x-1 transition-transform">
                  {link.label ?? link.platform}
                </p>
                <p className="text-xs font-mono text-white/20 mt-2 break-all">
                  {link.url}             {/* Shows the URL so user knows it's connected */}
                </p>
                <span className="block mt-4 text-white/20 group-hover:text-white/60 transition-colors">↗</span>
              </a>
            ))}
          </div>
        </div>
      </section>
    </div>
  );
}
```

---

### FILE 12: app/work/page.tsx — COMPLETE REWRITE (Firebase-connected)

```tsx
'use client';

import { useState, useEffect } from 'react';
import Link from 'next/link';
import Image from 'next/image';
import { useProjects } from '@/hooks/useProjects';
import { motion, AnimatePresence } from 'framer-motion';

export default function WorkPage() {
  const { projects } = useProjects();
  const [activeCategory, setActiveCategory] = useState('All');
  
  // Categories derived from actual project data
  const categories = ['All', ...Array.from(new Set(projects.map((p: any) => p.category).filter(Boolean)))];
  
  const filtered = activeCategory === 'All'
    ? projects
    : projects.filter((p: any) => p.category === activeCategory);

  return (
    <div className="min-h-screen pt-28">
      
      {/* HERO */}
      <section className="pt-20 pb-16 px-6 md:px-12 lg:px-20 border-b border-white/6">
        <div className="max-w-7xl mx-auto">
          <div className="relative">
            <span className="absolute -top-8 left-0 font-display text-[25vw] text-white/[0.02] leading-none select-none pointer-events-none">WORK</span>
            <p className="text-xs font-mono text-white/30 tracking-[0.2em] uppercase mb-4 relative">Selected Projects</p>
            <h1 className="font-display text-[clamp(48px,8vw,120px)] leading-[0.9] tracking-tight relative">
              {projects.length} Projects<br />Built With Precision
            </h1>
          </div>
        </div>
      </section>

      {/* FILTER BAR */}
      <div className="sticky top-20 z-30 border-b border-white/6 bg-[#080808]/95 backdrop-blur-xl">
        <div className="max-w-7xl mx-auto px-6 md:px-12 lg:px-20 py-4 flex gap-2 overflow-x-auto">
          {categories.map((cat) => (
            <button
              key={cat}
              onClick={() => setActiveCategory(cat)}
              className={`whitespace-nowrap px-4 py-2 text-xs font-mono tracking-widest uppercase transition-all rounded-sm ${
                activeCategory === cat
                  ? 'bg-white text-black'
                  : 'text-white/40 hover:text-white border border-white/10 hover:border-white/20'
              }`}
            >
              {cat}
            </button>
          ))}
        </div>
      </div>

      {/* PROJECT GRID — All data from Firebase */}
      <section className="py-20 px-6 md:px-12 lg:px-20">
        <div className="max-w-7xl mx-auto">
          <AnimatePresence mode="popLayout">
            <div className="grid md:grid-cols-2 gap-px bg-white/6">
              {filtered.map((project: any, i: number) => (
                <motion.div
                  key={project.id}
                  layout
                  initial={{ opacity: 0, y: 20 }}
                  animate={{ opacity: 1, y: 0 }}
                  exit={{ opacity: 0, scale: 0.95 }}
                  transition={{ duration: 0.4, delay: i * 0.05 }}
                >
                  <Link href={`/work/${project.slug}`} className="group block bg-[#080808] hover:bg-[#0d0d0d] transition-colors">
                    {/* cover_image_url — from Firebase: project.cover_image_url */}
                    <div className="aspect-[16/9] overflow-hidden relative">
                      {project.cover_image_url ? (
                        <Image
                          src={project.cover_image_url}
                          alt={project.title}
                          fill
                          className="object-cover grayscale-[20%] group-hover:grayscale-0 group-hover:scale-105 transition-all duration-700"
                          sizes="(max-width: 768px) 100vw, 50vw"
                        />
                      ) : (
                        <div className="w-full h-full bg-white/4 flex items-center justify-center">
                          <p className="text-xs font-mono text-white/20">Add cover_image_url in Firebase</p>
                        </div>
                      )}
                      {/* Hover overlay */}
                      <div className="absolute inset-0 bg-black/60 opacity-0 group-hover:opacity-100 transition-opacity flex items-center justify-center">
                        <span className="font-display text-2xl tracking-wider">VIEW CASE STUDY →</span>
                      </div>
                    </div>
                    <div className="p-8">
                      <div className="flex items-start justify-between gap-4 mb-3">
                        <div>
                          {/* category — from Firebase: project.category */}
                          <p className="text-xs font-mono text-white/30 tracking-widest uppercase mb-1">{project.category}</p>
                          {/* title — from Firebase: project.title */}
                          <h2 className="font-display text-2xl md:text-3xl leading-tight">{project.title}</h2>
                        </div>
                        {/* year — from Firebase: project.year */}
                        <span className="text-xs font-mono text-white/20 shrink-0 mt-1">{project.year}</span>
                      </div>
                      {/* short_description — from Firebase: project.short_description */}
                      <p className="text-sm text-white/50 font-body leading-relaxed mb-6 line-clamp-2">
                        {project.short_description}
                      </p>
                      {/* tags[] — from Firebase: project.tags */}
                      <div className="flex flex-wrap gap-2">
                        {(project.tags ?? []).map((tag: string) => (
                          <span key={tag} className="text-[10px] font-mono text-white/30 border border-white/8 px-2 py-0.5 rounded-sm">
                            {tag}
                          </span>
                        ))}
                      </div>
                    </div>
                  </Link>
                </motion.div>
              ))}
            </div>
          </AnimatePresence>
          
          {filtered.length === 0 && (
            <div className="py-32 text-center">
              <p className="text-white/20 font-mono text-sm">No projects found. Add projects in Firebase → projects collection</p>
            </div>
          )}
        </div>
      </section>
    </div>
  );
}
```

---

### FILE 13: app/work/[slug]/page.tsx — COMPLETE REWRITE

```tsx
'use client';

import { useParams, notFound } from 'next/navigation';
import Image from 'next/image';
import Link from 'next/link';
import { useProject } from '@/hooks/useProjects';
import { useEffect } from 'react';
import { incrementProjectViews } from '@/lib/firestore';

export default function ProjectPage() {
  const params = useParams();
  const slug = params?.slug as string;
  const { project, loading } = useProject(slug);

  // Increment view count in Firebase
  useEffect(() => {
    if (project?.id) incrementProjectViews(project.id);
  }, [project?.id]);

  if (loading) return <div className="min-h-screen pt-40 flex items-center justify-center"><div className="w-8 h-8 border border-white/20 border-t-white/60 rounded-full animate-spin" /></div>;
  if (!project) return notFound();

  return (
    <div className="min-h-screen pt-20">
      
      {/* FULL-BLEED HERO IMAGE */}
      {/* cover_image_url — from Firebase: project.cover_image_url */}
      <div className="relative h-[60vh] overflow-hidden">
        {project.cover_image_url ? (
          <Image src={project.cover_image_url} alt={project.title} fill className="object-cover" priority />
        ) : (
          <div className="w-full h-full bg-white/4" />
        )}
        <div className="absolute inset-0 bg-gradient-to-t from-[#080808] via-[#080808]/40 to-transparent" />
        {/* Breadcrumb */}
        <div className="absolute top-8 left-8">
          <Link href="/work" className="text-xs font-mono text-white/40 hover:text-white transition-colors">← BACK TO WORK</Link>
        </div>
      </div>

      {/* CONTENT */}
      <div className="max-w-7xl mx-auto px-6 md:px-12 lg:px-20 -mt-20 relative">
        <div className="grid lg:grid-cols-[2fr,1fr] gap-16">
          
          {/* Left: Main content */}
          <div>
            {/* category — from Firebase */}
            <p className="text-xs font-mono text-white/30 tracking-[0.2em] uppercase mb-3">{project.category}</p>
            {/* title — from Firebase: project.title */}
            <h1 className="font-display text-[clamp(40px,6vw,96px)] leading-[0.9] mb-6">{project.title}</h1>
            {/* subtitle — from Firebase: project.subtitle */}
            {project.subtitle && <p className="text-xl text-white/50 font-body mb-12">{project.subtitle}</p>}

            {/* tags[] — from Firebase */}
            <div className="flex flex-wrap gap-2 mb-16">
              {(project.tags ?? []).map((tag: string) => (
                <span key={tag} className="text-xs font-mono text-white/40 border border-white/10 px-3 py-1 rounded-sm">{tag}</span>
              ))}
            </div>

            {/* Challenge, Approach, Outcome — from Firebase */}
            {[
              { key: 'challenge', label: '01 — Challenge' },
              { key: 'approach',  label: '02 — Approach' },
              { key: 'outcome',   label: '03 — Outcome' },
            ].map(({ key, label }) => project[key] && (
              <div key={key} className="mb-12 pb-12 border-b border-white/6 last:border-0">
                <h2 className="font-display text-2xl mb-4">{label}</h2>
                <p className="text-white/60 font-body text-lg leading-relaxed">{project[key]}</p>
              </div>
            ))}

            {/* Gallery — from Firebase: project.gallery[] */}
            {project.gallery?.length > 0 && (
              <div className="grid gap-4 mt-12">
                {project.gallery.map((url: string, i: number) => (
                  <div key={i} className="relative aspect-video overflow-hidden rounded-sm">
                    <Image src={url} alt={`${project.title} — image ${i+1}`} fill className="object-cover" />
                  </div>
                ))}
              </div>
            )}
          </div>

          {/* Right: Sidebar — all from Firebase */}
          <div className="lg:sticky lg:top-28 h-fit">
            <div className="border border-white/6 p-8 space-y-6">
              {/* client — from Firebase: project.client */}
              {project.client && <InfoRow label="Client" value={project.client} />}
              {/* year — from Firebase */}
              {project.year  && <InfoRow label="Year"   value={String(project.year)} />}
              {/* role — from Firebase */}
              {project.role  && <InfoRow label="Role"   value={project.role} />}
              {/* duration — from Firebase */}
              {project.duration && <InfoRow label="Duration" value={project.duration} />}

              {/* live_url — from Firebase: project.live_url — REAL clickable link */}
              {project.live_url && (
                <a href={project.live_url} target="_blank" rel="noopener noreferrer"
                  className="block w-full text-center py-3 border border-white/20 text-sm font-mono tracking-widest hover:bg-white hover:text-black transition-all">
                  VISIT LIVE SITE ↗
                </a>
              )}
              {/* github_url — from Firebase: project.github_url — REAL clickable link */}
              {project.github_url && (
                <a href={project.github_url} target="_blank" rel="noopener noreferrer"
                  className="block w-full text-center py-3 border border-white/10 text-sm font-mono tracking-widest text-white/50 hover:text-white hover:border-white/30 transition-all">
                  VIEW CODE ↗
                </a>
              )}

              {/* tech_stack[] — from Firebase: project.tech_stack */}
              {project.tech_stack?.length > 0 && (
                <div>
                  <p className="text-xs font-mono text-white/30 tracking-widest uppercase mb-3">Tech Stack</p>
                  <div className="flex flex-wrap gap-2">
                    {project.tech_stack.map((t: string) => (
                      <span key={t} className="text-xs font-mono text-white/50 border border-white/8 px-2 py-1 rounded-sm">{t}</span>
                    ))}
                  </div>
                </div>
              )}
            </div>
          </div>
        </div>
      </div>
    </div>
  );
}

function InfoRow({ label, value }: { label: string; value: string }) {
  return (
    <div>
      <p className="text-xs font-mono text-white/20 tracking-widest uppercase mb-1">{label}</p>
      <p className="text-sm font-mono text-white/70">{value}</p>
    </div>
  );
}
```

---

### FILE 14: app/contact/page.tsx — COMPLETELY REBUILT CONTACT PAGE

**This is the most important fix. The contact form ACTUALLY SAVES to Firestore.**
**Everything comes from Firebase. Email from Firebase. Project types from Firebase.**

```tsx
'use client';

import { useState, useEffect, useRef } from 'react';
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';
import toast from 'react-hot-toast';
import { useConfig } from '@/hooks/useConfig';
import { useSocialLinks } from '@/hooks/useSocialLinks';
import { submitContactForm } from '@/lib/firestore';
import { motion, AnimatePresence } from 'framer-motion';
import gsap from 'gsap';

const schema = z.object({
  name:         z.string().min(2, 'Name must be at least 2 characters'),
  email:        z.string().email('Enter a valid email address'),
  project_type: z.string().min(1, 'Please select a project type'),
  budget:       z.string().optional(),
  message:      z.string().min(20, 'Tell me a bit more — at least 20 characters'),
  timeline:     z.string().optional(),
});
type FormData = z.infer<typeof schema>;

export default function ContactPage() {
  const { config } = useConfig();
  const { socialLinks } = useSocialLinks();
  const [submitted, setSubmitted] = useState(false);
  const [activeMethod, setActiveMethod] = useState<'form' | 'autofill'>('form');
  
  // Autofill compose state
  const [autofillEmail,   setAutofillEmail]   = useState('');
  const [autofillType,    setAutofillType]     = useState('');
  const [autofillBrief,   setAutofillBrief]    = useState('');
  const [autofillMethod,  setAutofillMethod]   = useState<'gmail' | 'mailto'>('gmail');

  const {
    register, handleSubmit, watch, setValue,
    formState: { errors, isSubmitting },
    reset,
  } = useForm<FormData>({ resolver: zodResolver(schema) });

  const selectedType = watch('project_type', '');

  const onSubmit = async (data: FormData) => {
    try {
      // REAL submission to Firebase
      await submitContactForm(data);
      setSubmitted(true);
      reset();
      toast.success('Message sent! I\'ll reply within 24 hours.');
    } catch (e: any) {
      toast.error('Something went wrong. Try emailing me directly.');
      console.error(e);
    }
  };

  // Build autofill email
  const buildEmailBody = () => {
    const to      = config.owner_email;
    const subject = `Project Inquiry — ${autofillType || '[Type]'} · coldpixels.co`;
    const body = [
      `Hi ${config.owner_name},`,
      '',
      `I found your portfolio at coldpixels.co and I'd like to discuss a project.`,
      '',
      '━━ Project Type ━━',
      autofillType || '[Select a type above]',
      '',
      '━━ Project Brief ━━',
      autofillBrief || '[Describe your project above]',
      '',
      '━━ My Reply Email ━━',
      autofillEmail || '[Enter your email above]',
      '',
      'Looking forward to connecting.',
    ].join('\n');
    return { to, subject, body };
  };

  const handleAutofillLaunch = () => {
    if (!autofillEmail || !autofillEmail.includes('@')) {
      toast.error('Enter your reply email first');
      return;
    }
    const { to, subject, body } = buildEmailBody();
    if (autofillMethod === 'gmail') {
      window.open(
        `https://mail.google.com/mail/u/0/?view=cm&fs=1&to=${encodeURIComponent(to)}&su=${encodeURIComponent(subject)}&body=${encodeURIComponent(body)}`,
        '_blank', 'noopener,noreferrer'
      );
    } else {
      const a = document.createElement('a');
      a.href = `mailto:${to}?subject=${encodeURIComponent(subject)}&body=${encodeURIComponent(body)}&reply-to=${encodeURIComponent(autofillEmail)}`;
      a.click();
    }
  };

  const { to, subject, body } = buildEmailBody();

  return (
    <div className="min-h-screen pt-20">
      
      {/* HERO */}
      <section className="pt-24 pb-16 px-6 md:px-12 lg:px-20 border-b border-white/6 overflow-hidden">
        <div className="max-w-7xl mx-auto relative">
          <span className="absolute right-0 top-0 font-display text-[18vw] text-white/[0.025] leading-none select-none pointer-events-none">SIGNAL</span>
          <p className="text-xs font-mono text-white/30 tracking-[0.2em] uppercase mb-4">Contact</p>
          <h1 className="font-display text-[clamp(52px,9vw,128px)] leading-[0.9] tracking-tight mb-6">
            LET'S BUILD<br />SOMETHING<br />REMARKABLE
          </h1>
          <div className="flex flex-wrap items-center gap-6 mt-8">
            {/* response_time — from Firebase: config.response_time */}
            <p className="text-sm text-white/40 font-mono">{config.response_time}</p>
            <span className="h-px w-12 bg-white/10 hidden sm:block" />
            {/* owner_email — from Firebase: config.owner_email */}
            <a href={`mailto:${config.owner_email}`} className="text-sm text-white/60 font-mono hover:text-white transition-colors">
              {config.owner_email}
            </a>
          </div>
        </div>
      </section>

      {/* METHOD TOGGLE */}
      <div className="border-b border-white/6">
        <div className="max-w-7xl mx-auto px-6 md:px-12 lg:px-20 flex">
          <button
            onClick={() => setActiveMethod('form')}
            className={`py-4 px-6 text-xs font-mono tracking-widest uppercase border-b-2 transition-colors ${
              activeMethod === 'form' ? 'border-white text-white' : 'border-transparent text-white/30 hover:text-white/60'
            }`}
          >
            Contact Form
          </button>
          <button
            onClick={() => setActiveMethod('autofill')}
            className={`py-4 px-6 text-xs font-mono tracking-widest uppercase border-b-2 transition-colors ${
              activeMethod === 'autofill' ? 'border-white text-white' : 'border-transparent text-white/30 hover:text-white/60'
            }`}
          >
            Autofill Compose
          </button>
        </div>
      </div>

      {/* MAIN CONTENT */}
      <div className="max-w-7xl mx-auto px-6 md:px-12 lg:px-20 py-20">
        <div className="grid lg:grid-cols-[3fr,2fr] gap-16">
          
          {/* LEFT: Form or Autofill */}
          <div>
            <AnimatePresence mode="wait">
              {activeMethod === 'form' ? (
                <motion.div key="form" initial={{ opacity: 0, x: -20 }} animate={{ opacity: 1, x: 0 }} exit={{ opacity: 0, x: 20 }} transition={{ duration: 0.3 }}>
                  
                  {submitted ? (
                    <div className="py-24 text-center border border-white/6">
                      <div className="font-display text-6xl mb-4">✓</div>
                      <h2 className="font-display text-3xl mb-3">Message Received</h2>
                      <p className="text-white/50 font-body mb-8">{config.response_time}</p>
                      <button onClick={() => setSubmitted(false)} className="text-xs font-mono text-white/30 hover:text-white underline underline-offset-4">
                        Send another message
                      </button>
                    </div>
                  ) : (
                    <form onSubmit={handleSubmit(onSubmit)} className="space-y-8" noValidate>
                      <h2 className="font-display text-3xl mb-8">Tell me about your project</h2>

                      {/* NAME */}
                      <div>
                        <label className="block text-xs font-mono text-white/30 tracking-widest uppercase mb-3">Your Name *</label>
                        <input
                          {...register('name')}
                          type="text"
                          placeholder="John Smith"
                          className="w-full bg-transparent border-b border-white/10 focus:border-white/50 text-white placeholder:text-white/20 py-3 font-mono text-sm outline-none transition-colors"
                        />
                        {errors.name && <p className="text-xs text-red-400/80 mt-2 font-mono">{errors.name.message}</p>}
                      </div>

                      {/* EMAIL */}
                      <div>
                        <label className="block text-xs font-mono text-white/30 tracking-widest uppercase mb-3">Your Email *</label>
                        <input
                          {...register('email')}
                          type="email"
                          placeholder="john@company.com"
                          className="w-full bg-transparent border-b border-white/10 focus:border-white/50 text-white placeholder:text-white/20 py-3 font-mono text-sm outline-none transition-colors"
                        />
                        {errors.email && <p className="text-xs text-red-400/80 mt-2 font-mono">{errors.email.message}</p>}
                      </div>

                      {/* PROJECT TYPE — from Firebase: config.contact_form_project_types[] */}
                      <div>
                        <label className="block text-xs font-mono text-white/30 tracking-widest uppercase mb-3">Project Type *</label>
                        <div className="flex flex-wrap gap-2">
                          {(config.contact_form_project_types ?? []).map((type: string) => (
                            <button
                              key={type}
                              type="button"
                              onClick={() => setValue('project_type', type, { shouldValidate: true })}
                              className={`px-4 py-2 text-xs font-mono tracking-widest border transition-all rounded-sm ${
                                selectedType === type
                                  ? 'bg-white text-black border-white'
                                  : 'border-white/10 text-white/40 hover:border-white/30 hover:text-white/70'
                              }`}
                            >
                              {type}
                            </button>
                          ))}
                        </div>
                        {errors.project_type && <p className="text-xs text-red-400/80 mt-2 font-mono">{errors.project_type.message}</p>}
                      </div>

                      {/* BUDGET — from Firebase: config.contact_form_budget_ranges[] */}
                      <div>
                        <label className="block text-xs font-mono text-white/30 tracking-widest uppercase mb-3">Estimated Budget</label>
                        <div className="flex flex-wrap gap-2">
                          {(config.contact_form_budget_ranges ?? []).map((range: string) => {
                            const w = watch('budget', '');
                            return (
                              <button
                                key={range}
                                type="button"
                                onClick={() => setValue('budget', w === range ? '' : range)}
                                className={`px-4 py-2 text-xs font-mono tracking-widest border transition-all rounded-sm ${
                                  w === range
                                    ? 'bg-white text-black border-white'
                                    : 'border-white/10 text-white/40 hover:border-white/30 hover:text-white/70'
                                }`}
                              >
                                {range}
                              </button>
                            );
                          })}
                        </div>
                      </div>

                      {/* TIMELINE */}
                      <div>
                        <label className="block text-xs font-mono text-white/30 tracking-widest uppercase mb-3">Timeline</label>
                        <select
                          {...register('timeline')}
                          className="w-full bg-[#0d0d0d] border border-white/10 focus:border-white/30 text-white py-3 px-4 font-mono text-sm outline-none transition-colors"
                        >
                          <option value="">Select a timeline</option>
                          <option value="ASAP">As soon as possible</option>
                          <option value="1-2 months">1–2 months</option>
                          <option value="3-4 months">3–4 months</option>
                          <option value="6+ months">6+ months</option>
                          <option value="flexible">Flexible</option>
                        </select>
                      </div>

                      {/* MESSAGE */}
                      <div>
                        <label className="block text-xs font-mono text-white/30 tracking-widest uppercase mb-3">Project Brief *</label>
                        <textarea
                          {...register('message')}
                          rows={6}
                          placeholder="Tell me about your project — what you're building, your goals, and any specific requirements..."
                          className="w-full bg-transparent border border-white/10 focus:border-white/30 text-white placeholder:text-white/20 p-4 font-body text-sm outline-none transition-colors resize-none"
                        />
                        {errors.message && <p className="text-xs text-red-400/80 mt-2 font-mono">{errors.message.message}</p>}
                      </div>

                      <button
                        type="submit"
                        disabled={isSubmitting}
                        className="w-full py-4 bg-white text-black font-display tracking-[0.1em] text-lg hover:bg-white/90 active:scale-95 transition-all disabled:opacity-50 disabled:cursor-not-allowed"
                      >
                        {isSubmitting ? 'SENDING...' : 'SEND MESSAGE →'}
                      </button>
                      
                      <p className="text-xs font-mono text-white/20 text-center">
                        Your message is saved to our secure system. {config.response_time}.
                      </p>
                    </form>
                  )}
                </motion.div>
              ) : (
                /* ── AUTOFILL COMPOSE ── */
                <motion.div key="autofill" initial={{ opacity: 0, x: 20 }} animate={{ opacity: 1, x: 0 }} exit={{ opacity: 0, x: -20 }} transition={{ duration: 0.3 }}>
                  <h2 className="font-display text-3xl mb-2">Autofill Compose</h2>
                  {/* Shows owner_email from Firebase live */}
                  <p className="text-xs font-mono text-white/30 mb-8">
                    TO: <span className="text-white/60">{config.owner_email}</span> · Live from Firebase
                  </p>

                  <div className="space-y-6">
                    {/* Your email */}
                    <div>
                      <label className="block text-xs font-mono text-white/30 tracking-widest uppercase mb-2">01 — Your Reply Email</label>
                      <input
                        type="email"
                        value={autofillEmail}
                        onChange={(e) => setAutofillEmail(e.target.value)}
                        placeholder="your@email.com"
                        className="w-full bg-transparent border-b border-white/10 focus:border-white/50 text-white placeholder:text-white/20 py-3 font-mono text-sm outline-none transition-colors"
                      />
                    </div>

                    {/* Project type — from Firebase config.contact_form_project_types */}
                    <div>
                      <label className="block text-xs font-mono text-white/30 tracking-widest uppercase mb-2">02 — Project Type</label>
                      <div className="flex flex-wrap gap-2">
                        {(config.contact_form_project_types ?? []).map((type: string) => (
                          <button
                            key={type}
                            type="button"
                            onClick={() => setAutofillType(autofillType === type ? '' : type)}
                            className={`px-4 py-2 text-xs font-mono tracking-widest border transition-all rounded-sm ${
                              autofillType === type
                                ? 'bg-white text-black border-white'
                                : 'border-white/10 text-white/40 hover:border-white/30'
                            }`}
                          >
                            {type}
                          </button>
                        ))}
                      </div>
                    </div>

                    {/* Brief */}
                    <div>
                      <div className="flex justify-between mb-2">
                        <label className="text-xs font-mono text-white/30 tracking-widest uppercase">03 — Project Brief</label>
                        <span className="text-xs font-mono text-white/20">{autofillBrief.length}/600</span>
                      </div>
                      <textarea
                        value={autofillBrief}
                        onChange={(e) => setAutofillBrief(e.target.value.slice(0, 600))}
                        rows={5}
                        placeholder="Describe your project briefly..."
                        className="w-full bg-transparent border border-white/10 focus:border-white/30 text-white placeholder:text-white/20 p-4 font-body text-sm outline-none transition-colors resize-none"
                      />
                    </div>

                    {/* Method toggle */}
                    <div className="flex gap-2">
                      <button type="button" onClick={() => setAutofillMethod('gmail')}
                        className={`flex-1 py-2 text-xs font-mono tracking-widest border transition-all ${autofillMethod === 'gmail' ? 'bg-white text-black border-white' : 'border-white/10 text-white/40'}`}>
                        ↗ Gmail
                      </button>
                      <button type="button" onClick={() => setAutofillMethod('mailto')}
                        className={`flex-1 py-2 text-xs font-mono tracking-widest border transition-all ${autofillMethod === 'mailto' ? 'bg-white text-black border-white' : 'border-white/10 text-white/40'}`}>
                        ✉ Mail App
                      </button>
                    </div>

                    <button
                      type="button"
                      onClick={handleAutofillLaunch}
                      className="w-full py-4 bg-white text-black font-display tracking-[0.1em] text-lg hover:bg-white/90 active:scale-95 transition-all"
                    >
                      COMPOSE & OPEN ↗
                    </button>

                    {/* Live preview */}
                    <div className="border border-white/6 p-6 space-y-3">
                      <p className="text-xs font-mono text-white/20 uppercase tracking-widest mb-4">Email Preview</p>
                      <div className="flex gap-3">
                        <span className="text-xs font-mono text-white/20 w-14 shrink-0">TO:</span>
                        <span className="text-xs font-mono text-white/50 break-all">{config.owner_email}</span>
                      </div>
                      <div className="flex gap-3">
                        <span className="text-xs font-mono text-white/20 w-14 shrink-0">FROM:</span>
                        <span className="text-xs font-mono text-white/50 break-all">{autofillEmail || '...'}</span>
                      </div>
                      <div className="flex gap-3">
                        <span className="text-xs font-mono text-white/20 w-14 shrink-0">RE:</span>
                        <span className="text-xs font-mono text-white/50">Project Inquiry — {autofillType || '...'}</span>
                      </div>
                      <div className="h-px bg-white/6 my-3" />
                      <p className="text-xs font-mono text-white/30 line-clamp-4 whitespace-pre-line">{body.slice(0, 300)}{body.length > 300 ? '...' : ''}</p>
                    </div>
                  </div>
                </motion.div>
              )}
            </AnimatePresence>
          </div>

          {/* RIGHT: Info panel */}
          <div className="space-y-12">
            {/* Direct contact — from Firebase */}
            <div>
              <h3 className="text-xs font-mono text-white/30 tracking-widest uppercase mb-6">Direct Contact</h3>
              <div className="space-y-4">
                <div>
                  <p className="text-xs font-mono text-white/20 mb-1">EMAIL</p>
                  {/* owner_email — from Firebase: config.owner_email */}
                  <a href={`mailto:${config.owner_email}`} className="font-mono text-sm text-white/70 hover:text-white transition-colors">
                    {config.owner_email}
                  </a>
                </div>
                {config.owner_phone && (
                  <div>
                    <p className="text-xs font-mono text-white/20 mb-1">PHONE</p>
                    {/* owner_phone — from Firebase: config.owner_phone */}
                    <a href={`tel:${config.owner_phone}`} className="font-mono text-sm text-white/70 hover:text-white transition-colors">
                      {config.owner_phone}
                    </a>
                  </div>
                )}
                <div>
                  <p className="text-xs font-mono text-white/20 mb-1">LOCATION</p>
                  {/* owner_location — from Firebase: config.owner_location */}
                  <p className="font-mono text-sm text-white/70">{config.owner_location}</p>
                </div>
                <div>
                  <p className="text-xs font-mono text-white/20 mb-1">RESPONSE TIME</p>
                  {/* response_time — from Firebase: config.response_time */}
                  <p className="font-mono text-sm text-white/70">{config.response_time}</p>
                </div>
              </div>
            </div>

            {/* Social links — from Firebase: social_links collection */}
            <div>
              <h3 className="text-xs font-mono text-white/30 tracking-widest uppercase mb-6">Find Me Online</h3>
              <div className="space-y-3">
                {socialLinks.map((link: any) => (
                  /* REAL URL from Firebase — opens in new tab */
                  <a
                    key={link.id}
                    href={link.url}
                    target="_blank"
                    rel="noopener noreferrer"
                    className="flex items-center justify-between group py-3 border-b border-white/6 hover:border-white/20 transition-colors"
                  >
                    <span className="text-sm font-mono text-white/50 group-hover:text-white transition-colors">
                      {link.platform}
                    </span>
                    <span className="text-white/20 group-hover:text-white/60 transition-colors text-sm">↗</span>
                  </a>
                ))}
              </div>
            </div>

            {/* Calendly — from Firebase: config.calendly_url */}
            {config.calendly_url && (
              <a
                href={config.calendly_url}
                target="_blank"
                rel="noopener noreferrer"
                className="block text-center py-4 border border-white/10 text-sm font-mono tracking-widest hover:bg-white hover:text-black transition-all"
              >
                BOOK A CALL ↗
              </a>
            )}
          </div>
        </div>
      </div>
    </div>
  );
}
```

---

### FILE 15: app/services/page.tsx — COMPLETE REWRITE (Firebase-connected)

```tsx
'use client';

import { useState } from 'react';
import { useServices } from '@/hooks/useServices';
import { motion, AnimatePresence } from 'framer-motion';

export default function ServicesPage() {
  const { services } = useServices();
  const [active, setActive] = useState<string | null>(null);

  return (
    <div className="min-h-screen pt-20">
      
      {/* HERO */}
      <section className="pt-24 pb-20 px-6 md:px-12 lg:px-20 border-b border-white/6">
        <div className="max-w-7xl mx-auto">
          <p className="text-xs font-mono text-white/30 tracking-[0.2em] uppercase mb-4">What I Offer</p>
          <h1 className="font-display text-[clamp(52px,9vw,128px)] leading-[0.9]">Services</h1>
        </div>
      </section>

      {/* SERVICES LIST — all from Firebase services collection */}
      <section className="px-6 md:px-12 lg:px-20">
        <div className="max-w-7xl mx-auto">
          {services.map((service: any, i: number) => (
            <div key={service.id} className="border-b border-white/6">
              <button
                className="w-full py-8 flex items-start justify-between gap-8 text-left group hover:bg-white/1 transition-colors px-2"
                onClick={() => setActive(active === service.id ? null : service.id)}
              >
                <div className="flex items-center gap-6">
                  <span className="text-xs font-mono text-white/20 w-8">{String(i+1).padStart(2,'0')}</span>
                  <div>
                    {/* name — from Firebase: service.name */}
                    <h2 className="font-display text-2xl md:text-4xl group-hover:translate-x-2 transition-transform">{service.name}</h2>
                    {/* tagline — from Firebase: service.tagline */}
                    {service.tagline && <p className="text-sm text-white/40 font-mono mt-1">{service.tagline}</p>}
                  </div>
                </div>
                <div className="flex items-center gap-4 shrink-0 mt-2">
                  {/* popular — from Firebase: service.popular */}
                  {service.popular && (
                    <span className="text-[10px] font-mono tracking-widest border border-white/20 px-2 py-1 text-white/60">POPULAR</span>
                  )}
                  {/* starting_price — from Firebase: service.starting_price */}
                  {service.starting_price && (
                    <span className="text-sm font-mono text-white/30">from {service.starting_price}</span>
                  )}
                  <motion.span
                    animate={{ rotate: active === service.id ? 45 : 0 }}
                    transition={{ duration: 0.2 }}
                    className="text-white/30 text-2xl"
                  >+</motion.span>
                </div>
              </button>
              
              <AnimatePresence>
                {active === service.id && (
                  <motion.div
                    initial={{ height: 0, opacity: 0 }}
                    animate={{ height: 'auto', opacity: 1 }}
                    exit={{ height: 0, opacity: 0 }}
                    transition={{ duration: 0.4, ease: [0.33, 1, 0.68, 1] }}
                    className="overflow-hidden"
                  >
                    <div className="pb-12 px-14 grid md:grid-cols-3 gap-8">
                      {/* description — from Firebase: service.description */}
                      <div className="md:col-span-3 mb-4">
                        <p className="text-white/60 font-body text-lg leading-relaxed max-w-2xl">{service.description}</p>
                      </div>
                      {/* features[] — from Firebase: service.features */}
                      <div>
                        <h3 className="text-xs font-mono text-white/30 tracking-widest uppercase mb-4">What's Included</h3>
                        <ul className="space-y-2">
                          {(service.features ?? []).map((f: string, j: number) => (
                            <li key={j} className="flex gap-3 text-sm text-white/60 font-body">
                              <span className="text-white/20 shrink-0 mt-0.5">✓</span>{f}
                            </li>
                          ))}
                        </ul>
                      </div>
                      {/* deliverables[] — from Firebase: service.deliverables */}
                      <div>
                        <h3 className="text-xs font-mono text-white/30 tracking-widest uppercase mb-4">Deliverables</h3>
                        <ul className="space-y-2">
                          {(service.deliverables ?? []).map((d: string, j: number) => (
                            <li key={j} className="text-sm text-white/60 font-body">→ {d}</li>
                          ))}
                        </ul>
                      </div>
                      <div>
                        <h3 className="text-xs font-mono text-white/30 tracking-widest uppercase mb-4">Timeline & Price</h3>
                        {/* timeline — from Firebase: service.timeline */}
                        {service.timeline && <p className="text-sm text-white/60 font-mono mb-2">⏱ {service.timeline}</p>}
                        {/* starting_price — from Firebase */}
                        {service.starting_price && <p className="text-sm text-white/60 font-mono">💰 from {service.starting_price}</p>}
                        <a href="/contact" className="inline-block mt-6 px-6 py-3 border border-white/20 text-xs font-mono tracking-widest hover:bg-white hover:text-black transition-all">
                          GET A QUOTE →
                        </a>
                      </div>
                    </div>
                  </motion.div>
                )}
              </AnimatePresence>
            </div>
          ))}
          {services.length === 0 && (
            <div className="py-32 text-center border border-white/6">
              <p className="text-white/20 font-mono text-sm">Add services in Firebase → services collection</p>
            </div>
          )}
        </div>
      </section>

      {/* CTA */}
      <section className="py-32 px-6 md:px-12 lg:px-20 text-center">
        <div className="max-w-2xl mx-auto">
          <h2 className="font-display text-[clamp(40px,6vw,80px)] mb-6">Ready to start?</h2>
          <p className="text-white/50 font-body mb-8">Let's discuss your project and find the right service combination.</p>
          <a href="/contact" className="inline-block px-12 py-4 bg-white text-black font-display text-xl tracking-widest hover:bg-white/90 transition-colors">
            GET IN TOUCH →
          </a>
        </div>
      </section>
    </div>
  );
}
```

---

### FILE 16: app/faq/page.tsx — COMPLETE REWRITE (Firebase-connected)

```tsx
'use client';

import { useState, useMemo } from 'react';
import { motion, AnimatePresence } from 'framer-motion';
import { useFAQs } from '@/hooks/useFAQs';
import { voteFAQ } from '@/lib/firestore';
import toast from 'react-hot-toast';

export default function FAQPage() {
  const { faqs } = useFAQs();
  const [search, setSearch] = useState('');
  const [openId, setOpenId] = useState<string | null>(null);
  const [voted, setVoted] = useState<Set<string>>(new Set());

  const categories = ['All', ...Array.from(new Set(faqs.map((f: any) => f.category).filter(Boolean)))];
  const [activeCategory, setActiveCategory] = useState('All');

  const filtered = useMemo(() => {
    return faqs.filter((f: any) => {
      const matchesSearch = search === '' || 
        f.question?.toLowerCase().includes(search.toLowerCase()) ||
        f.answer?.toLowerCase().includes(search.toLowerCase());
      const matchesCat = activeCategory === 'All' || f.category === activeCategory;
      return matchesSearch && matchesCat;
    });
  }, [faqs, search, activeCategory]);

  const handleVote = async (id: string, helpful: boolean) => {
    if (voted.has(id)) { toast('You already voted on this one!'); return; }
    await voteFAQ(id, helpful);
    setVoted(prev => new Set([...prev, id]));
    toast.success(helpful ? 'Thanks for the feedback!' : 'Thanks, I\'ll improve this answer.');
  };

  return (
    <div className="min-h-screen pt-20">
      
      {/* HERO */}
      <section className="pt-24 pb-16 px-6 md:px-12 lg:px-20 border-b border-white/6 relative overflow-hidden">
        <span className="absolute right-0 bottom-0 font-display text-[20vw] text-white/[0.025] leading-none select-none pointer-events-none">FAQ</span>
        <div className="max-w-7xl mx-auto relative">
          <p className="text-xs font-mono text-white/30 tracking-[0.2em] uppercase mb-4">Knowledge Base</p>
          <h1 className="font-display text-[clamp(52px,9vw,100px)] leading-[0.9]">Frequently<br />Asked Questions</h1>
        </div>
      </section>

      {/* SEARCH */}
      <div className="sticky top-20 z-30 border-b border-white/6 bg-[#080808]/95 backdrop-blur-xl">
        <div className="max-w-7xl mx-auto px-6 md:px-12 lg:px-20 py-4 flex gap-4 items-center">
          <input
            type="text"
            value={search}
            onChange={(e) => setSearch(e.target.value)}
            placeholder="Search questions..."
            className="flex-1 bg-transparent text-sm font-mono text-white placeholder:text-white/20 outline-none py-2"
          />
          {search && (
            <button onClick={() => setSearch('')} className="text-xs font-mono text-white/20 hover:text-white transition-colors">CLEAR</button>
          )}
        </div>
      </div>

      {/* CATEGORIES */}
      <div className="border-b border-white/6">
        <div className="max-w-7xl mx-auto px-6 md:px-12 lg:px-20 py-4 flex gap-2 overflow-x-auto">
          {categories.map((cat) => (
            <button key={cat} onClick={() => setActiveCategory(cat)}
              className={`whitespace-nowrap px-4 py-2 text-xs font-mono tracking-widest uppercase transition-all rounded-sm ${
                activeCategory === cat ? 'bg-white text-black' : 'text-white/40 hover:text-white border border-white/10 hover:border-white/20'
              }`}>
              {cat}
            </button>
          ))}
        </div>
      </div>

      {/* FAQ ITEMS — all from Firebase faqs collection */}
      <section className="py-16 px-6 md:px-12 lg:px-20">
        <div className="max-w-4xl mx-auto">
          {filtered.length === 0 ? (
            <div className="py-24 text-center">
              <p className="text-white/20 font-mono text-sm">
                {faqs.length === 0 ? 'Add FAQs in Firebase → faqs collection' : 'No FAQs match your search.'}
              </p>
            </div>
          ) : (
            <div className="divide-y divide-white/6">
              {filtered.map((faq: any) => (
                <div key={faq.id}>
                  <button
                    className="w-full py-6 flex items-start justify-between gap-6 text-left group"
                    onClick={() => setOpenId(openId === faq.id ? null : faq.id)}
                  >
                    {/* question — from Firebase: faq.question */}
                    <span className="font-mono text-base text-white/80 group-hover:text-white transition-colors">{faq.question}</span>
                    <motion.span
                      animate={{ rotate: openId === faq.id ? 45 : 0 }}
                      transition={{ duration: 0.2 }}
                      className="text-white/30 text-xl shrink-0 mt-0.5"
                    >+</motion.span>
                  </button>
                  <AnimatePresence>
                    {openId === faq.id && (
                      <motion.div
                        initial={{ height: 0, opacity: 0 }}
                        animate={{ height: 'auto', opacity: 1 }}
                        exit={{ height: 0, opacity: 0 }}
                        transition={{ duration: 0.35, ease: [0.33, 1, 0.68, 1] }}
                        className="overflow-hidden"
                      >
                        <div className="pb-6">
                          {/* answer — from Firebase: faq.answer */}
                          <p className="text-white/60 font-body leading-relaxed mb-6">{faq.answer}</p>
                          {/* category — from Firebase: faq.category */}
                          {faq.category && (
                            <span className="text-[10px] font-mono text-white/20 border border-white/8 px-2 py-0.5 rounded-sm mr-4">{faq.category}</span>
                          )}
                          {/* Helpful voting — saves to Firebase */}
                          {!voted.has(faq.id) ? (
                            <span className="text-xs font-mono text-white/20">
                              Helpful?{' '}
                              <button onClick={() => handleVote(faq.id, true)} className="hover:text-white transition-colors mx-1">Yes ({faq.helpful_count ?? 0})</button>
                              /
                              <button onClick={() => handleVote(faq.id, false)} className="hover:text-white transition-colors mx-1">No ({faq.not_helpful_count ?? 0})</button>
                            </span>
                          ) : (
                            <span className="text-xs font-mono text-white/20">Thanks for voting!</span>
                          )}
                        </div>
                      </motion.div>
                    )}
                  </AnimatePresence>
                </div>
              ))}
            </div>
          )}
        </div>
      </section>

      {/* CTA */}
      <section className="py-24 px-6 md:px-12 lg:px-20 border-t border-white/6 text-center">
        <div className="max-w-xl mx-auto">
          <h2 className="font-display text-3xl mb-4">Still have questions?</h2>
          <p className="text-white/50 font-body mb-8">Can't find what you're looking for? Reach out directly.</p>
          <a href="/contact" className="inline-block px-10 py-4 border border-white/20 font-display text-lg tracking-widest hover:bg-white hover:text-black transition-all">
            ASK ME DIRECTLY →
          </a>
        </div>
      </section>
    </div>
  );
}
```

---

### FILE 17: src/components/layout/Footer.tsx — COMPLETE REWRITE (Firebase-connected)

```tsx
'use client';
import Link from 'next/link';
import { useConfig } from '@/hooks/useConfig';
import { useSocialLinks } from '@/hooks/useSocialLinks';

export function Footer() {
  const { config } = useConfig();
  const { socialLinks } = useSocialLinks();

  return (
    <footer className="border-t border-white/6 mt-32">
      
      {/* CTA Belt — footer_tagline from Firebase: config.footer_tagline */}
      <div className="border-b border-white/6 py-20 px-6 md:px-12 lg:px-20">
        <div className="max-w-7xl mx-auto">
          <Link href="/contact" className="group block">
            <h2 className="font-display text-[clamp(40px,8vw,120px)] leading-[0.9] group-hover:translate-x-2 transition-transform">
              {config.footer_tagline}
            </h2>
          </Link>
        </div>
      </div>

      {/* 4-col grid */}
      <div className="py-16 px-6 md:px-12 lg:px-20">
        <div className="max-w-7xl mx-auto grid sm:grid-cols-2 lg:grid-cols-4 gap-12">
          
          {/* Brand */}
          <div>
            <Link href="/" className="font-display text-2xl mb-4 block">coldpixels.co</Link>
            {/* owner_bio_short — from Firebase: config.owner_bio_short */}
            <p className="text-sm text-white/40 font-body leading-relaxed mb-6">{config.owner_bio_short}</p>
            {/* Availability */}
            <div className="flex items-center gap-2">
              <span className="relative flex h-2 w-2">
                <span className="animate-ping absolute inline-flex h-full w-full rounded-full bg-white opacity-30" />
                <span className="relative inline-flex rounded-full h-2 w-2 bg-white" />
              </span>
              {/* availability_label — from Firebase */}
              <span className="text-xs font-mono text-white/30">{config.availability_label}</span>
            </div>
          </div>

          {/* Navigate */}
          <div>
            <h3 className="text-xs font-mono text-white/20 tracking-widest uppercase mb-6">Navigate</h3>
            <nav className="space-y-3">
              {[['/', 'Home'], ['/work', 'Work'], ['/about', 'About'], ['/services', 'Services'], ['/contact', 'Contact'], ['/faq', 'FAQ']].map(([href, label]) => (
                <Link key={href} href={href} className="block text-sm text-white/50 hover:text-white font-mono transition-colors">{label}</Link>
              ))}
            </nav>
          </div>

          {/* Social — from Firebase: social_links collection — REAL URLs */}
          <div>
            <h3 className="text-xs font-mono text-white/20 tracking-widest uppercase mb-6">Social</h3>
            <nav className="space-y-3">
              {socialLinks.map((link: any) => (
                /* REAL clickable URL from Firebase */
                <a
                  key={link.id}
                  href={link.url}
                  target="_blank"
                  rel="noopener noreferrer"
                  className="block text-sm text-white/50 hover:text-white font-mono transition-colors"
                >
                  {link.platform} ↗
                </a>
              ))}
            </nav>
          </div>

          {/* Contact — all from Firebase */}
          <div>
            <h3 className="text-xs font-mono text-white/20 tracking-widest uppercase mb-6">Contact</h3>
            <div className="space-y-3">
              {/* owner_email — from Firebase */}
              <a href={`mailto:${config.owner_email}`} className="block text-sm text-white/50 hover:text-white font-mono transition-colors break-all">
                {config.owner_email}
              </a>
              {/* owner_location — from Firebase */}
              <p className="text-sm text-white/30 font-mono">{config.owner_location}</p>
              {/* calendly_url — from Firebase: clickable if set */}
              {config.calendly_url && (
                <a href={config.calendly_url} target="_blank" rel="noopener noreferrer"
                  className="inline-block text-xs font-mono text-white/30 border border-white/10 px-3 py-2 hover:border-white/30 hover:text-white transition-colors">
                  Book a call ↗
                </a>
              )}
            </div>
          </div>
        </div>
      </div>

      {/* Bottom bar */}
      <div className="border-t border-white/4 px-6 md:px-12 lg:px-20 py-6">
        <div className="max-w-7xl mx-auto flex flex-col sm:flex-row justify-between gap-4">
          {/* footer_copy + copyright_year — from Firebase */}
          <p className="text-xs font-mono text-white/20">
            © {config.copyright_year} {config.footer_copy}
          </p>
          <p className="text-xs font-mono text-white/10">BUILT WITH NEXT.JS + FIREBASE</p>
        </div>
      </div>
    </footer>
  );
}
```

---

### FILE 18: src/components/layout/Navbar.tsx — COMPLETE REWRITE (Firebase-connected)

```tsx
'use client';

import { useState, useEffect } from 'react';
import Link from 'next/link';
import { usePathname } from 'next/navigation';
import { useConfig } from '@/hooks/useConfig';
import { useSocialLinks } from '@/hooks/useSocialLinks';
import { motion, AnimatePresence } from 'framer-motion';

const NAV_LINKS = [
  { href: '/', label: 'Home' },
  { href: '/work', label: 'Work' },
  { href: '/about', label: 'About' },
  { href: '/services', label: 'Services' },
  { href: '/contact', label: 'Contact' },
  { href: '/faq', label: 'FAQ' },
];

export function Navbar() {
  const pathname = usePathname();
  const { config } = useConfig();
  const { socialLinks } = useSocialLinks();
  const [scrolled, setScrolled] = useState(false);
  const [menuOpen, setMenuOpen] = useState(false);

  useEffect(() => {
    const onScroll = () => setScrolled(window.scrollY > 60);
    window.addEventListener('scroll', onScroll, { passive: true });
    return () => window.removeEventListener('scroll', onScroll);
  }, []);

  // Close menu on navigation
  useEffect(() => { setMenuOpen(false); }, [pathname]);

  return (
    <>
      <nav className={`fixed top-0 left-0 right-0 z-50 h-20 flex items-center justify-between transition-all duration-500 ${
        scrolled ? 'bg-[#080808]/90 backdrop-blur-xl border-b border-white/6 px-6 md:px-12 lg:px-20' : 'bg-transparent px-6 md:px-12 lg:px-20'
      }`}>
        {/* Logo — owner_name from Firebase */}
        <Link href="/" className="font-display text-2xl tracking-tight hover:opacity-70 transition-opacity">
          coldpixels.co
        </Link>
        
        {/* Desktop nav */}
        <div className="hidden lg:flex items-center gap-8">
          {NAV_LINKS.map((link) => (
            <Link
              key={link.href}
              href={link.href}
              className={`text-xs font-mono tracking-widest uppercase transition-colors relative group ${
                pathname === link.href ? 'text-white' : 'text-white/40 hover:text-white'
              }`}
            >
              {link.label}
              {pathname === link.href && (
                <motion.span layoutId="nav-underline" className="absolute -bottom-1 left-0 right-0 h-px bg-white" />
              )}
            </Link>
          ))}
        </div>

        {/* Right side */}
        <div className="flex items-center gap-4">
          {/* Availability dot + label — from Firebase: config.availability_status + config.availability_label */}
          <div className="hidden md:flex items-center gap-2">
            <span className="relative flex h-1.5 w-1.5">
              <span className="animate-ping absolute inline-flex h-full w-full rounded-full bg-white opacity-40" />
              <span className="relative inline-flex rounded-full h-1.5 w-1.5 bg-white" />
            </span>
            <span className="text-xs font-mono text-white/30 hidden lg:block">{config.availability_label}</span>
          </div>

          <Link
            href="/contact"
            className="hidden md:block px-5 py-2.5 border border-white/20 text-xs font-mono tracking-widest hover:bg-white hover:text-black transition-all"
          >
            GET IN TOUCH
          </Link>

          {/* Hamburger */}
          <button
            className="lg:hidden w-10 h-10 flex flex-col items-center justify-center gap-1.5"
            onClick={() => setMenuOpen(!menuOpen)}
            aria-label="Toggle menu"
          >
            <motion.span animate={{ rotate: menuOpen ? 45 : 0, y: menuOpen ? 6 : 0 }} className="block w-6 h-px bg-white origin-center" />
            <motion.span animate={{ opacity: menuOpen ? 0 : 1 }} className="block w-6 h-px bg-white" />
            <motion.span animate={{ rotate: menuOpen ? -45 : 0, y: menuOpen ? -6 : 0 }} className="block w-6 h-px bg-white origin-center" />
          </button>
        </div>
      </nav>

      {/* Mobile menu — social links from Firebase */}
      <AnimatePresence>
        {menuOpen && (
          <motion.div
            initial={{ opacity: 0, y: -20 }}
            animate={{ opacity: 1, y: 0 }}
            exit={{ opacity: 0, y: -20 }}
            transition={{ duration: 0.3 }}
            className="fixed inset-0 z-40 bg-[#080808] pt-20 px-6 flex flex-col"
          >
            <nav className="flex-1 flex flex-col justify-center space-y-4">
              {NAV_LINKS.map((link, i) => (
                <motion.div
                  key={link.href}
                  initial={{ opacity: 0, x: -20 }}
                  animate={{ opacity: 1, x: 0 }}
                  transition={{ delay: i * 0.06 }}
                >
                  <Link href={link.href}
                    className={`font-display text-[clamp(36px,8vw,64px)] leading-tight block transition-opacity ${
                      pathname === link.href ? 'opacity-100' : 'opacity-40 hover:opacity-100'
                    }`}
                  >
                    {link.label}
                  </Link>
                </motion.div>
              ))}
            </nav>
            <div className="py-8 border-t border-white/6 flex justify-between items-end">
              {/* owner_email from Firebase */}
              <a href={`mailto:${config.owner_email}`} className="text-sm font-mono text-white/30 hover:text-white transition-colors">
                {config.owner_email}
              </a>
              <div className="flex gap-4">
                {/* Social links — from Firebase: REAL URLs */}
                {socialLinks.slice(0, 4).map((link: any) => (
                  <a key={link.id} href={link.url} target="_blank" rel="noopener noreferrer"
                    className="text-xs font-mono text-white/30 hover:text-white transition-colors"
                  >
                    {link.platform}
                  </a>
                ))}
              </div>
            </div>
          </motion.div>
        )}
      </AnimatePresence>
    </>
  );
}
```

---

## 🔐 FIREBASE SETUP — EXACT COLLECTION SCHEMA

### How to create every collection

**portfolio_config / main** — Create this document manually in Firebase Console:
```
Field name                Type      Example value
owner_name                string    "coldpixels.co"
owner_title               string    "Digital Designer & Developer"
owner_bio_short           string    "I design and build premium digital products."
owner_bio_long            string    "Full multi-paragraph bio here. Use \n\n for paragraphs."
owner_email               string    "hello@coldpixels.co"
owner_phone               string    "+1 (555) 000-0000"
owner_location            string    "New York, NY"
owner_timezone            string    "EST"
availability_status       string    "open"
availability_label        string    "Available for new projects"
hero_headline_word1       string    "DESIGN"
hero_headline_word2       string    "DEVELOP"
hero_headline_word3       string    "DELIVER"
hero_subheadline          string    "Precision in every pixel."
hero_cta_primary_label    string    "View My Work"
hero_cta_primary_url      string    "/work"
hero_cta_secondary_label  string    "Get In Touch"
cv_url                    string    "/cv.pdf"
about_headline            string    "Building the web with cold precision."
marquee_items             array     ["Design Systems", "Motion Design", "Web Dev"]
footer_tagline            string    "Let's build something remarkable."
footer_copy               string    "Crafted by coldpixels.co"
copyright_year            number    2025
show_testimonials         boolean   true
show_awards               boolean   true
announcement_bar_enabled  boolean   false
announcement_bar_text     string    "Now accepting new projects"
announcement_bar_link     string    "/contact"
contact_form_project_types array   ["UI/UX Design","Web Development","Brand Identity"]
contact_form_budget_ranges array   ["< $5K","$5K–$15K","$15K–$30K","$30K+"]
response_time             string    "Usually responds within 24 hours"
calendly_url              string    "" (leave blank if no Calendly)
```

**social_links / {auto-id}** — Create one document per link:
```
platform    string    "GitHub"
url         string    "https://github.com/yourusername"    ← FULL URL with https://
icon        string    "github"
label       string    "GitHub"
order       number    1
visible     boolean   true
```
Repeat for: Twitter → "https://twitter.com/yourhandle", LinkedIn, Dribbble, Instagram etc.

**projects / {auto-id}:**
```
slug                string    "brand-identity-project"
title               string    "Brand Identity — Studio X"
subtitle            string    "Complete visual identity"
short_description   string    "Full rebrand including logo and digital guidelines."
category            string    "Brand Identity"
tags                array     ["Branding", "Logo", "Typography"]
client              string    "Studio X"
year                number    2024
role                string    "Brand Designer"
duration            string    "6 weeks"
cover_image_url     string    "https://images.unsplash.com/photo-xxx?w=1200"
thumbnail_url       string    "https://images.unsplash.com/photo-xxx?w=800"
challenge           string    "The brand had no consistent visual language."
approach            string    "Built a modular system with 80-page brand book."
outcome             string    "Brand recognition doubled in 3 months."
tech_stack          array     ["Figma", "Illustrator", "After Effects"]
live_url            string    "https://clientwebsite.com"  ← FULL URL — shows as live link
github_url          string    "https://github.com/yourrepo" ← FULL URL — shows as code link
gallery             array     [] (add image URLs)
featured            boolean   true
featured_order      number    1
visible             boolean   true
order               number    1
view_count          number    0
```

**skills / {auto-id}:**
```
name               string    "Figma"
category           string    "Design"   ← Must be: "Design" | "Development" | "Tools"
proficiency        number    95         ← 0-100, shows as fill bar %
years_experience   number    5
order              number    1
visible            boolean   true
```

**experience / {auto-id}:**
```
company        string    "Your Company"
role           string    "Senior Designer"
type           string    "full-time"   ← "full-time" | "freelance" | "contract"
start_date     string    "Jan 2022"
end_date       string    "Present"     ← Use "Present" for current job
description    string    "Led design for..."
achievements   array     ["Reduced load time 40%", "Led team of 5", "Won design award"]
order          number    1
visible        boolean   true
```

**faqs / {auto-id}:**
```
question          string    "What is your typical project timeline?"
answer            string    "Most projects take 4-8 weeks depending on scope..."
category          string    "Process"   ← Choose any category label you like
helpful_count     number    0
not_helpful_count number    0
order             number    1
visible           boolean   true
```

**services / {auto-id}:**
```
name             string    "Brand Identity"
tagline          string    "From concept to complete brand system"
description      string    "Complete visual identity design including logo..."
features         array     ["Logo Design", "Color System", "Typography", "Brand Guidelines"]
deliverables     array     ["Logo files (SVG/PNG/PDF)", "Brand book PDF", "Style guide"]
timeline         string    "3-4 weeks"
starting_price   string    "$2,500"
popular          boolean   false
order            number    1
visible          boolean   true
```

**stats / {auto-id}:**
```
value    number    50
suffix   string    "+"
label    string    "Projects Completed"
icon     string    "briefcase"
order    number    1
visible  boolean   true
```

**awards / {auto-id}:**
```
title          string    "Best Digital Design 2024"
organization   string    "Awwwards"
year           number    2024
category       string    "UI Design"
url            string    "https://awwwards.com/..."  ← FULL URL — shows as clickable link
order          number    1
visible        boolean   true
```

---

## 🚫 REMOVE SPLINE COMPLETELY

Search codebase for any of these strings and DELETE them:
```
import Spline
import { Spline }
from '@splinetool/react-spline'
from '@splinetool/runtime'
spline_hero_url
spline_contact_url
config.spline_hero_url
config.spline_contact_url
<Spline
```

Replace any Spline component JSX with this CSS-only animated background:
```tsx
<div className="w-full h-full relative overflow-hidden">
  {/* Animated gradient orbs — pure CSS, no library */}
  <div className="absolute inset-0">
    <div className="absolute top-1/4 left-1/4 w-96 h-96 rounded-full bg-white/3 blur-3xl animate-pulse" style={{ animationDuration: '4s' }} />
    <div className="absolute bottom-1/4 right-1/4 w-64 h-64 rounded-full bg-white/2 blur-2xl animate-pulse" style={{ animationDuration: '6s', animationDelay: '2s' }} />
    <div className="absolute top-1/2 left-1/2 -translate-x-1/2 -translate-y-1/2 w-48 h-48 rounded-full bg-white/4 blur-xl animate-pulse" style={{ animationDuration: '3s', animationDelay: '1s' }} />
  </div>
  {/* Grid lines */}
  <div className="absolute inset-0" style={{
    backgroundImage: 'linear-gradient(rgba(255,255,255,0.03) 1px, transparent 1px), linear-gradient(90deg, rgba(255,255,255,0.03) 1px, transparent 1px)',
    backgroundSize: '80px 80px'
  }} />
</div>
```

---

## 🔗 FIRESTORE SECURITY RULES

Replace the entire content of `firestore.rules`:
```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    // Public read for everything
    match /{document=**} {
      allow read: if true;
      allow write: if false;
    }
    // Contact form: create only, with validation
    match /contact_submissions/{id} {
      allow create: if
        request.resource.data.name is string && request.resource.data.name.size() >= 2 &&
        request.resource.data.email is string && request.resource.data.email.matches('[^@]+@[^@]+\\.[^@]+') &&
        request.resource.data.message is string && request.resource.data.message.size() >= 10;
    }
    // FAQ voting: only helpful_count or not_helpful_count can be incremented
    match /faqs/{id} {
      allow update: if
        request.resource.data.diff(resource.data).affectedKeys()
          .hasOnly(['helpful_count', 'not_helpful_count']);
    }
    // Project views: only view_count can be incremented
    match /projects/{id} {
      allow update: if
        request.resource.data.diff(resource.data).affectedKeys()
          .hasOnly(['view_count']);
    }
  }
}
```

---

## ✅ GSAP ANIMATIONS — ADD EVERYWHERE

Install: `npm install gsap @gsap/react`

Register plugins in each component that needs them:
```typescript
import gsap from 'gsap';
import { ScrollTrigger } from 'gsap/ScrollTrigger';
gsap.registerPlugin(ScrollTrigger);
```

**Pattern — add to any section component:**
```typescript
useEffect(() => {
  const ctx = gsap.context(() => {
    // Stagger reveal children
    gsap.fromTo('[data-reveal]',
      { y: 50, opacity: 0 },
      { y: 0, opacity: 1, duration: 0.8, stagger: 0.12, ease: 'power3.out',
        scrollTrigger: { trigger: '[data-anim-section]', start: 'top 80%', once: true } }
    );
    // Counter animation for stats
    document.querySelectorAll('[data-count]').forEach((el) => {
      const target = parseInt(el.getAttribute('data-target') ?? '0');
      gsap.to({ val: 0 }, {
        val: target, duration: 2, ease: 'power2.out',
        scrollTrigger: { trigger: el, start: 'top 85%', once: true },
        onUpdate: function() { el.textContent = Math.floor(this.targets()[0].val) + ''; }
      });
    });
  });
  return () => ctx.revert();
}, []);
```

---

## ⚡ FINAL EXECUTION ORDER

```
1. Replace src/lib/firebase.ts
2. Replace src/lib/firestore.ts
3. Replace src/hooks/useConfig.ts
4. Replace src/hooks/useSocialLinks.ts
5. Replace src/hooks/useStats.ts
6. Replace src/hooks/useProjects.ts
7. Replace src/components/layout/PageTransition.tsx
8. Replace app/layout.tsx (adds Navbar, Footer, PageTransition to all pages)
9. Replace app/page.tsx
10. Replace app/about/page.tsx
11. Replace app/work/page.tsx
12. Replace app/work/[slug]/page.tsx
13. Replace app/contact/page.tsx
14. Replace app/services/page.tsx
15. Replace app/faq/page.tsx
16. Replace src/components/layout/Navbar.tsx
17. Replace src/components/layout/Footer.tsx
18. Delete all Spline imports anywhere
19. Create .env.local.example
20. Deploy firestore.rules: firebase deploy --only firestore:rules
21. Run: npm install gsap @gsap/react framer-motion react-hot-toast
22. npm run dev — verify no errors
23. npm run build — must complete with zero TypeScript errors
```

---

## 🎯 EVERY FIREBASE FIELD ON WEBSITE — VERIFICATION TABLE

| Firebase Field | Collection | Shows On |
|---|---|---|
| owner_name | portfolio_config/main | Navbar logo, About hero, Footer |
| owner_title | portfolio_config/main | About hero h1 |
| owner_bio_short | portfolio_config/main | About hero, Footer bio |
| owner_bio_long | portfolio_config/main | About biography section |
| owner_email | portfolio_config/main | Contact page, Footer, Autofill TO: |
| owner_phone | portfolio_config/main | Contact page info |
| owner_location | portfolio_config/main | Contact page, Footer |
| owner_timezone | portfolio_config/main | About hero |
| availability_status | portfolio_config/main | Navbar dot, About hero dot, Footer dot |
| availability_label | portfolio_config/main | Navbar label, About hero, Footer |
| hero_headline_word1/2/3 | portfolio_config/main | Home hero h1 (3 lines) |
| hero_subheadline | portfolio_config/main | Home hero paragraph |
| hero_cta_primary_label | portfolio_config/main | Home hero button |
| hero_cta_primary_url | portfolio_config/main | Home hero button href |
| footer_tagline | portfolio_config/main | Footer CTA belt |
| footer_copy | portfolio_config/main | Footer bottom bar |
| copyright_year | portfolio_config/main | Footer bottom bar |
| show_awards | portfolio_config/main | Awards section visible/hidden |
| show_testimonials | portfolio_config/main | Testimonials section visible/hidden |
| announcement_bar_enabled | portfolio_config/main | Announcement bar shows/hides |
| announcement_bar_text | portfolio_config/main | Announcement bar text |
| contact_form_project_types | portfolio_config/main | Contact form chips |
| contact_form_budget_ranges | portfolio_config/main | Contact form budget chips |
| response_time | portfolio_config/main | Contact page info |
| calendly_url | portfolio_config/main | Contact page "Book a call" button |
| social_links.platform | social_links collection | About page, Contact page, Footer, Navbar mobile |
| social_links.url | social_links collection | REAL clickable links everywhere above |
| project.title | projects collection | Work grid card, Detail page h1 |
| project.cover_image_url | projects collection | Work grid card image, Detail hero |
| project.live_url | projects collection | Detail page "VISIT LIVE SITE" button |
| project.github_url | projects collection | Detail page "VIEW CODE" button |
| project.tags | projects collection | Work grid chips, Detail page chips |
| project.challenge/approach/outcome | projects collection | Detail page sections |
| skill.name + proficiency | skills collection | About skills bars |
| experience.company/role | experience collection | About timeline |
| faq.question/answer | faqs collection | FAQ page accordion |
| faq.helpful_count | faqs collection | FAQ voting display |
| service.name/features | services collection | Services page accordion |
| stat.value/label | stats collection | Home stats, About stats grid |
| award.title/url | awards collection | About awards grid (url is clickable) |