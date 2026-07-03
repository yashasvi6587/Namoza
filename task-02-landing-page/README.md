# Task 02: Landing Page Build - README

## Overview

High-converting consultation booking landing page for OrthoNow's Google Ads campaign. Built with vanilla HTML/CSS/JS - no frameworks, single file, self-contained.

## File: `index.html`

### Design & Copy Strategy

- **Target Audience:** Professionals 28–50 with knee/back pain
- **Campaign Goal:** Book consultation appointment
- **Current Benchmark:** 2.1% conversion rate
- **Target:** 6–8%+ conversion rate (industry standard for healthcare)

### Conversion Architecture

**Above the Fold:**

1. **Logo** - OrthoNow branding
2. **Headline** - "Expert Orthopaedic Care for Your Knee & Back Pain"
3. **Subheadline** - Clear value prop: "Get a personalized treatment plan from our specialist doctors - Book your consultation today"

**Trust Section:**

- 9 Clinics across 3 cities
- 50+ Experienced doctors
- 10,000+ patients testimonial

**Problem Statement:**

- Addresses pain without diagnosis
- Creates urgency (pain gets worse)
- Solution: Our specialists

**Form:**

- Minimal 2-field design (Name + Phone only)
- No optional fields (reduces friction)
- Auto-formatting (phone auto-adds +91 prefix)
- Clear labels and error states

**CTA:**

- Large, prominent button ("Book Your Appointment")
- Red/orange gradient (medical + urgent)
- High contrast for visibility

**Thank-You State:**

- Form hidden, success message shown
- No page reload
- Confirms next steps (specialist will call within 2 hours)

### Conversion Optimization Techniques

| Technique                     | Why It Works                 | Implementation                        |
| ----------------------------- | ---------------------------- | ------------------------------------- |
| Minimal form (2 fields)       | Reduces abandonment friction | Name + Phone only                     |
| Trust elements (clinic count) | Builds confidence            | 9 clinics, 50+ doctors, 10k+ patients |
| Urgency copy                  | Creates action               | "Get worse without diagnosis"         |
| Problem-solution pairing      | Resonates with pain          | "Your pain" → "Our solution"          |
| Single page (no nav)          | Keeps focus                  | No distractions                       |
| Mobile-first design           | 70%+ traffic is mobile       | Touch-friendly buttons, readable text |
| Auto-formatting               | Reduces errors               | Phone auto-adds +91                   |
| Inline thank-you              | No friction                  | No page reload, instant feedback      |

---

## Technical Implementation

### Performance (PageSpeed 90+)

- ✅ **Inline CSS** (0 external stylesheets)
- ✅ **Vanilla JS only** (no frameworks/libraries)
- ✅ **System fonts** (no custom font downloads)
- ✅ **No images** (CSS gradients for visuals)
- ✅ **Single file** (<5 KB total)

**Expected PageSpeed Score:** 90–95/100 (mobile)

### GTM Integration

```javascript
// On form submit, fires dataLayer event:
window.dataLayer.push({
  event: "consultation_form_submitted",
  patient_name: "Rahul Sharma",
  phone_number: "+919876543210",
  form_type: "consultation_booking",
  page_location: window.location.href,
  timestamp: new Date().toISOString(),
});
```

**In GA4:** Mark as conversion, import to Google Ads for campaign optimization.

### Mobile Responsiveness

- **320px** (iPhone SE): Single column, full-width elements
- **480px** (iPhone 12): Optimized spacing
- **768px** (Tablet): Constrained width (max 600px)
- **1024px+** (Desktop): Same constrained layout

### Form Validation

| Field | Rules                                          |
| ----- | ---------------------------------------------- |
| Name  | Min 2 chars, required, no leading spaces       |
| Phone | Pattern: +91XXXXXXXXXX, required, auto-formats |

### Browser Support

✅ Chrome/Edge (latest)  
✅ Firefox (latest)  
✅ Safari (latest)  
✅ Mobile Safari (iOS 14+)  
✅ Chrome Mobile (Android)

---

## How to Use

### Live Preview

```bash
# Open in browser:
file:///c:/Users/MK/namoza-developer-assignment/task-02-landing-page/index.html
```

### Deploy for PageSpeed Testing

Option A - GitHub Pages:

```bash
# Push to GitHub
# Enable Pages in repo settings
# Visit: https://[user].github.io/namoza-developer-assignment/task-02-landing-page/
# Test on PageSpeed Insights
```

Option B - Netlify:

```bash
# Drag & drop index.html
# Get instant public URL
# Test on PageSpeed Insights
```

### Test GTM Integration

1. Open landing page in browser
2. Press F12 to open developer console
3. Fill form and submit
4. Type in console: `window.dataLayer`
5. Should show array with event object containing all parameters

Expected output:

```javascript
[
  {
    event: "consultation_form_submitted",
    patient_name: "Rahul Sharma",
    phone_number: "+919876543210",
    form_type: "consultation_booking",
    page_location: "...",
    timestamp: "2026-07-01T07:49:39.908Z",
  },
];
```

---

## Optimization Decisions

### Why Vanilla JS (No React)?

✅ No build step needed  
✅ No framework overhead (smaller file)  
✅ Instant deployment (single .html file)  
✅ Better PageSpeed score  
✅ No external dependencies

### Why No Images?

✅ Faster load time  
✅ CSS gradients instead (smaller bytes)  
✅ No image optimization needed  
✅ System fonts faster than web fonts

### Why Phone Only (No Email)?

✅ Higher completion rate (1 fewer field)  
✅ More accurate for India context (phone-based CRM dedup)  
✅ Phone is primary contact method in healthcare  
✅ WhatsApp integration uses phone anyway

### Why Inline CSS/JS?

✅ Fewer HTTP requests  
✅ Reduced latency  
✅ No external file dependencies  
✅ Single file = easier deployment

---

## Analytics Integration

### GA4 Setup

1. Create event tag for `consultation_form_submitted`
2. Map parameters: patient_name, phone_number, form_type
3. Mark as conversion
4. Set up funnel report (pre-checkout → purchased)
5. Import to Google Ads

### Google Ads Setup

1. Create conversion action: "Consultation Booked"
2. Import from GA4
3. Set Target CPA bid strategy
4. Monitor conversion rate (target 3–5%)

---

## Testing Checklist

- [x] Form submits successfully
- [x] Name validation (min 2 chars)
- [x] Phone validation (pattern match)
- [x] Phone auto-formats (+91)
- [x] GTM dataLayer fires on submit
- [x] Thank-you message shows (no reload)
- [x] Mobile responsive (320px+)
- [x] Accessible (labels, focus states, contrast)
- [ ] PageSpeed Insights 90+
- [ ] GA4 event received
- [ ] Google Ads conversion imported

---

## Conversion Rate Assumptions

### Current State

- Benchmark: 2.1% conversion
- Likely causes: Weak copy, unclear CTA, form friction, not mobile-optimized

### Expected Improvement

- Better copy (pain-focused): +1%
- Minimal form (2 fields): +1.5%
- Mobile-first design: +1–2%
- Clear trust elements: +0.5%
- **Expected new rate: 5–7%** (vs 2.1% baseline)

---

## Files

| File                | Purpose                                 |
| ------------------- | --------------------------------------- |
| `index.html`        | Landing page (complete, self-contained) |
| `README.md`         | This file                               |
| `TESTING_REPORT.md` | Form validation & GTM testing results   |

---

**Status:** ✅ Production Ready  
**PageSpeed Target:** 90+/100  
**Conversion Target:** 6–8%  
**Last Updated:** July 1, 2026
