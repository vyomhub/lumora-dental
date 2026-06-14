# ONE-SHOT PROMPT — "Build a premium dental clinic website from scratch"

> Paste everything below the line into any capable coding agent (Claude Code, Cursor, etc.).
> It reconstructs this exact website, animations and all, using `[PLACEHOLDERS]` for the
> business identity. Replace the placeholders at the top and the agent does the rest.
> The only obvious thing you change is **[BUSINESS_NAME]**. Everything else (copy, layout,
> animations, sections, legal pages) is fully specified.

---

You are a senior front-end engineer and designer. Build a complete, production-ready, **fully
self-contained static website** for a premium dental clinic. It must look like a real, original
agency build — never like a template. Deliver multi-page static HTML + CSS + vanilla JS with
GSAP-driven animations. No build step, no framework. It must run from `python3 -m http.server`.

## 0. Placeholders (fill these in once)
- `[BUSINESS_NAME]` = e.g. "Lumora Dental"  (the brand; appears in nav logo, titles, footer, copy)
- `[SHORT_NAME]` = the one-word brand mark, e.g. "Lumora"
- `[EMAIL]` = e.g. hello@lumoradental.com
- `[PHONE]` = e.g. +1 (555) 120-8787
- `[BOOKING_URL]` = a real scheduling link (Calendly etc.) for every appointment CTA
- `[ACCENT]` = primary accent color. Default teal `#24a3b1`. (Blue variant: `#2f80ff`.)
- `[CREDIT]` = footer maker credit, e.g. "Crafted by RapidXAI"
Use `[BUSINESS_NAME]` literally in copy where the clinic refers to itself ("At [SHORT_NAME], we…").

## 1. Tech & non-negotiables
- Static multi-page site. Pages: `index.html` (home), `about.html`, `service.html`, `blog.html`,
  plus legal: `privacy.html`, `terms.html`, `cookies.html`, `licenses.html`, and `404.html`.
- Font: **Sora** (Google Fonts, weights 300/400/500/600/700). Load via Google Fonts `<link>`.
- Animation libs (CDN or local): **GSAP 3.x**, **ScrollTrigger**, **SplitText**, plus **jQuery 3.5**.
- All assets local under `assets/css/`, `assets/js/`, `assets/img/`. Zero external CDN image calls.
- Every page: `<link rel="shortcut icon" href="assets/img/favicon.svg" type="image/svg+xml">` and
  an apple-touch-icon PNG.
- No em dashes in any copy. Use commas/periods.
- Accessibility: alt text on all images, semantic headings, focus-visible on interactive elements.

## 2. Design system (exact)
CSS custom properties on `:root`. This is the spine of the whole look.
```
--font: 'Sora', 'Helvetica Neue', Arial, sans-serif;
/* primary ramp (teal default; swap to the blue ramp for the blue variant) */
--primary-100:#eaf2ff; --primary-200:#d6e6ff; --primary-300:#a9c9ff;
--primary-400:#587d81; --primary-500:[ACCENT];  --primary-600:#1c91a1;
--primary-700:#022f34; --primary-800:#002124; --primary-900:#011f23;
/* blue variant ramp: 400:#6aa8ff 500:#2f80ff 600:#1f6fe5 700:#0b2a55 800:#07203f 900:#06182e */
--ink:#011f23;          /* primary text on light */
--muted:#5d6c7b;        /* secondary text */
--bg:#fafafa;           /* page background */
--bg-tint:#e2f1f3;      /* light section background (blue variant: #e8f1ff) */
--white:#ffffff;
--radius-card:18px; --radius-pill:999px;
--shadow-soft:0 18px 50px rgba(2,47,52,.10);
--maxw:1240px;
```
Type scale: hero H1 `clamp(48px,7vw,96px)` weight 600 letter-spacing -2px; section H2
`clamp(32px,5vw,64px)` weight 600; body 16-18px line-height 1.7; eyebrow/pill text 14px.
Buttons: pill (`--radius-pill`), 14px 26px padding, weight 500, with a circular arrow icon
(↗) on the right inside a contrasting disc. Primary button = dark (`--primary-900`) bg, white text.
Light/ghost button = white bg, dark text, subtle border.

## 3. Global components
### 3a. Navbar (transparent, absolute over hero, white links)
- Left: logo (SVG wordmark "[SHORT_NAME]" + a small tooth/drop mark in `[ACCENT]`). Use the
  **light/white** logo here because the nav sits over the dark hero image.
- Center: links Home, About Us, Services, Blog, and a "Pages" dropdown (About, Services, Blog,
  Privacy, Terms, Cookies, Licenses).
- Right: a primary pill button "Get Appointment" → `[BOOKING_URL]` (new tab).
- Links color white (`#f3f6ff`). Nav is `position:absolute`, transparent (it scrolls away with hero).

### 3b. Section eyebrow pill
A small rounded pill used above each section heading: `[ICON] LABEL` (e.g. "Our Story").
`[ICON]` is an inline 12x12 SVG sparkle using `fill="currentColor"`. Default = 4-point sparkle:
`<path d="M6 0C6.4 3.2 8.8 5.6 12 6C8.8 6.4 6.4 8.8 6 12C5.6 8.8 3.2 6.4 0 6C3.2 5.6 5.6 3.2 6 0Z"/>`

### 3c. Footer (dark, `--primary-900`)
- Big `[PHONE]` and `[EMAIL]` at top.
- Tagline + a "Get Appointment" button → `[BOOKING_URL]`.
- Columns: **Navigation** (Home/About/Services/Blog), **Legal** (Terms & Conditions→terms.html,
  Cookies→cookies.html, Licenses→licenses.html, 404→404.html), **Follow us** (Facebook, X,
  LinkedIn, Instagram — placeholder `#` or real URLs).
- Footer logo = white variant. Bottom bar: "© 2026 [BUSINESS_NAME]. [CREDIT]" on the left,
  "© 2026 [BUSINESS_NAME] · Privacy Policy" on the right.

## 4. The reveal engine (CRITICAL — this is the "feel")
Every content block fades and slides up as it enters the viewport. Implement with GSAP +
ScrollTrigger. Mark reveal targets with `data-reveal` (or reuse a baked `opacity:0` inline state).
Include a **safety net** so nothing can ever stay invisible. **Do not use blur** (it leaves images
stuck blurry if a trigger misfires). Exact script, before `</body>` on every page:
```html
<script>
(function () {
  function run() {
    if (!window.gsap) return;
    var hasST = !!window.ScrollTrigger;
    if (hasST) gsap.registerPlugin(ScrollTrigger);
    var els = gsap.utils.toArray('[data-reveal]').filter(function (el) {
      return !el.closest('nav') && !el.closest('.dropdown-list');
    });
    els.forEach(function (el) {
      gsap.fromTo(el, { opacity: 0, y: 40 }, {
        opacity: 1, y: 0, duration: 0.9, ease: 'power2.out',
        scrollTrigger: hasST ? { trigger: el, start: 'top 95%', once: true } : undefined
      });
    });
    if (hasST) { window.addEventListener('load', function(){ ScrollTrigger.refresh(); }); ScrollTrigger.refresh(); }
    setTimeout(function () { els.forEach(function (el) {
      if (parseFloat(getComputedStyle(el).opacity) === 0) gsap.set(el, { opacity: 1, y: 0 });
    }); }, 2600);
  }
  document.readyState !== 'loading' ? setTimeout(run,150)
    : document.addEventListener('DOMContentLoaded', function(){ setTimeout(run,150); });
})();
</script>
```
Also add **animated counters**: any `.stat-number` counts from 0 to its target on scroll
(`gsap.to` with `scrollTrigger`, snap, suffix preserved like "25+", "10k+").
Hero headline uses a subtle word-by-word rise (SplitText optional; a plain fade-up is fine).

## 5. Sliders
Two horizontal sliders (drag + dots + visible prev/next arrows):
- **Story slider** (home): rotating cards, each = a photo with a big metric overlay + caption.
- **Testimonial slider** (home): quote card with avatar, name, role; dots below.
Build with a simple translateX engine: track of slides, dot nav, arrow buttons (circular, with
←/→), pointer-drag support, and an auto-advance optional. Arrows MUST be visible.

## 6. Pages, section by section (with copy)

### HOME (`index.html`)
1. **Hero** (full-bleed photo background, dark overlay, content left):
   - Eyebrow none. H1: "Trusted Dental Care for Every Generation".
   - Sub: "We combine modern technology with heartfelt service to ensure every generation."
   - Primary button "Book Appointment" → `[BOOKING_URL]`.
2. **Our Story slider**: pill "Our Story". H2: "Redefining Dental Care with Trust, Innovation in
   Dental Wellness". Cards (metric + label + line):
   - "92% — Comfort & Care — Patients say they feel less anxious during visits with us"
   - "3/4 — Trusted by Community — New patients come from word-of-mouth referrals"
   - "7 Mi — Quick & Accessible — Just 7 minutes is the average wait time before being seen"
   - "24/7 — Emergency Support — Experiencing sudden pain or injury? Our team is here for you 24/7"
   - "85% — Healthy Habits — We're proud to help our community stay ahead of dental issues"
3. **Value / Why us**: pill "Why [SHORT_NAME]". H2 + paragraph + a supporting photo + a
   "Learn More" / book button. Copy: "At [SHORT_NAME], we combine expertise, compassion, and
   modern technology to create a dental experience patients truly value."
4. **Services overview**: pill "Our Services". H2 "Comprehensive Dental Care for Every Smile".
   4 cards w/ thumbnail: Preventive dentistry, Cosmetic dentistry, Restorative treatments,
   Orthodontics. Each with one-line description + arrow.
5. **Team**: pill "Our Team". H2 "Our Experts in Oral Health". 3 doctor cards (photo + name +
   role + social icons). Names: "Dr. [Name], Pediatric Dentist" etc. Keep names consistent with
   photo genders.
6. **Testimonials slider**: pill "Testimonials". H2 "Voices of Trust and Care". 3 quotes w/ avatar.
   Example: "I've always been nervous about visiting the dentist, but [SHORT_NAME] changed
   everything. The staff is warm, and the care is exceptional. I finally enjoy smiling again." —
   Kristin Watson, Business Owner.
7. **Closing CTA band** (dark): "Your health journey starts with one simple step, we're here to
   guide you." + "Get Started" button → `[BOOKING_URL]`.
8. **Footer**.

### ABOUT (`about.html`)
- Hero: pill "About Us". H1 "Trusted Dental Experts, Decades of Care". Sub: "At [SHORT_NAME], we
  combine compassion, innovation, and expertise to provide care you can trust. Your smile is our
  mission, and your comfort is our priority." Button "Book An Appointment".
  Right side stat counters: "25+ Years of Dental Excellence", "10k+ Smiles Transformed".
- Story block: pill "Our Story". H2 "Redefining Dental Care with Trust, Innovation in Dental
  Wellness, For more than two decades". Paragraph + masonry photo gallery (5 images).
- Success metrics cards (e.g. "50K+ Patient Visits", "4.9★ Google Rating").
- Awards / certifications strip.
- Team grid (6 doctors).
- Closing CTA + footer.

### SERVICES (`service.html`)
- Hero: pill "Our Services". H1 "Comprehensive Dental Care for You". Sub paragraph.
- 4 service cards with thumbnails + descriptions (Preventive, Cosmetic, Restorative, Orthodontics).
- An expandable service list (accordion rows: title + short description + arrow): e.g.
  "Restorative treatments — Fillings, crowns, implants, and more to restore the function and
  beauty of your smile"; "Orthodontics — Straighten your teeth and improve your bite with modern
  braces or clear aligners".
- **FAQ** accordion: pill "FAQ". H2 "Questions We Get Often". Q&A, e.g. "How often should I visit
  the dentist? We recommend a routine checkup every six months." / "Do you offer emergency dental
  services?" etc.
- Closing CTA + footer.

### BLOG (`blog.html`)
- Hero: H1 "Insights for a Healthier Smile". Sub "From everyday habits to advanced treatments,
  our dental experts share insights".
- Featured post card (big image left, category pill + date + title + excerpt + "Read More").
  Example title "The ultimate guide to brushing: are you doing it right?", category "Preventive
  Care", "Oral Health Tips · April 30, 2026".
- Grid of 6 post cards (image, category, title, excerpt). Titles: "5 daily dental habits",
  "How veneers transform your smile", "Foods that harm your teeth", "The truth about flossing",
  "Dental myths busted".
- Footer.

### LEGAL PAGES (privacy, terms, cookies, licenses) + 404
Simple, clean, on-brand pages sharing one lightweight inline style (max-width 760px, Sora,
`--ink` text, `[ACCENT]` links). Header = logo + "← Back to home". Footer = "© 2026
[BUSINESS_NAME]". Each has H1, "Last updated" line, and 4-6 sections:
- **Privacy**: information we collect, how we use it, how we protect it, your choices, contact.
- **Terms**: use of website, appointments (24h cancellation), intellectual property, liability,
  contact.
- **Cookies**: what cookies are, essential/analytics/preference cookies, managing cookies, contact.
- **Licenses**: site content ownership, imagery, third-party software (GSAP, Sora/OFL), permitted
  use, contact.
- **404**: big "404", "We couldn't find that page", "Back to home" button.
All legal links in the footer must resolve to these real files (no dead links).

## 7. Imagery (generate, do not use stock as-is)
Generate every photo with an image model (e.g. Magnific `gpt-2`, quality low, 1k, cheapest) so the
site is original, not a copy. Keep aspect ratios per slot. Save as JPG, reference locally. Prompts:
- Hero (16:9): "Photorealistic bright modern dental clinic, a friendly dentist in mask and gloves
  examining a relaxed smiling patient in the chair, soft daylight, [ACCENT]-and-white palette."
- About hero (16:9): friendly diverse dental team in a modern reception.
- Team portraits (2:3, distinct people): vary age/gender/ethnicity, white coats/scrubs, studio.
- Testimonial avatars (1:1): friendly patient headshots, different people.
- Service thumbnails (4:3): preventive cleaning; bright white smile (cosmetic); crown/implant model
  (restorative); clear aligners on a teen (orthodontics).
- Story slider cards (3:4): comfortable patient; team greeting family; bright reception; urgent care;
  healthy smile + toothbrush.
- Blog thumbnails (16:9): habits flatlay; veneers smile; sugary foods vs teeth; brushing; flossing;
  dentist explaining.
- Logo: code an SVG wordmark "[SHORT_NAME]" with a small tooth/drop mark in `[ACCENT]`; make a
  white variant for nav/footer and a dark variant for light pages. Favicon: rounded square in
  `[ACCENT]`→dark gradient with a white tooth mark; export a 180px PNG apple-touch-icon.

## 8. Color VARIANT (optional second theme)
To produce a brighter-blue variant without touching layout: duplicate the site, then
(a) override the `--primary-*` ramp to the blue ramp above, (b) sweep any hardcoded teal hexes
(`#24a3b1→#2f80ff`, `#011f23→#06182e`, `#022f34→#0b2a55`, `#1c91a1→#1f6fe5`, light tint
`#e2f1f3→#e8f1ff`), (c) recolor the logo/favicon SVGs the same way, (d) optionally change the
eyebrow sparkle to a different glyph. Same vibe, distinct identity.

## 9. Acceptance criteria (test before declaring done)
Run the site and verify in a real browser (Playwright):
1. All pages load with **zero console errors** and **zero external image/CDN requests**.
2. **All text is visible** (hero headline, every section heading and paragraph) and **no image is
   blurry** (the reveal must never leave anything at opacity 0 or blurred).
3. Scroll reveals fire; counters animate to their targets.
4. Both sliders: visible arrows, dots, drag all advance the slides.
5. Every nav/footer link resolves (including all 5 legal/404 pages). No dead links.
6. Every "Book/Get Appointment" CTA opens `[BOOKING_URL]`.
7. Logo is readable on the dark hero (use the white logo there).
8. `grep` the source: zero occurrences of any template/builder branding.

Deliver the full file tree, a short README, and run instructions. Make it feel premium,
calm, and trustworthy: generous whitespace, soft shadows, rounded cards, smooth easing.
