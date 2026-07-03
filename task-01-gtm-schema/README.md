# Task 01: GTM Event Schema - README

## Overview

Complete Google Tag Manager event tracking schema for OrthoNow's 3-step appointment booking funnel.

## Deliverables

- ✅ 18 event definitions with trigger types and GA4 parameters
- ✅ Booking form funnel tracking (3 steps) with exact JSON dataLayer pushes
- ✅ GA4 Funnel Exploration setup guide
- ✅ Google Ads conversion import recommendation
- ✅ Data quality validation rules
- ✅ Frontend developer briefing

## File: `GTM_EVENT_SCHEMA.md`

### Key Sections

**Part A: Complete Event Schema Table**

- 18 events across 7 categories
- Event name, trigger type, parameters, GA4 report, audience
- Covers: booking, CTAs, calls, WhatsApp, PDF, clinics, blog

**Part B: 3-Step Booking Funnel (DETAILED)**

- Step 1: Clinic + Specialty selection → JSON example
- Step 2: Contact info + preferred date → JSON example
- Step 3: Booking confirmation → JSON example + conversion fire
- Includes exact JavaScript code snippets for frontend dev
- Drop-off tracking in GA4 Funnel Exploration

**Part C: GA4 Conversion Setup**

- Which events to mark as conversions
- Why `booking_step_3_complete` is recommended
- GA4 funnel report configuration

**Part D: Google Ads Conversion Import**

- Recommended: `booking_step_3_complete`
- Why over alternatives
- Ads bidding strategy (Target CPA, conversion window)

**Part E: Implementation Checklist**

- Frontend dev responsibilities (dataLayer pushes)
- GTM setup responsibilities
- Data quality validation function

## Critical Implementation Notes

### The Interview "Trap" Question

**"Who writes the dataLayer push - you or the frontend dev?"**

**Correct Answer:** The frontend developer writes the dataLayer push. GTM cannot natively listen to multi-step form interactions. The frontend must:

1. Detect form submission
2. Validate data
3. Call `window.dataLayer.push(eventObject)`
4. Then proceed with form submission or page change

### Phone Number Deduplication (Task 03 Link)

⚠️ This schema MUST be paired with Task 03's HubSpot integration strategy, which handles phone-based deduplication (not email-based).

---

## How to Use This Schema

### For Performance Marketers

1. Use Part A to understand what events drive conversions
2. Use Part C & D to set up GA4 conversion tracking and Ads import
3. Use Part B to interpret booking funnel drop-off data

### For Frontend Developers

1. Focus on Part B (JSON examples)
2. Focus on Part E (implementation checklist)
3. Copy the exact JavaScript code snippets
4. Implement per the code comments

### For GTM Specialists

1. Use Part A to create event tags
2. Use Part B to create booking funnel triggers
3. Test with GTM Preview Mode before deployment
4. Reference Part E for QA checklist

---

## Validation Checklist

Before deploying GTM:

- [ ] All 18 events have corresponding GA4 event tags
- [ ] dataLayer push JSON validated (no typos, correct parameter names)
- [ ] Booking funnel triggers created (3 events)
- [ ] `booking_step_3_complete` marked as conversion
- [ ] Funnel report created in GA4 Exploration
- [ ] Google Ads conversion import configured
- [ ] Test with GTM Preview Mode (fire all events)
- [ ] Backend logging captures dataLayer pushes
- [ ] Analytics reports showing event counts within 24 hours

---

## Performance Targets

| Metric                 | Target  |
| ---------------------- | ------- |
| Events fired on submit | 100%    |
| Event latency          | <500ms  |
| GA4 processing         | <15 min |
| Funnel completion rate | >3%     |
| Step 1→2 drop-off      | <20%    |
| Step 2→3 drop-off      | <30%    |

---

**Document:** GTM_EVENT_SCHEMA.md  
**Status:** Ready for Implementation  
**Last Updated:** July 1, 2026
