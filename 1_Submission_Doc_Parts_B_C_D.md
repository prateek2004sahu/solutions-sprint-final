# Solutions Sprint – Prateek

_Part A (business audit) is not included yet; see email. Part D includes the Meta ad; the LinkedIn post will follow with Part A, since it is built on that audit._

---

# Part B – Requirement Doc: Vardhan Steelware – Phase 1 "Order Kahan Hai?"
Sources: first-call notes (12 Oct 2026), WhatsApp "Vardhan Dispatch" (3–7 Sep), Kunal's order sheet (1–7 Sep, 25 rows).

## 1. The problem in the client's words
- "Problem orders ki nahi hai. Problem yeh hai ki pata nahi chalta maal kahan atka hai. Customer phone karta hai toh mujhe Ramesh ko phone karna padta hai. Ramesh ki diary mein sab hai." – Suresh
- "Mujhe bas phone pe dekhna hai kaunsa order kahan hai… Pehle wale mein paisa doob gaya. Jo kaam kare, uska de denge. Diwali se pehle chahiye." – Suresh

In short: order status lives in Ramesh's diary and head; the owner is the human switchboard. Kunal separately asks for Amazon/Flipkart integration, AI sales prediction and a "modern" look.

## 2. What the material shows
| Client said | Material shows | Verdict |
|---|---|---|
| "60–70 orders a month" (Suresh) | 25 rows in 7 days. Removing 3 probable duplicates, 1 return and 1 balance-dispatch row ≈ 20 real orders ≈ **85/month** (pre-Diwali week, so maybe peak) | Kunal ("usse zyada") is right |
| Online "growing 30%/month" (Kunal) vs "<5%" (Suresh) | 3 online rows (2 Amazon, 1 Flipkart) ≈ ₹46k of ≈ ₹4.6 lakh ≈ **10% of value** that week | Neither matches; growth can't be tested from 1 week – ask for 6 months |
| "Payment koi problem nahi" | 2 rows on payment hold (₹26,200), Sharma Karnal, old dues ₹1.8 lakh. Yet Sharma's 1 Sep order (₹36,800) was dispatched 2 days before the hold. On 5 Sep Ramesh and Kunal ask each other "hold pe hai na?" / "Papa se pucho" for 43 min with no answer, while the customer chases "Diwali ka maal hai" | Disagrees: not a payment problem, a **decision-routing** problem (only Suresh can release; the message goes via Kunal) |
| "Thoda idhar udhar chalta hai… wastage" | Karigar Pappu: 420 kg out, 1180 pcs back, **6 kg short (1.4%)**, "next lot mein adjust" = loss carried, not booked. Kunal: audit gap 3–4% of steel | Disagrees: leakage is real. Illustration (assumption, ask for real figure): at ₹1 crore/yr steel, 3–4% = ₹3–4 lakh/yr, more than the failed ₹1.2 lakh ERP |
| "Maal banta hai, polish hota hai" | 4 of 9 pending lines wait on polish (Bansal Tope 120 pending since 3 Sep; Bhagona 60+40). The only partial dispatch (Mehra 180/300) was held by polish. Sheet has no polish field, only free-text remarks | **Polish at outside karigars is where orders stall – and nobody's system sees it** |

Data quality (why the sheet can't be the source of truth):
- 3 probable duplicate lines: Gupta Kadai 200 (1 Sep, twice, "repeat msg?"), Jain Kadhai 150 (2 & 4 Sep, "again sent on whatsapp"), Bansal Tope 120 (3 & 5 Sep). These are **₹1.07 lakh of the ₹2.37 lakh shown "pending" (45%)**. New Delhi Thali 50 dozen @610 appears on 2 and 7 Sep (repeat or duplicate?). Risk: double production/dispatch.
- Same item spelt 4 ways (Kadai/Kadhai/Kadhai No 10/Kadhai 10 no.); 4 date formats; 1 row with no party; Verma kadhai in **kg** with no rate; Kadai No.10 rate 180/185/192/240 (online); status free text ("pend", "pending", "partly disp").
- Partial dispatch creates a new row; the original stays "partly disp". Ramesh has to ask "Bhagona sheet mein do jagah likha hai?" – the diary, not the sheet, is the truth.

## 3. As-is process
1. **Order arrives** – distributor phones/WhatsApps Suresh or Kunal. (Online: Amazon/Flipkart, Kunal re-keys by hand.)
2. **Entry** – Kunal types into Google Sheet. No dedupe.
3. **Credit check** – Suresh decides by memory; tells Kunal verbally; Kunal posts in WhatsApp group.
4. **Floor instruction** – WhatsApp group "Vardhan Dispatch" (Kunal, Ramesh, Munshi ji).
5. **Make** – floor (~70 people); Ramesh keeps diary.
6. **Polish** – steel weighed out to 4–5 outside karigars, counted back (Munshi ji records).
7. **Karigar reconciliation** – weight vs pieces; shortage "adjusted in next lot".
8. **Pack & dispatch** – Ramesh; full or partial; Kunal updates sheet later.
9. **Invoice/payment** – Munshi ji, Tally.
10. **Status enquiry** – customer -> Suresh -> Ramesh -> diary -> back. (The pain loop.)

## 4. Phase 1 scope (target: usable before Diwali – confirm after call with Ramesh)
**Goal:** Suresh opens his phone and sees every open order, its stage and days stuck, without calling anyone.

**In**
1. One order entry (Kunal) using pick-lists for party/item/unit (no free text) + **duplicate warning** (same party+item+qty within 7 days).
2. Stages **per order line**: New -> Making -> At polish -> Ready -> Dispatched. Partial dispatch = qty on the same line, no new row. (Mehra 180/120 shows how an order splits; order status is derived from its lines.)
3. **Hold** with reason; only Suresh can release; visible to Ramesh and Kunal so nobody has to ask.
4. **Polish lot (lite):** karigar, kg out, date out, expected back, pcs in -> short-kg auto-calculated.
5. Ramesh's screen: today's list, 2-tap updates, Hindi labels, numbers only.
6. Three morning reports (section 7); every change logged with who/when.

**Out (and why)**
- Amazon/Flipkart integration, AI sales prediction: ~10% of one week's value and no clean history to predict from. Phase 2, once clean data exists.
- Tally integration/invoicing/ledger: Munshi ji's workflow works; dues are typed in by hand on held parties only.
- Raw-steel stock, purchasing, karigar payments: separate problem, bigger scope.
- Distributor-facing portal: Kunal wants to show distributors; Phase 1 UI will be clean but internal.
- Old data migration: start with open orders only.
- Replacing Ramesh's diary: run in parallel for 2 weeks; the diary is the safety net.

## 5. Data model
| Entity | Key fields | Relations |
|---|---|---|
| Party | id, name, city, phone, aliases[], dues_amount (manual) | 1–n Order |
| Item | id, name, size, unit (pcs/dozen/set/kg), aliases[] | 1–n OrderLine |
| Order | id, party_id, received_on, channel (phone/WhatsApp/Amazon/Flipkart), entered_by, hold_flag, hold_reason | 1–n OrderLine, 1–n StatusEvent |
| OrderLine | id, order_id, item_id, qty, unit, rate (nullable), stage, qty_dispatched | 1–n Dispatch, 1–n PolishLot |
| Dispatch | id, line_id, qty, date, by, LR/vehicle no. | n–1 OrderLine |
| Karigar | id, name, phone | 1–n PolishLot |
| PolishLot | id, karigar_id, line_id, kg_out, date_out, expected_back, pcs_in, date_in, kg_short (calc) | n–1 Karigar/OrderLine (a lot covering many lines = open question 7) |
| StatusEvent | id, order_id/line_id, from, to, by, at, note | audit trail |
| User | id, name, role, phone, PIN | role: owner/desk/floor/accounts |

## 6. Users (all on phone; desk also laptop)
| Role | Sees | Does |
|---|---|---|
| Owner – Suresh | Open-order board, holds awaiting him, polish/variance | Releases/keeps holds. No data entry. |
| Order desk – Kunal | Everything | Enters/edits orders, resolves duplicate warnings, maintains party/item lists |
| Floor/dispatch – Ramesh | Open lines grouped by stage, holds | Taps stage changes, records dispatch qty, polish out/in. Cannot edit rates. |
| Accounts – Munshi ji | Holds + dues, karigar variance | Updates dues, reviews karigar short-kg. Read-only otherwise. |
| Karigars | – (no login) | Ramesh records their lots. Reason: floor staff never used the 2023 ERP. |
Login: phone number + 4-digit PIN. Hindi/English toggle.

## 7. Three reports Suresh opens every morning
1. **Kahan atka hai** – all undispatched lines, oldest first: party, item, stage, days in stage; red if >3 days at polish.
2. **Aapka faisla chahiye** – held orders (party, value, dues, days on hold) + probable duplicates awaiting confirmation.
3. **Polish & steel** – lots out and overdue; kg short per karigar on closed lots; flag >2%.

## 8. Questions for the next call
| # | Question | Ask |
|---|---|---|
| 1 | Show us a page of the diary: what do you write at each stage? Phone type, comfort typing vs tapping? | Ramesh (on the floor) |
| 2 | How do you hear about a new order today, and what do you do if the same order comes twice? | Ramesh |
| 3 | Are Gupta 200, Jain 150, Bansal 120, New Delhi Thali 50 dozen genuine repeats or duplicates? Why do repeat messages happen? | Kunal |
| 4 | Send the latest 2 months of sheet, last year's audit report, and annual steel purchase | Kunal |
| 5 | What's the credit rule (limit/days)? Why was Sharma's 1 Sep order dispatched with ₹1.8L dues? Who may release a hold? | Suresh |
| 6 | How do you record weight out/pcs in per karigar? What is acceptable wastage %? What is Tally used for? | Munshi ji |
| 7 | Does one polish lot cover one order or several? | Ramesh + Munshi ji |
| 8 | Exactly why did the 2023 ERP fail (language, login, too many screens)? Who chose it? | Suresh + Ramesh |
| 9 | Are some items priced by kg (Verma kadhai) and are rates fixed per party? | Kunal + Suresh |
| 10 | What does "ready before Diwali" mean – which date, and what is the minimum that helps? Do Ramesh and Suresh have Android phones and floor Wi-Fi/data? | Suresh |

## 9. Biggest risk: the floor doesn't use it (repeat of 2023)
Ramesh (22 years, diary) wasn't on the call and the karigars aren't users. If he doesn't update stages, the board is empty and Suresh stops trusting it.
Reduce: (1) meet Ramesh on the floor before design is frozen; (2) every update ≤2 taps, Hindi, no typing; (3) run beside the diary for 2 weeks; (4) success metric: ≥90% of lines updated the same day; (5) Suresh publicly asks "app mein dekha?" instead of phoning Ramesh; (6) keep Phase 1 small – scope creep from Kunal's wish-list is the second risk.

---

# Part C – Handoff Note for the AE (Vardhan Steelware)

**Who decides / influences / uses**
- **Suresh** – decides and signs; also the only person who releases payment holds. Wants "phone pe dekhna".
- **Kunal** – influencer/champion, writes the wish-list (Amazon/Flipkart, AI, modern look). Will push scope.
- **Ramesh** – the real user and silent veto. Not on the call. 22 years, diary-based.
- **Munshi ji** – influence on karigar data/Tally; karigar visibility may feel like scrutiny of his work.

**Real pain + evidence**
Nobody can see where an order is (Suresh is the switchboard). Sheet shows ₹1.07 lakh of "pending" is probable duplicates; 4 of 9 pending lines wait at polish; a hold decision looped 43 min between Kunal and Ramesh while the customer chased. Hidden money: karigar steel gap of 3–4%, which Suresh calls "wastage".

**Asking vs needing**
Asking: Amazon/Flipkart, AI sales prediction, modern look. Needs: a phone order board with stage, holds and polish tracking that Ramesh actually updates.

**Objections and handling**
- *"Last ERP wasted ₹1.2 lakh."* – Small fixed scope; Ramesh tries it first; payment tied to milestones and real use, not a licence.
- *"It must be modern / Amazon / AI."* – Phase 2 once clean data exists; Phase 1 will look clean enough to show.
- *"Ramesh won't use it."* – Agree. Let him test a clickable demo on the floor.
- *"Wastage is normal."* – Don't argue; ask to look at two lots together.
- *"Need it before Diwali."* – Don't promise a date before meeting Ramesh; offer a core board first, rest after.

**Next step**
Book a floor visit/call with Suresh + Ramesh together; get latest 2 months of sheet and karigar records; send a one-page Phase 1 scope. Timeline line: "A working order board in a few weeks after we sit with Ramesh; exact date after the visit." You set price and terms.

---

# Part D – Content

## Meta ad (Hinglish)
**Why Hinglish:** owners like Suresh speak and WhatsApp in Hinglish; pure Hindi feels formal and English feels like it's meant for their son.

**Primary text:**
Customer ka phone aaya: "Mera maal kahan hai?"
Aapne Ramesh ko phone kiya. Ramesh ne diary kholi. Customer ka bharosa thoda aur kam ho gaya.

Order kahan atka hai (polish mein, hold pe, ya packing mein) ab apne phone pe dekhiye. Na laptop, na training. Aapka staff 2 tap mein update karta hai.

Chhota shuru kijiye. Kaam chale to hi aage badhiye.
Apni factory ke liye free 20-minute call book kijiye.

**Headline:** Order kahan atka hai? Phone pe dekhiye.

**CTA button:** WhatsApp karein / Book a call

**Creative concept:** Split screen. Left: a worn diary and a ringing phone with a worried owner. Right: one phone screen with three order cards: "Polish mein – 3 din", "Hold – payment", "Dispatch ho gaya ✓". Caption: "Diary se phone tak."
