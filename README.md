# ☎️ Sofia: Autonomous Inbound Telephony Agent for Luxury Salons

![Retell AI](https://img.shields.io/badge/Retell_AI-2B1A4A?style=for-the-badge&logo=openai&logoColor=white)
![n8n](https://img.shields.io/badge/n8n-FF6C37?style=for-the-badge&logo=n8n&logoColor=white)
![Twilio](https://img.shields.io/badge/Twilio-F22F46?style=for-the-badge&logo=Twilio&logoColor=white)
![Google Calendar](https://img.shields.io/badge/Google_Calendar-4285F4?style=for-the-badge&logo=google-calendar&logoColor=white)

> An event-driven, single-prompt conversational voice agent deployed for **LUX SALON (Miami, FL)** capable of handling concurrent customer bookings, calendar math, and live rescheduling over standard phone lines.
<img width="1536" height="1024" alt="WhatsApp Image 2026-06-21 at 4 08 56 PM" src="https://github.com/user-attachments/assets/bc94932c-b84c-4065-9b75-c7005564e726" />



---

## Executive Summary

Traditional salons lose up to 35% of their potential booking revenue due to unanswered calls during peak service hours. **Sofia** solves this by acting as an autonomous, 24/7 SIP-trunked receptionist. 

Unlike basic "press 1 for hours" IVR trees, Sofia utilizes a **Single-Prompt LLM architecture** hooked into an event-driven `n8n` orchestration layer. She holds natural, turn-taking conversations, performs real-time calendar availability math against variable service durations, and writes directly to the business's Google Calendar.
<img width="1600" height="900" alt="WhatsApp Image 2026-06-21 at 4 03 36 PM (1)" src="https://github.com/user-attachments/assets/2991fb63-b7af-488d-b894-48cac1636323" />
---

## 🏛️ System Architecture
# Voice Agent — Salon Booking (US Client)


**Stack:** Retell AI (voice) + n8n (orchestration) + [Client CRM]

## The Problem
The salon was losing bookings because callers got voicemail during peak hours.

## The Solution
Inbound AI voice agent that handles appointment booking 24/7.

## Call Flow
[Caller] → [Retell AI] → [n8n webhook] → [Booking API] → [SMS Confirmation]

## Results
- Handled X calls/day (put your real number or estimate)
- Reduced no-show rate by ~Y% (if you know it; if not, say "measured via booking completion rate")
- Average call duration: Z minutes

<img width="521" height="167" alt="Screenshot 2026-06-21 160156" src="https://github.com/user-attachments/assets/4dbdb1e4-5479-4800-9f08-9c4eb82a3c69" />


## What I learned
- Retell prompt engineering for natural conversation flow
- n8n error handling when CRM API is down
- Multilingual considerations (if any)
```text
 [ Miami Caller ] ──(PSTN / VoIP)──► [ Twilio SIP Trunk ]
                                            │
                                            ▼
                                  ┌───────────────────┐
                                  │ Retell AI Engine  │◄── (Single Prompt Persona)
                                  └─────────┬─────────┘
                                            │
                                  (Custom Tool Call Fired)
                                            │
                                            ▼
                                  ┌───────────────────┐
                                  │   n8n Webhook     │
                                  │ (ID: HO8OEQl...)  │
                                  └─────────┬─────────┘
                                            │
                     ┌──────────────────────┴──────────────────────┐
                     ▼                                             ▼
          [ Tool: check_availability ]                 [ Tool: book_appointment ]
                     │                                             │
         (Fetch events & subtract logic)                  (POST New Event & Contact)
                     │                                             │
                     ▼                                             ▼
          [ Google Calendar API ]                       [ Google Calendar + Gmail ]
```

<img width="1600" height="900" alt="WhatsApp Image 2026-06-21 at 4 03 37 PM" src="https://github.com/user-attachments/assets/b1cec401-a883-4d4d-91a7-1d21531f9028" />
<img width="589" height="521" alt="WhatsApp Image 2026-06-21 at 4 03 37 PM (1)" src="https://github.com/user-attachments/assets/fa9e427c-2951-4c3c-b81f-9a5133b16613" />

### The Service Duration Matrix
Sofia relies on an internal lookup table to prevent double-booking. When a user requests a slot, the system checks the calendar for an open window matching the exact duration of the requested service:

| Service | Allocated Duration | Internal Key |
| :--- | :--- | :--- |
| **Haircut** | 45 Minutes | `haircut` |
| **Blow-dry** | 30 Minutes | `blowdry` |
| **Full Colour** | 120 Minutes | `full_colour` |
| **Highlights** | 90 Minutes | `highlights` |
| **Treatment** | 45 Minutes | `treatment` |
| **Facial** | 60 Minutes | `facial` |
| **Bridal Package**| 150 Minutes | `bridal` |

---

## 🛠️ Tool Calling Specifications (Sofia ➔ n8n)

When the LLM identifies a hard intent, it halts standard text generation and emits a structured JSON payload to the n8n webhook endpoint. 

#### 1. `check_availability`
Emitted when the caller requests an open time.
```json
{
  "tool_name": "check_availability",
  "payload": {
    "service": "highlights",
    "preferred_date": "2026-06-25",
    "preferred_time_of_day": "afternoon"
  }
}
```

#### 2. `book_appointment`
Emitted once the caller verbally agrees to a proposed timestamp.
```json
{
  "tool_name": "book_appointment",
  "payload": {
    "caller_name": "Sarah Jenkins",
    "caller_phone": "+13055558912",
    "caller_email": "sarah.j@email.com",
    "service": "highlights",
    "confirmed_datetime": "2026-06-25T14:00:00-04:00"
  }
}
```

#### 3. `cancel_or_reschedule`
Emitted when matching an existing phone number to an active calendar block.

---

## 📞 Verified Production Call Transcript 

*Target Latency: < 850ms*

| Speaker | Live Audio Transcript | Sub-system Trigger |
| :--- | :--- | :--- |
| **Sofia** | "Thanks for calling Lux Salon Miami, my name is Sofia! How can I help you look fabulous today?" | `Twilio Stream Active` |
| **Caller**| "Hey Sofia, I need to get a Full Colour done sometime this Friday morning." | *Entity extraction: Full Colour (120m)* |
| **Sofia** | "I'd love to set that up for you. Let me look at our Friday morning schedule right now..." | `Emit: check_availability` |
| **System**| *(n8n queries G-Cal, locates a 2-hour gap between 9:00 AM and 11:00 AM)* | `Return: ["09:00:00"]` |
| **Sofia** | "We have a 9:00 AM slot open that will fit the two hours needed for a Full Colour. Does that work?" | *Contextual presentation* |
| **Caller**| "Yeah 9 AM is great. My name is Jessica Alba, phone is 305-555-0199." | *Verbal commitment* |
| **Sofia** | "Got it, Jessica. You're locked in for Friday at 9 AM. I'll text and email the confirmation right now!" | `Emit: book_appointment` |

---

## 🧠 Production Edge-Cases & Bug Fixes

1. **The Empty-Calendar Crash:** * *The Issue:* If n8n queried Google Calendar for a day with zero existing appointments, the Google API returned `undefined` rather than an empty array `[]`, causing the downstream JavaScript `.map()` function to throw a fatal unhandled exception.
   * *The Fix:* Injected a custom **Normalize Calendar Output** code node to strictly type-cast all payload responses to `Array.isArray(data) ? data : []`.

2. **The `call_ended` Race Condition:**
   * *The Issue:* Initially bound the n8n execution trigger to Retell's `call_ended` event. If the user hung up the phone instantly after saying "Yes book it", the socket closed before the final LLM transcript chunk generated, resulting in lost bookings.
   * *The Fix:* Migrated the orchestration listener to `call_analyzed`, forcing the webhook to hold execution until the post-call NLP processing object reported `status: "complete"`.

3. **Miami DST (Daylight Savings) Drift:**
   * *The Issue:* Standard UTC ISO strings were booking clients 1 hour off during the Spring offset shift. 
   * *The Fix:* Hardcoded the n8n environment runtime strictly to `America/New_York` with an explicit Luxon `.setZone()` override on all incoming JSON payloads.
