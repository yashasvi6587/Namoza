# Task 01: GTM Event Schema - OrthoNow

## Complete Event Tracking Architecture for GA4 & Google Ads

---

## Executive Summary

This document defines every event OrthoNow needs to track for paid campaign optimization. The schema covers:

- **7 event categories** with 15+ distinct events
- **3-step booking form funnel** with exact dataLayer push JSON
- **GA4 conversion mapping** (which events become conversions)
- **Google Ads import strategy** (recommended conversion action)

---

## Part A: Complete GTM Event Schema

### Event Schema Table

| Event Name                    | Trigger Type              | Parameters (Min 3)                                                          | GA4 Report         | Audience         | Notes                             |
| ----------------------------- | ------------------------- | --------------------------------------------------------------------------- | ------------------ | ---------------- | --------------------------------- |
| **booking_form_start**        | Form Load (JS)            | `form_id`, `page_location`, `user_type`                                     | Funnel (Step 1)    | Re-engagement    | Fires when step 1 renders         |
| **booking_step_1_complete**   | Form Submit (Custom)      | `step_number: 1`, `clinic_location`, `specialty_selected`                   | Funnel (Step 1)    | Conversion Path  | User selects clinic + specialty   |
| **booking_step_2_complete**   | Form Submit (Custom)      | `step_number: 2`, `patient_name`, `phone_number`, `preferred_date`          | Funnel (Step 2)    | Lead Intent      | User enters contact details       |
| **booking_step_3_complete**   | Form Submit (Custom)      | `step_number: 3`, `booking_confirmation_id`, `final_clinic`, `booking_date` | Funnel (Step 3)    | Conversion       | User confirms booking             |
| **booking_form_abandon**      | Form Visibility (Exit)    | `step_abandoned`, `page_location`, `time_on_page`                           | Funnel             | Remarketing      | User leaves mid-funnel            |
| **cta_click**                 | Click Listener (JS)       | `button_text`, `button_location`, `page_section`                            | Event (Engagement) | Interaction      | 'Call Now', 'Book Now', etc.      |
| **call_now_click**            | Click Listener (JS)       | `phone_number`, `clinic_location`, `call_source`                            | Event (Engagement) | Intent           | Click-to-call button (tel: link)  |
| **whatsapp_widget_open**      | Click Listener (JS)       | `widget_location`, `page_location`, `timestamp`                             | Event (Engagement) | Intent           | Floating WhatsApp button          |
| **whatsapp_chat_initiated**   | Window Open Event (JS)    | `wa_phone_number`, `clinic_name`, `referrer_page`                           | Event (Engagement) | Lead Gen         | wa.me link opened                 |
| **pdf_download_start**        | Link Click (GTM Listener) | `pdf_name: 'patient_guide'`, `page_location`, `user_type`                   | Event (Engagement) | Lead Gen         | Download Patient Guide form shown |
| **pdf_download_gate_submit**  | Form Submit (Custom)      | `patient_name`, `patient_phone`, `pdf_title`                                | Event (Conversion) | Lead Gen         | Gated form submit                 |
| **pdf_download_complete**     | File Download (Custom)    | `pdf_name`, `file_size_mb`, `download_time_ms`                              | Event (Conversion) | Lead Gen         | Download completed                |
| **clinic_location_page_view** | Page View (GA4)           | `clinic_name`, `clinic_location`, `clinic_id`                               | Pages (Engagement) | Audience         | Visit /clinics/[clinic-name]      |
| **clinic_location_cta_click** | Click Listener (JS)       | `clinic_id`, `cta_button`, `page_location`                                  | Event (Engagement) | Location Intent  | 'Book' or 'Call' from clinic page |
| **blog_article_view**         | Page View (GA4)           | `article_title`, `article_category`, `article_id`                           | Pages (Engagement) | Content Interest | Blog page loaded                  |
| **blog_scroll_depth**         | Scroll Listener (JS)      | `article_id`, `scroll_depth_percent`, `time_reading_ms`                     | Event (Engagement) | Content Interest | 25%, 50%, 75%, 100% milestones    |
| **generic_form_submit**       | Form Submit (GTM)         | `form_name`, `form_fields_count`, `page_location`                           | Event (Conversion) | Lead Gen         | Catch-all for untagged forms      |
| **page_scroll_depth**         | Scroll Listener (JS)      | `page_location`, `scroll_depth_percent`, `scroll_time_ms`                   | Event (Engagement) | User Behavior    | Track engagement depth            |

---

## Part B: 3-Step Booking Form Funnel - Detailed Tracking

### User Journey Flow

```
Step 1: Select Clinic + Specialty
    ↓
Step 2: Enter Name/Phone/Date
    ↓
Step 3: Confirm Booking
    ↓
Conversion Fire (GA4 + Google Ads)
```

### Step 1: Clinic & Specialty Selection

**When it fires:** User selects a clinic location and specialty, clicks "Next"

**GTM Trigger:** Custom JavaScript event listener on the "Next" button

**dataLayer Push JSON:**

```json
{
  "event": "booking_step_1_complete",
  "step_number": 1,
  "step_name": "location_specialty_selected",
  "clinic_location": "Bengaluru - Indiranagar",
  "clinic_id": "clinic_001",
  "specialty": "Knee Pain",
  "specialty_id": "spec_knee",
  "form_id": "booking_form_main",
  "timestamp": "2026-07-01T14:23:45Z",
  "page_location": "https://orthotnow.com/book"
}
```

**Where in code:** Fire immediately after form.step1_data.validate() returns true

```javascript
if (step1Data.isValid) {
  window.dataLayer = window.dataLayer || [];
  window.dataLayer.push({
    event: "booking_step_1_complete",
    step_number: 1,
    clinic_location: step1Data.clinic.name,
    clinic_id: step1Data.clinic.id,
    specialty: step1Data.specialty.name,
    specialty_id: step1Data.specialty.id,
    timestamp: new Date().toISOString(),
  });
}
```

---

### Step 2: Contact Information & Preferred Date

**When it fires:** User enters name, phone, and preferred date, clicks "Next"

**GTM Trigger:** Custom JavaScript event listener on Step 2 "Next" button

**dataLayer Push JSON:**

```json
{
  "event": "booking_step_2_complete",
  "step_number": 2,
  "step_name": "contact_details_collected",
  "patient_name": "Rahul Sharma",
  "phone_number": "+91-9876543210",
  "preferred_date": "2026-07-15",
  "preferred_time_slot": "10:00-11:00",
  "clinic_location": "Bengaluru - Indiranagar",
  "clinic_id": "clinic_001",
  "specialty": "Knee Pain",
  "form_id": "booking_form_main",
  "timestamp": "2026-07-01T14:25:10Z",
  "page_location": "https://orthotnow.com/book"
}
```

**Where in code:**

```javascript
if (step2Data.isValid) {
  window.dataLayer = window.dataLayer || [];
  window.dataLayer.push({
    event: "booking_step_2_complete",
    step_number: 2,
    patient_name: step2Data.name,
    phone_number: step2Data.phone,
    preferred_date: step2Data.date,
    preferred_time_slot: step2Data.timeSlot,
    clinic_location: step1Data.clinic.name,
    clinic_id: step1Data.clinic.id,
    specialty: step1Data.specialty.name,
    timestamp: new Date().toISOString(),
  });
}
```

---

### Step 3: Booking Confirmation

**When it fires:** User reviews and confirms booking, clicks "Confirm Booking"

**GTM Trigger:** Custom JavaScript event listener on "Confirm" button (also fires conversion)

**dataLayer Push JSON:**

```json
{
  "event": "booking_step_3_complete",
  "step_number": 3,
  "step_name": "booking_confirmed",
  "booking_confirmation_id": "BK_20260701_00342",
  "patient_name": "Rahul Sharma",
  "phone_number": "+91-9876543210",
  "clinic_location": "Bengaluru - Indiranagar",
  "clinic_id": "clinic_001",
  "specialty": "Knee Pain",
  "final_booking_date": "2026-07-15",
  "final_booking_time": "10:00",
  "utm_source": "google",
  "utm_medium": "cpc",
  "utm_campaign": "ortho_consultation_2026",
  "form_id": "booking_form_main",
  "timestamp": "2026-07-01T14:27:33Z",
  "page_location": "https://orthotnow.com/book"
}
```

**Where in code:**

```javascript
if (step3Data.isValid && userConfirmed) {
  window.dataLayer = window.dataLayer || [];

  // Fire the conversion
  window.dataLayer.push({
    "event": "booking_step_3_complete",
    "step_number": 3,
    "booking_confirmation_id": generateConfirmationID(),
    "patient_name": step2Data.name,
    "phone_number": step2Data.phone,
    "clinic_location": step1Data.clinic.name,
    "clinic_id": step1Data.clinic.id,
    "specialty": step1Data.specialty.name,
    "final_booking_date": step2Data.date,
    "final_booking_time": step2Data.timeSlot,
    "timestamp": new Date().toISOString()
  });

  // Then show thank-you and submit to backend
  showThankYouScreen();
  submitBookingToBackend({...});
}
```

---

### Step-Level Drop-off Tracking

**GA4 Funnel Exploration Setup:**

1. Go to GA4 → Explore → Funnel Exploration
2. Create funnel with 3 steps:
   - Step 1: `booking_step_1_complete`
   - Step 2: `booking_step_2_complete`
   - Step 3: `booking_step_3_complete`

3. Segment by:
   - `clinic_location` (which clinic has highest drop-off?)
   - `specialty` (do knee pain patients drop faster than back pain?)
   - `utm_campaign` (which paid campaign has best conversion funnel?)

4. Key metrics:
   - **Step 1 → 2 drop-off %:** Contact info friction
   - **Step 2 → 3 drop-off %:** Confirmation hesitation
   - **Overall conversion rate:** Booking completion

5. **Anomaly alerts:** Set threshold at 50% drop-off at Step 2 → triggers alert to performance team

---

## Part C: GA4 Conversion Setup

### Recommended Conversions to Mark

| Event                      | Mark as Conversion? | Why                               | GA4 Event Parameter            |
| -------------------------- | ------------------- | --------------------------------- | ------------------------------ |
| `booking_step_3_complete`  | ✅ YES              | Primary business objective        | Convert event to conversion    |
| `pdf_download_gate_submit` | ✅ YES              | Qualified lead (gated content)    | Valuable for remarketing       |
| `booking_step_1_complete`  | ⚠️ OPTIONAL         | Early-funnel indicator (optional) | Use only for audience building |
| `call_now_click`           | ❌ NO               | Unmeasured offline (can't verify) | Use for audience only          |

---

## Part D: Google Ads Conversion Import Strategy

### Recommended Import: `booking_step_3_complete`

**Why this one over others:**

- ✅ **Highest intent:** Completed booking (not abandoned form)
- ✅ **Measurable offline:** Can validate in HubSpot/CRM
- ✅ **ROI traceable:** Direct link to clinic appointments
- ✅ **Campaign optimization ready:** Ads can bid on actual conversions
- ✅ **Brand safe:** No accidental clicks (real booking confirmation)

**Alternative consideration:** `pdf_download_gate_submit`

- ⚠️ Lower quality (just lead capture, not booking)
- ⚠️ Could inflate conversion count without driving revenue
- ✅ Good for top-funnel campaigns (brand awareness)

**Not recommended:** `call_now_click`

- ❌ Can't verify if call connected
- ❌ Can't track if call resulted in booking
- ❌ High fraud risk (click spam)

### Google Ads Setup

**In Google Ads → Conversions → Import from GA4:**

```
Conversion Name: OrthoNow - Consultation Booked
Event Name: booking_step_3_complete
Conversion Value: Use dynamic (clinic_id + specialty for segmentation)
Attribution Model: First-click (paid campaign gets full credit)
Conversion Window: 30 days (industry standard for appointment bookings)
Mobile-optimized: Yes (most bookings on mobile)
```

**Campaign Optimization:**

```
Smart Bidding: Target CPA = INR 800–1200 (depends on revenue per booking)
Conversion Rate Expected: 3–5% (landing page improvement)
Monthly Conversions Target: 200–250 from paid (scale from 2.1% → 5%+)
```

---

## Part E: Implementation Checklist for Frontend Dev

**Your brief to the frontend team:**

### What You Need to Do (Frontend Dev)

- [ ] Add `window.dataLayer = []` initialization to page HEAD
- [ ] Create `logBookingEvent(step, data)` function
- [ ] Call function on each step form submission (before page navigation)
- [ ] Validate form data before pushing to dataLayer
- [ ] Add try-catch wrapper (don't break form if GTM fails)
- [ ] Test in browser console: `window.dataLayer` should show array of events
- [ ] Verify timestamps are ISO-8601 format
- [ ] Confirm phone numbers include country code (+91)

### What You Need to Do (GTM)

- [ ] Create GA4 event tags for each dataLayer event
- [ ] Map parameters to GA4 event properties
- [ ] Mark `booking_step_3_complete` as conversion
- [ ] Create funnel report in GA4 Exploration
- [ ] Test with GTM Preview Mode
- [ ] Set up Google Ads import (once GA4 fires conversions)

---

## Part F: Data Quality Rules

### Required Parameter Validation

- **Phone number:** Must include country code (+91), 10 digits after +91
- **Date:** ISO-8601 format (YYYY-MM-DD)
- **clinic_id:** Must match backend clinic table
- **specialty_id:** Predefined list only (knee, back, shoulder, etc.)

### Example Validation Function

```javascript
function validateBookingEvent(eventData) {
  const errors = [];

  if (!eventData.clinic_id || !eventData.clinic_id.match(/^clinic_\d+$/)) {
    errors.push("Invalid clinic_id format");
  }

  if (
    !eventData.phone_number ||
    !eventData.phone_number.match(/^\+91\d{10}$/)
  ) {
    errors.push("Invalid phone format (must be +91XXXXXXXXXX)");
  }

  if (
    !eventData.preferred_date ||
    !eventData.preferred_date.match(/^\d{4}-\d{2}-\d{2}$/)
  ) {
    errors.push("Invalid date format (must be YYYY-MM-DD)");
  }

  return { isValid: errors.length === 0, errors };
}
```

---

## Summary

| Aspect                  | Count                                     |
| ----------------------- | ----------------------------------------- |
| Total Events            | 18 events                                 |
| Conversion Events (GA4) | 2 primary + 1 optional                    |
| 3-Step Funnel Events    | 3 events + 1 abandon                      |
| Parameters per Event    | 3–7 (minimum 3)                           |
| GA4 Reports             | 5+ (Funnel, Pages, Events, Audiences)     |
| Google Ads Conversions  | 1 recommended (`booking_step_3_complete`) |

---

## Appendix: Full Event dataLayer Reference

### Complete dataLayer Push Template

```javascript
// Initialize dataLayer
window.dataLayer = window.dataLayer || [];

// Push any event
window.dataLayer.push({
  event: "[event_name]",
  step_number: [number], // If applicable
  patient_name: "[name]", // If applicable
  phone_number: "[+91XXXXXXXXXX]",
  clinic_id: "[clinic_001]",
  specialty: "[Knee|Back|Shoulder|etc]",
  timestamp: new Date().toISOString(),
  page_location: window.location.href,
});
```

---

**Document Version:** 1.0  
**Last Updated:** July 1, 2026  
**Status:** Ready for Implementation
