# Firebase Setup Guide for coldpixels.co
## Complete Step-by-Step — Written for Someone Who Has Never Used Firebase
## Takes about 20-30 minutes. Zero experience needed.

---

## 🧒 WHAT IS FIREBASE? (5-year-old explanation)

Imagine your website is a toy shop.

The **shop itself** is your website — it has shelves, lights, and a front door.

But the **items on the shelves** (your projects, bio, email, FAQ answers) — where do you store them? You could hardcode them into the website code, but then every time you want to change your email or add a new project, you'd have to open code files and redeploy.

**Firebase is like a magic notebook** in the cloud. Your website reads from this notebook in real-time. You want to change your email? Open the notebook, change one line. Done. Your website updates everywhere instantly — no code changes, no redeployment.

That magic notebook is called **Firestore Database**.

Firebase also gives you:
- **Hosting** (optional — we use Vercel for this project)
- **Analytics** (who visits your site)
- **Storage** (for uploading images — optional)

For this project, we use **Firestore Database** as our magic notebook.

---

## PART 1 — CREATE YOUR FIREBASE PROJECT

### Step 1: Go to Firebase Console
1. Open your browser
2. Go to: **https://console.firebase.google.com**
3. Sign in with your Google account (the same one you use for Gmail)
   — If you don't have a Google account, create one first at google.com

### Step 2: Create a new project
1. Click the big button that says **"Add project"** or **"Create a project"**
2. **Project name:** Type `coldpixels-co` (or anything you like — this is just a label for you)
3. Click **Continue**
4. Next screen asks about Google Analytics — you can **disable it** for now (toggle it off)
   — You can always turn it on later
5. Click **Create project**
6. Wait 10-20 seconds while it spins
7. Click **Continue** when it's done

You now have a Firebase project. 🎉

---

## PART 2 — SET UP FIRESTORE DATABASE

Firestore is the "magic notebook" where all your content lives.

### Step 3: Create the database
1. In the left sidebar, click **"Build"** to expand the menu
2. Click **"Firestore Database"**
3. Click the blue button **"Create database"**
4. A popup appears asking about security rules:
   - Select **"Start in test mode"**
   - ⚠️ This allows anyone to read AND write for 30 days — we'll fix this shortly
5. Click **Next**
6. Choose a location — pick the one closest to you
   - If you're in India: choose `asia-south1 (Mumbai)` or `asia-southeast1`
   - If you're in USA: choose `us-central1`
   - If you're in Europe: choose `europe-west1`
7. Click **Enable**
8. Wait for it to create. Your database is ready!

---

## PART 3 — GET YOUR SECRET KEYS

Your website needs special keys to talk to Firebase. Like a password.

### Step 4: Register your web app
1. Click the **gear icon ⚙️** next to "Project Overview" in the top-left
2. Click **"Project settings"**
3. Scroll down to the section that says **"Your apps"**
4. Click the **web icon** (it looks like `</>`)
5. **App nickname:** Type `coldpixels-web`
6. **Do NOT check** "Also set up Firebase Hosting" (we use Vercel)
7. Click **"Register app"**

### Step 5: Copy your keys
After registering, you'll see a block of code that looks like this:
```javascript
const firebaseConfig = {
  apiKey: "AIzaSy...",
  authDomain: "your-project.firebaseapp.com",
  projectId: "your-project-id",
  storageBucket: "your-project.appspot.com",
  messagingSenderId: "123456789",
  appId: "1:123456789:web:abc123"
};
```

**COPY EACH VALUE.** You'll need them in the next step.

Click **"Continue to console"** when done.

---

## PART 4 — ADD YOUR KEYS TO THE WEBSITE

### Step 6: Create your .env.local file
In your project folder on your computer, find the file called `.env.local.example`.

1. **Duplicate** that file
2. **Rename** the duplicate to exactly: `.env.local` (remove the word "example")
3. Open `.env.local` in any text editor (Notepad, VS Code, TextEdit)
4. Fill in your values from Step 5:

```
NEXT_PUBLIC_FIREBASE_API_KEY=AIzaSy... (paste your apiKey here)
NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN=your-project.firebaseapp.com
NEXT_PUBLIC_FIREBASE_PROJECT_ID=your-project-id
NEXT_PUBLIC_FIREBASE_STORAGE_BUCKET=your-project.appspot.com
NEXT_PUBLIC_FIREBASE_MESSAGING_SENDER_ID=123456789
NEXT_PUBLIC_FIREBASE_APP_ID=1:123456789:web:abc123
```

**Important rules:**
- No spaces around the `=` sign
- No quotes around the values
- No comma at the end
- Save the file

---

## PART 5 — DEPLOY SECURITY RULES

Right now your database is in "test mode" — anyone can write to it! We need to lock it down.

### Step 7: Deploy the security rules

**Option A — Using Firebase CLI (recommended):**

1. Open your terminal (Command Prompt on Windows, Terminal on Mac)
2. Install Firebase tools by typing:
   ```
   npm install -g firebase-tools
   ```
   Press Enter and wait for it to finish.

3. Login to Firebase:
   ```
   firebase login
   ```
   This opens your browser — click "Allow"

4. In your project folder, run:
   ```
   firebase init firestore
   ```
   - "Use an existing project" → select your project
   - "What file should be used for Firestore Rules?" → press Enter (use default: firestore.rules)
   - "What file should be used for Firestore indexes?" → press Enter

5. Deploy the rules:
   ```
   firebase deploy --only firestore:rules
   ```
   Done! Your database is now secure.

**Option B — Manual (if CLI is too confusing):**

1. Go to Firebase Console → Firestore Database
2. Click the **"Rules"** tab at the top
3. Delete everything in the text box
4. Paste the entire contents of your `firestore.rules` file
5. Click **"Publish"**

---

## PART 6 — ADD YOUR CONTENT (The Magic Notebook)

Now we fill the notebook. All your site content lives here.

### HOW FIRESTORE WORKS:
- **Collections** = chapters in the notebook (like "Projects", "FAQ", "Skills")
- **Documents** = individual pages in each chapter
- **Fields** = lines on each page (like "title: My App", "year: 2024")

Think of it like a spreadsheet:
- Collection = a spreadsheet file
- Document = a row in the spreadsheet
- Field = a cell/column in that row

---

### Step 8: Create the portfolio_config document

This is the most important one — it controls your entire site.

1. In Firebase Console → Firestore Database → click **"+ Start collection"**
2. **Collection ID:** `portfolio_config`
3. Click **Next**
4. **Document ID:** type exactly `main`
5. Now add fields one by one (click **"+ Add field"** for each):

```
FIELD NAME                          TYPE        VALUE
─────────────────────────────────────────────────────────
owner_name                          string      coldpixels.co
owner_name_display                  string      coldpixels.co
owner_title_line1                   string      Digital Designer
owner_title_line2                   string      & Developer
owner_tagline                       string      Precision. Pixels. Perfect.
owner_bio_short                     string      I design and build premium digital products that blend aesthetics with functionality.
owner_bio_long                      string      [your full bio here, can be multiple sentences]
owner_email                         string      hello@coldpixels.co
owner_phone                         string      +1 (555) 123-4567
owner_location                      string      New York, NY
owner_timezone                      string      EST (UTC-5)
owner_availability_status           string      open
owner_availability_label            string      Available for new projects
owner_response_time                 string      Usually responds within 24 hours

hero_headline_word1                 string      DESIGN
hero_headline_word2                 string      DEVELOP
hero_headline_word3                 string      DELIVER
hero_subheadline                    string      Creating digital experiences that resonate
hero_cta_primary_label              string      View My Work
hero_cta_primary_url                string      /work
hero_cta_secondary_label            string      Download CV
cv_url                              string      /cv.pdf

logo_text                           string      coldpixels.co
og_title                            string      coldpixels.co — Digital Craft Studio
og_description                      string      Premium digital portfolio by coldpixels.co
site_url                            string      https://coldpixels.co

footer_tagline                      string      Let's build something remarkable.
footer_copy                         string      Crafted by coldpixels.co
copyright_year                      number      2025

show_blog                           boolean     false
show_pricing                        boolean     false
show_testimonials                   boolean     true
show_awards                         boolean     true
loading_screen_enabled              boolean     true
scroll_progress_bar                 boolean     true
back_to_top_button                  boolean     true
announcement_bar_enabled            boolean     false
announcement_bar_text               string      Available for new projects — Let's talk
announcement_bar_link               string      /contact

contact_form_project_types          array       ["UI/UX Design", "Brand Identity", "Web Development", "Motion Design", "Full-Stack App", "Creative Direction", "Something Else"]

ripple_enabled                      boolean     true
hero_particle_count                 number      120
```

**How to add an array field:**
1. Set type to "array"
2. Click "Add value" for each item
3. Each item type: "string"

6. Click **"Save"**

---

### Step 9: Add your Projects

1. Click **"+ Start collection"**
2. **Collection ID:** `projects`
3. Click **Next**
4. **Document ID:** click "Auto-ID"
5. Add these fields for your first project:

```
FIELD NAME            TYPE      VALUE
─────────────────────────────────────────────────────
slug                  string    my-first-project
title                 string    Brand Identity — Studio X
subtitle              string    Visual identity for a creative studio
short_description     string    Complete rebrand including logo, typography, and digital guidelines.
category              string    Brand Identity
tags                  array     ["Branding", "Logo", "Typography"]
client                string    Studio X
year                  number    2024
cover_image_url       string    https://your-image-url.com/cover.jpg
featured              boolean   true
featured_order        number    1
visible               boolean   true
view_count            number    0
order                 number    1
live_url              string    https://studiox.com
tech_stack            array     ["Figma", "Illustrator", "After Effects"]
```

6. Click **Save**
7. Repeat for each project (click **"+ Add document"** inside the projects collection)

**IMPORTANT:** The `slug` must be URL-friendly:
- ✅ `my-project-name`
- ❌ `My Project Name`
- Only lowercase letters, numbers, and hyphens

---

### Step 10: Add your Skills

1. New collection: **skills**
2. Auto-ID documents
3. Add fields for each skill:

```
name            string    Figma
category        string    Design
proficiency     number    95
years_experience number   5
featured        boolean   true
order           number    1
visible         boolean   true
```

Repeat for each skill. Categories must be: `Design`, `Development`, or `Tools`

---

### Step 11: Add your Experience

1. New collection: **experience**
2. Auto-ID documents

```
company         string    Your Company Name
role            string    Senior Designer
type            string    full-time
start_date      string    Jan 2022
end_date        string    Present
description     string    Brief description of your role and impact
achievements    array     ["Achievement 1", "Achievement 2", "Achievement 3"]
tech_used       array     ["Figma", "React", "Webflow"]
order           number    1
visible         boolean   true
```

---

### Step 12: Add FAQs

1. New collection: **faqs**
2. Auto-ID documents

```
question        string    What is your typical project timeline?
answer          string    Most projects take 4-8 weeks depending on scope...
category        string    Process
featured        boolean   true
helpful_count   number    0
not_helpful_count number  0
order           number    1
visible         boolean   true
```

Categories can be anything: `Process`, `Pricing`, `Technical`, `General`, etc.

---

### Step 13: Add Social Links

1. New collection: **social_links**
2. Auto-ID documents

```
platform        string    GitHub
url             string    https://github.com/yourusername
icon            string    github
label           string    GitHub
order           number    1
visible         boolean   true
```

Repeat for: LinkedIn, Twitter/X, Dribbble, Instagram, etc.
The `icon` field is the Lucide icon name (lowercase).

---

### Step 14: Add Stats

1. New collection: **stats**
2. Auto-ID documents

```
value           number    50
suffix          string    +
label           string    Projects Completed
icon            string    briefcase
order           number    1
visible         boolean   true
```

Suggested stats: Projects, Clients, Years of Experience, Coffee Cups ☕

---

### Step 15: Add Testimonials (optional)

1. New collection: **testimonials**

```
name            string    Jane Smith
role            string    CEO
company         string    Acme Inc
avatar_url      string    https://...
quote           string    Working with coldpixels.co was incredible...
rating          number    5
featured        boolean   true
approved        boolean   true
order           number    1
visible         boolean   true
```

---

### Step 16: Add Services

1. New collection: **services**

```
name              string    Brand Identity
icon              string    pen-tool
tagline           string    From concept to brand system
short_description string    Complete visual identity design
full_description  string    Detailed description of what's included...
features          array     ["Logo Design", "Color System", "Typography", "Brand Guidelines"]
deliverables      array     ["Logo files (SVG, PNG, PDF)", "Brand book PDF"]
timeline          string    3-4 weeks
starting_price    string    $2,500
popular           boolean   false
order             number    1
visible           boolean   true
```

---

### Step 17: Add Beam Nodes (for the animated diagram)

The animated beam diagram is the visual showing connected nodes.

1. New collection: **beam_nodes**

```
node_key        string    design
label           string    Design
sublabel        string    UI/UX
icon_svg        string    <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5"><path d="M12 2L2 7l10 5 10-5-10-5zM2 17l10 5 10-5M2 12l10 5 10-5"/></svg>
position_x      number    50
position_y      number    50
is_center       boolean   true
section         string    home
visible         boolean   true
order           number    1
```

Add 5-7 nodes for each section (home and features).
`position_x` and `position_y` are percentages (0-100) of the container.
Center node: usually at 50, 50.

2. New collection: **beam_connections**

```
from_node       string    design
to_node         string    development
curvature       number    0.3
reverse         boolean   false
delay           number    0
duration        number    2000
section         string    home
visible         boolean   true
```

Connect your nodes to create the beam diagram.

---

## PART 7 — DEPLOY SECURITY RULES

Your database is now full of content. Let's secure it properly.

### What the rules do:
- ✅ Anyone can READ all data (so your website can show it)
- ❌ Nobody can WRITE (except through specific safe operations)
- ✅ Contact form: users can submit (but not read other submissions)
- ✅ Project views: users can increment view count only
- ✅ FAQ helpful: users can vote once

The `firestore.rules` file in your project contains all these rules.
Deploy them using Step 7 from above (firebase deploy --only firestore:rules).

---

## PART 8 — HOW TO UPDATE CONTENT

### 🔄 Change your email (the MOST IMPORTANT example)

1. Go to Firebase Console
2. Click Firestore Database
3. Click `portfolio_config` in the left panel
4. Click the document `main`
5. Find the field `owner_email`
6. Click the pencil ✏️ icon next to it
7. Type your new email
8. Click the checkmark ✓

**That's it.** Your website will update within seconds everywhere:
- The "TO:" field in Autofill Compose
- The contact info panel
- The footer
- Everywhere your email appears

No code change. No deployment. Just change the Firebase field.

---

### 📝 Add a new project

1. Firebase Console → Firestore Database
2. Click `projects` collection
3. Click **"+ Add document"**
4. Auto-ID
5. Fill in all the fields (copy from an existing project document for reference)
6. Set `visible: true`
7. Set `featured: true` if you want it on the home page
8. Save

Your website shows it immediately (within 60 seconds due to ISR cache).

---

### 🚦 Toggle availability status

Field: `portfolio_config / main / owner_availability_status`
Values: `open`, `busy`, or `closed`

The status dot on your navbar and contact page updates in real-time.

---

### 📢 Show an announcement bar

1. `portfolio_config / main / announcement_bar_enabled` → change to `true`
2. `portfolio_config / main / announcement_bar_text` → type your message
3. Save

A bar appears at the top of your site immediately.

---

### 🎨 Add a FAQ

1. `faqs` collection → **"+ Add document"** → Auto-ID
2. Fill in: `question`, `answer`, `category`, `order: 99`, `visible: true`
3. Save

Appears on the FAQ page immediately.

---

## PART 9 — ENVIRONMENT VARIABLES ON VERCEL

When you deploy to Vercel, you need to add your Firebase keys there too.

1. Go to **vercel.com** and open your project
2. Click **Settings** → **Environment Variables**
3. Add each variable from your `.env.local` file:
   - Name: `NEXT_PUBLIC_FIREBASE_API_KEY`
   - Value: your actual key
   - Environment: Production + Preview + Development
4. Repeat for all 6 Firebase variables
5. Click **Redeploy** to apply changes

---

## PART 10 — COMMON MISTAKES (and how to fix them)

### ❌ "Firebase: No Firebase App" error
**Cause:** Your .env.local file is missing or the variable names are wrong
**Fix:** Double-check your .env.local file. Restart your dev server after editing it (`npm run dev` again).

### ❌ Site shows no data / shows placeholder content
**Cause:** Firebase is not connected, falling back to mock data
**Fix:** Check browser console for Firebase errors. Verify your Project ID in .env.local matches your Firebase project.

### ❌ "Missing or insufficient permissions"
**Cause:** Firestore security rules are blocking the read
**Fix:** Make sure you deployed the firestore.rules file. Check that the rule allows public reads.

### ❌ Autofill shows old email even after changing in Firebase
**Cause:** The email is cached (SWR caches for 60 seconds)
**Fix:** Wait 60 seconds OR hard refresh (Ctrl+Shift+R). The real-time listener should update faster.

### ❌ "Test mode will expire in X days" warning
**Cause:** You're still in test mode with broad write permissions
**Fix:** Deploy the firestore.rules file immediately (Step 7)

### ❌ Images not showing
**Cause:** Image URLs from Firebase need to be added to next.config.mjs domains
**Fix:** Add your image host to the `images.domains` array in next.config.mjs:
```javascript
images: {
  domains: ['firebasestorage.googleapis.com', 'images.unsplash.com']
}
```

---

## PART 11 — QUICK REFERENCE CARD

```
Collection            What it controls
──────────────────────────────────────────────────────────────────
portfolio_config      Everything: your name, email, bio, hero text,
                      feature toggles, Autofill email, nav links
projects              Your work/case studies
skills                The skills section on About page
experience            Your work history timeline
testimonials          Client quotes
faqs                  FAQ page content
services              Services page offerings
stats                 The "50+ Projects" stats section
awards                Awards & recognition grid
process_steps         The 5-step process section on Services page
social_links          Social media links (footer + navbar)
beam_nodes            Nodes in the animated beam diagram
beam_connections      Lines between beam diagram nodes
contact_submissions   Where contact form submissions are stored
                      (you READ these, users WRITE to them)
```

```
To change...                  Go to...
──────────────────────────────────────────────────────────────────
Your email (everywhere)       portfolio_config / main / owner_email
Your name                     portfolio_config / main / owner_name
Hero headline                 portfolio_config / main / hero_headline_word1/2/3
Availability status           portfolio_config / main / owner_availability_status
Add/remove project type chips portfolio_config / main / contact_form_project_types
Turn blog on/off              portfolio_config / main / show_blog
Announcement bar text         portfolio_config / main / announcement_bar_text
New project                   projects / (new document)
New FAQ                       faqs / (new document)
New skill                     skills / (new document)
New experience entry          experience / (new document)
Edit testimonial              testimonials / (find document, edit)
```

---

## ✅ SETUP CHECKLIST

```
Firebase Project:
  ☐ Project created at console.firebase.google.com
  ☐ Firestore Database created (test mode)
  ☐ Web app registered
  ☐ Firebase keys copied

.env.local:
  ☐ .env.local file created (not .env.local.example)
  ☐ All 6 NEXT_PUBLIC_ variables filled in
  ☐ No spaces around = signs
  ☐ File saved

Security Rules:
  ☐ firebase-tools installed (npm install -g firebase-tools)
  ☐ firebase login completed
  ☐ firebase deploy --only firestore:rules run
  ☐ Rules confirmed in Firebase Console → Rules tab

Content:
  ☐ portfolio_config / main document created
  ☐ owner_email field set to your real email
  ☐ owner_name set to coldpixels.co
  ☐ At least 1 project added to projects collection
  ☐ At least 3 FAQs added to faqs collection
  ☐ At least 5 skills added to skills collection
  ☐ At least 1 experience entry added
  ☐ Social links added
  ☐ Stats added (4 stats for the white section)
  ☐ 5+ services added
  ☐ Beam nodes added (5-7 per section)
  ☐ Beam connections added

Vercel (when deploying):
  ☐ All 6 environment variables added to Vercel
  ☐ Project redeployed after adding variables

Test:
  ☐ npm run dev starts without errors
  ☐ Site shows your actual data (not mock/placeholder)
  ☐ Autofill shows your email in the "TO:" field
  ☐ Changing owner_email in Firebase updates Autofill within 60 seconds
```

---

## 🆘 NEED HELP?

If something goes wrong:
1. Open your browser's Developer Tools (press F12)
2. Click the **Console** tab
3. Look for red error messages
4. Search the error text on Google — Firebase errors are very well documented

Firebase documentation: **https://firebase.google.com/docs/firestore/quickstart**

The most common issue is a typo in the .env.local file. Double-check every character.