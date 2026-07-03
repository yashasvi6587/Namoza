# Task 03: Integration Architecture - Landing Page to CRM & WhatsApp

## End-to-End Integration Design for OrthoNow Consultation Lead Gen

---

## Architecture Overview

### System Flow Diagram

```
Landing Page Form Submit
    ↓
Google Analytics 4 (GA4)
    ↓
Google Ads Conversion Import
    ↓
Webhook → Make/Zapier Automation
    ↓
HubSpot CRM (with phone deduplication)
    ↓
WhatsApp Message (via Karix API)
```

---

## Part A: End-to-End Architecture - Technology Stack

### Step 1: Landing Page Form Submit → DataLayer Push

**Technology:** Vanilla JavaScript  
**Method:** `window.dataLayer.push()` on form submit

```json
{
  "event": "consultation_form_submitted",
  "patient_name": "Rahul Sharma",
  "phone_number": "+91-9876543210",
  "form_id": "consultation_booking",
  "timestamp": "2026-07-01T14:30:00Z"
}
```

**Why:** No latency, no external dependency - fires immediately in user's browser.

---

### Step 2: GA4 Event Collection

**Technology:** Google Analytics 4  
**Configuration:**

- Event: `consultation_form_submitted` → Auto-converted to GA4 event
- Parameters: `patient_name`, `phone_number`
- Conversion: Marked as conversion in GA4 admin panel
- Storage: GA4 360 for 14-month retention

**Why:** GA4 is the source of truth for all conversions. Auditable, historical, integrates with Google Ads natively.

---

### Step 3: Google Ads Conversion Import

**Technology:** Google Ads → GA4 Integration  
**Setup:** Admin panel → Linked products → GA4 property → Import conversion `consultation_form_submitted`

**Configuration:**

```
Conversion Name: OrthoNow - Consultation Booked
Event Name: consultation_form_submitted
Conversion Value: Dynamic (INR 0 for lead, will map to CRM later)
Conversion Window: 30 days
Conversion counting: Every conversion (don't allow multiple)
```

**Why:** Allows paid campaign to optimize toward actual form fills. Campaign manager sees ROI in real-time.

**Latency:** ~15 minutes (GA4 to Ads sync window)

---

### Step 4: Automation Layer - Make.com (Formerly Integromat)

**Technology:** Make Webhook Trigger + Conditional Router  
**Trigger:** GA4 → Make Webhook (real-time)

**Process:**

1. GA4 fires conversion event → Make webhook receives payload
2. Extract: `patient_name`, `phone_number`, `utm_campaign`, `utm_source`
3. **Phone Deduplication Check** (CRITICAL - see Part B)
4. Route to HubSpot API + Karix WhatsApp API in parallel

**Why Make over Zapier?**

- ✅ Cheaper ($15/month vs $30+)
- ✅ Better conditional logic (necessary for phone dedup)
- ✅ Supports parallel API calls (HubSpot + Karix simultaneously)
- ✅ More flexible error handling + retry logic

---

### Step 5: HubSpot CRM - Contact Creation/Update

**Technology:** HubSpot CRM API v3 - Contacts Endpoint  
**Method:** Direct API call from Make (not native form embed)

**Why direct API over native embed:**

- ✅ Controls deduplication logic (phone-based, not email-based)
- ✅ Can pre-populate `Source` field (`Google Ads - Consultation Landing Page`)
- ✅ Can set `Lead Status` to `New Enquiry` programmatically
- ✅ Doesn't require form embedded on landing page (cleaner UX)

**API Payload:**

```json
{
  "properties": {
    "firstname": "Rahul",
    "lastname": "Sharma",
    "phone": "9876543210",
    "clinic_preference": "Bengaluru - Indiranagar",
    "hs_lead_status": "New Enquiry",
    "source": "Google Ads - Consultation Landing Page",
    "utm_campaign": "ortho_consultation_2026",
    "utm_source": "google"
  },
  "searchProperty": "phone",
  "deduplicationStrategy": "CONTACT_ONLY"
}
```

**Deduplication:** Search by `phone` field. If exists → UPDATE. If not → CREATE.

---

### Step 6: WhatsApp Message Delivery (Karix)

**Technology:** Karix WhatsApp Business API  
**Method:** Parallel API call alongside HubSpot

**WhatsApp Message:**

```
"Hi Rahul! 👋 We've received your consultation request.
A specialist will confirm your appointment at Bengaluru - Indiranagar
within 2 hours. Reply or call us anytime. Thank you, OrthoNow Team"
```

**API Payload:**

```json
{
  "message_type": "text",
  "to": "+919876543210",
  "body": "Hi Rahul! 👋 We've received your consultation request...",
  "template_id": "ortho_consultation_confirm",
  "timestamp": "2026-07-01T14:30:15Z"
}
```

**SLA:** Must send within 2 minutes of form submit (see Part C for monitoring).

---

## Part B: Critical Failure Point - Phone Number Deduplication

### The Problem

**Default HubSpot deduplication:** Email-based (NOT phone-based)

- If two patients submit with **different names** but **same phone number**, HubSpot treats them as separate contacts
- Creates duplicate leads → Double WhatsApp messages → Wasted campaign spend
- Especially problematic in India where family members may share phone numbers

### Why This Breaks

```
First submission: Rahul Sharma, +919876543210 → Contact created
Second submission: Priya Sharma (wife), +919876543210 → NEW CONTACT CREATED (WRONG!)
Result: Same phone gets 2 WhatsApp messages, CRM misattributes revenue
```

### Solution: Phone-Based Deduplication

**Implementation:** Use HubSpot's contact deduplication API parameter

**In Make workflow:**

```
1. Before pushing to HubSpot, query existing contacts by phone
   GET /crm/v3/objects/contacts?limit=1&after=0&filterGroups=[
     { "filters": [{"propertyName": "phone", "operator": "EQ", "value": "+919876543210"}] }
   ]

2. If contact exists:
   - Return contact ID
   - Update only fields: clinic_preference, lead_status, source, utm_campaign
   - SKIP WhatsApp send (don't re-message existing customer)

3. If no contact exists:
   - Create new contact
   - Send WhatsApp welcome message
```

**Fallback:** If phone format invalid or missing → PAUSE workflow, alert ops team. Don't create contact.

---

## Part C: WhatsApp SLA Monitoring (2-Minute Delivery Window)

### What Could Break the SLA

| Failure Mode               | Probability | Monitoring                | Mitigation                                 |
| -------------------------- | ----------- | ------------------------- | ------------------------------------------ |
| Make webhook delayed       | MEDIUM      | Check Make execution logs | Set webhook timeout to 30s, retry 3x       |
| HubSpot API timeout        | LOW         | HTTP 504 logs             | Queue message even if HubSpot create fails |
| Karix API throttled        | LOW         | Karix rate limit headers  | Use connection pooling, batch queue        |
| Phone format invalid       | MEDIUM      | Karix error response      | Validate regex before sending              |
| Network latency spike      | LOW         | End-to-end timing log     | Use CDN for webhook ingestion              |
| WhatsApp template approval | FIXED       | Manual check              | Pre-approve templates before campaign      |

### SLA Monitoring Strategy

**Real-time Logging:**

```javascript
// In Make automation:
{
  "timestamp_form_submit": "2026-07-01T14:30:00Z",
  "timestamp_make_trigger": "2026-07-01T14:30:01Z",
  "timestamp_hubspot_create": "2026-07-01T14:30:04Z",
  "timestamp_whatsapp_send": "2026-07-01T14:30:06Z",
  "timestamp_whatsapp_delivered": "2026-07-01T14:30:08Z",
  "total_sla_time_seconds": 8,
  "sla_status": "PASS" // if < 120 seconds
}
```

**Alert Thresholds:**

- ⚠️ WARNING: Time-to-WhatsApp > 60 seconds → Slack alert to ops
- 🔴 CRITICAL: Time-to-WhatsApp > 120 seconds → Page on-call engineer
- 🟢 PASS: Time-to-WhatsApp < 120 seconds → Log to dashboard

**Dashboard (Google Sheet + Apps Script):**

- Auto-refresh Make logs every 5 minutes
- SLA compliance %: (Passes / Total) × 100
- Target: 99% of messages within 120 seconds
- Weekly report to performance team

**Escalation:**

- If SLA drops below 95% → Investigate Make/Karix capacity
- If Karix is bottleneck → Increase connection limit or add failover provider (Twilio)
- If Make is slow → Add webhook retry queue to landing page (send async)

---

## Part D: Data Flow Summary

| Component      | Technology        | Latency           | Failure Risk        |
| -------------- | ----------------- | ----------------- | ------------------- |
| Form → GA4     | JS dataLayer      | ~100ms            | Low                 |
| GA4 → Ads      | GA4 API sync      | 15 min            | Low (batch)         |
| GA4 → Make     | Webhook           | 2-5 sec           | Medium (timeout)    |
| Make → HubSpot | REST API          | 1-2 sec           | Low (retry logic)   |
| Make → Karix   | REST API          | 2-3 sec           | Medium (rate limit) |
| **Total E2E**  | **~5-10 seconds** | **SLA: <120 sec** | Medium              |

---

## Part E: Deployment Checklist

- [ ] HubSpot: Create custom field `clinic_preference` (text)
- [ ] HubSpot: Create custom field `source` (dropdown: "Google Ads - Consultation Landing Page")
- [ ] HubSpot: Create custom field `lead_status` (dropdown: includes "New Enquiry")
- [ ] Make: Set up GA4 webhook trigger (requires GA4 export to BigQuery first)
- [ ] Make: Create conditional router for phone deduplication logic
- [ ] Make: Set up HubSpot contact create/update module
- [ ] Make: Set up Karix WhatsApp send module
- [ ] Karix: Approve WhatsApp template ("Hi [Name]! We've received...")
- [ ] Google Ads: Import GA4 conversion `consultation_form_submitted`
- [ ] Monitoring: Set up Slack alerts for SLA breaches
- [ ] Testing: Send 5 test submissions, verify HubSpot + WhatsApp delivery within 2 min

---

## Appendix: Why Not Other Approaches?

### Why not Zapier?

❌ More expensive (limited free tier)  
❌ Slower conditional logic  
❌ Can't do parallel API calls efficiently

### Why not native HubSpot form embed?

❌ Email-only deduplication (ruins phone-based CRM strategy)  
❌ Can't customize Source field dynamically  
❌ Can't control WhatsApp send independently

### Why not direct form → HubSpot (no GA4)?

❌ Breaks analytics pipeline  
❌ Can't optimize Google Ads toward conversion  
❌ No audit trail in GA4

---

**Document Version:** 1.0  
**Last Updated:** July 1, 2026  
**Status:** Ready for Development
