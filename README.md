# PayFlow · UPI prototype

An interactive, single-file simulation of a UPI payment. It shows what really happens between tapping **Pay** on a phone and seeing "Payment successful": the DNS lookup, the TCP and TLS handshakes, authentication, firewall checks, database writes, the NPCI switch, the two banks, and what happens when something goes wrong along the way.

Everything runs in the browser. No real payments, banks, servers or network calls are involved.

---

## Table of contents

1. [Overview](#overview)
2. [Highlights](#highlights)
3. [Run it](#run-it)
4. [Quick start (60 seconds)](#quick-start-60-seconds)
5. [Screen layout](#screen-layout)
6. [The phone app](#the-phone-app)
7. [The inspector](#the-inspector)
8. [Scenarios](#scenarios)
9. [Guided walkthroughs](#guided-walkthroughs)
10. [A payment, step by step](#a-payment-step-by-step)
11. [Simulated API](#simulated-api)
12. [Concepts it demonstrates](#concepts-it-demonstrates)
13. [Controls and shortcuts](#controls-and-shortcuts)
14. [Validation rules and demo data](#validation-rules-and-demo-data)
15. [How the code is organised](#how-the-code-is-organised)
16. [Customising the prototype](#customising-the-prototype)
17. [Design and accessibility](#design-and-accessibility)
18. [Browser support](#browser-support)
19. [Testing checklist](#testing-checklist)
20. [Troubleshooting](#troubleshooting)
21. [FAQ](#faq)
22. [Glossary](#glossary)
23. [Notes and limits](#notes-and-limits)
24. [Ideas for future work](#ideas-for-future-work)
25. [Contributing](#contributing)
26. [License](#license)
27. [Project structure](#project-structure)

---

## Overview

Most people only see a spinner and then a green tick. PayFlow makes the invisible parts visible. You use a realistic mock UPI app on the left, and on the right an inspector narrates every event behind it, tagged by network layer, with the actual request and response text, while a packet travels across a diagram of the real payment path.

It was built to explain, in a hands-on way, how a payment can be fast, secure and safe against failure: why a double tap doesn't charge you twice, why a dropped signal doesn't lose your money, and how an attacker or a malicious input gets stopped.

**Who it is for**

- Students learning networking, web security or payment systems
- Developers who want a concrete picture of idempotency, retries and reversals
- Anyone presenting or teaching how digital payments work, who needs something visual to click through
- Product and design teams prototyping the user-facing states of a payment flow (pending, failed, refunded)

## Highlights

- A full mock UPI app: register, home, send money, PIN entry and result screens
- A live network diagram with an animated packet that can be lost, blocked or tampered with
- A layer-by-layer event log (App, DNS, TLS, TCP, IP, Link, UPI, Security) with real-looking request and response text
- Six one-click scenarios covering success and the most important failure modes
- Live server counters, a transactions table and both bank ledgers
- Step-through mode, three speeds and keyboard control, so it works for self-study and for presenting
- A single `index.html` file: no build, no framework, no dependencies
- Light and dark themes, responsive layout, and accessibility support

## Run it

No build step and no dependencies. Open `index.html` in a browser, or serve the folder:

```bash
python3 -m http.server 8000
# then visit http://localhost:8000
```

The only external request is Google Fonts (IBM Plex Sans and IBM Plex Mono). If you are offline, it falls back to system fonts.

## Quick start (60 seconds)

1. Open the page and click **Skip with a demo account** on the phone.
2. Leave **Normal payment** selected in the scenario card.
3. Click **Send money**, then **Continue** (the form is pre-filled).
4. Type the PIN `1234` on the keypad.
5. Watch the packet travel across the diagram and read the log as each step appears.
6. Press **Back to home**, pick another scenario such as **Double tap**, and repeat.

For a slower, more careful look, tick **Step through** before you pay and press **Space** to advance one event at a time.

## Screen layout

| Area | Contents |
|---|---|
| Header | Logo, title, a short note that nothing here is real, and a **Reset demo** button |
| Left column | The phone mock-up (stays in view while you scroll on large screens) |
| Top right | **Path of a payment**, an animated network diagram |
| Middle right | **What happens, step by step**, the live event log and its controls |
| Side cards | **Demo scenario** picker, and **Server and bank state** |

On narrow screens the layout stacks into a single column.

## The phone app

A five-screen mock UPI app with a status bar (live clock, and a network indicator that changes from **5G** to **No signal** when the connection drops).

1. **Register:** full name, 10-digit mobile number and a bank (SBI, HDFC, ICICI or Axis). The app derives a UPI address from your first name and bank, for example `arpit@sbi`. A note explains the SIM is confirmed by SMS (device binding). A **Skip with a demo account** link jumps straight to the home screen.
2. **Home:** a greeting, your VPA, account balance, a **Send money** button, and a list of your last six payments with colour-coded status labels.
3. **Send money:** payee VPA, amount in rupees, and an optional remark (up to 40 characters), with inline validation.
4. **Enter UPI PIN:** a custom 4-digit keypad with dots that fill in as you type. The demo PIN is `1234`; any other PIN is declined by the bank.
5. **Result:** a status icon, title and message, plus a details panel (transaction ID, payee, amount, time and any note). A **Back to home** button appears once the payment is final.

A toast notification slides down on completion, and a "No internet connection" banner appears on the result screen when the signal drops.

### Transaction states

`INITIATED` → `PENDING` → `SUCCESS` or `FAILED`

| State | Icon | Meaning on the phone |
|---|---|---|
| `INITIATED` | Spinner | The app is opening a connection and sending the request |
| `PENDING` | Clock | The outcome is not known yet. The user is told not to pay again |
| `SUCCESS` | Tick | The money moved and the payee was credited |
| `FAILED` | Cross | The payment did not complete. The message says whether money was debited and returned, or never debited |

The phone and the server track their state separately, so you can see moments when they disagree (for example, the server already says `SUCCESS` while the phone still shows `PENDING` after a dropped signal).

## The inspector

### Path of a payment

An animated diagram with six nodes:

**Phone → Internet → PayFlow server (port 443) → NPCI (UPI switch) → Payer bank / Beneficiary bank**

A packet moves hop by hop along the route, labelled with what it carries (`SYN`, `Hello`, `POST`, `ReqPay`, `Debit?`, `200 OK` and so on). Nodes highlight as the packet passes. Packets can be lost mid-route (shown with a red ✕), the phone-to-internet link turns red and dashed when the signal drops, and an **attacker** badge appears on the internet node in the tampering scenario.

### Step-by-step log

Every event is a row with a layer chip, a title, a detail block in monospace, and a timestamp. The chips are colour-coded by layer:

| Layer | Meaning |
|---|---|
| App | HTTP requests and responses, server logic, database actions |
| DNS | Name lookups and caching |
| TLS | Encryption and certificate handling |
| TCP | Connection set-up and reliable delivery |
| IP | Addressing and routing |
| Link | Wi-Fi or 5G delivery, signal loss |
| UPI | Messages between the server, NPCI and the banks |
| Security | Device binding, tokens, firewall and rate limiting |

Rows are marked green when something goes right (a connection is established, a reversal succeeds) and red when something fails or is attacked. Each run starts with a separator line naming the scenario, so runs are easy to tell apart. Timestamps are simulated: they advance by small random amounts per event, and jump by whole seconds where the app waits (for example before a retry).

### Server and bank state

- **Counters:** requests received and payments processed. These show, for example, that a double tap sends two requests but processes only one payment.
- **Server database table:** the last five transactions with ID, payee, amount and status, so you can watch a row appear as `INITIATED` and move through its states.
- **Bank ledgers:** live balances for the payer and the payee. The payer's balance drops when the payment is debited, the payee's rises when it is credited, and a row flashes whenever a value changes. A payee gets a random starting balance the first time you pay them, and keeps its balance across later payments.

## Scenarios

Pick one before sending a payment. The picker locks while a payment is in progress.

| Scenario | What happens | What it demonstrates |
|---|---|---|
| **Normal payment** | Phone to server to NPCI to both banks, and back | The full happy path |
| **Network drops after paying** | The bank debits and the server saves SUCCESS, but the reply is lost and the phone loses signal | The app must not assume failure. It shows "pending", retries the status check with exponential backoff (1 s, 2 s), reconnects with TLS session resumption, and finds out the payment succeeded, with no double debit |
| **Double tap** | The same request arrives a second time with the same idempotency key | The `UNIQUE(txn_id)` rule in the database ignores the duplicate. Only one payment is processed and the result shows a note that you tapped twice |
| **Beneficiary bank is down** | The payer is debited but the receiving bank never answers | The payment goes `PENDING`, NPCI re-checks twice, declares it `FAILED`, and an automatic reversal returns the money to the payer |
| **Attacker tampers with data** | An attacker flips bits in the encrypted request on the way | TLS's authentication tag fails (`bad_record_mac`), the connection closes, and the altered data never reaches the payment code |
| **SQL injection attempt** | The payee field carries a malicious SQL payload (`x' OR '1'='1`) | The firewall matches a SQL injection rule and returns `403 Forbidden`. Parameterised queries would protect the database even without the firewall |

### Other failures that happen without a special scenario

| Trigger | Result |
|---|---|
| Unknown bank handle in the payee VPA (for example `someone@unknownbank`) | NPCI cannot find the payee: **Payee VPA not found**. Nothing is debited |
| Any PIN other than `1234` | The payer bank declines: **Incorrect UPI PIN**. Nothing is debited. The log notes that a real bank locks UPI after 3 wrong tries |
| Amount larger than the balance | The payer bank declines: **Insufficient balance**. Nothing is debited |

## Guided walkthroughs

Use these to see each idea clearly. Tick **Step through** first if you want to read each event before moving on.

### 1. The happy path

1. Select **Normal payment** and send ₹500 to the pre-filled payee with PIN `1234`.
2. Watch for the connection set-up (DNS, TCP, TLS), then the `POST /api/v1/transactions` request.
3. Note that the **Payer** ledger drops by ₹500 and the **Payee** ledger rises by ₹500 at the debit and credit steps.
4. The database row goes from `INITIATED` to `PENDING` to `SUCCESS`.

### 2. Double tap and idempotency

1. Select **Double tap** and pay.
2. Find the row "Second tap: a duplicate POST arrives". **Requests received** goes up by two, but **Payments processed** only goes up by one.
3. Check the payer balance: it is debited once. The result screen adds a note that you tapped twice.

### 3. Lost reply and safe retries

1. Select **Network drops after paying** and pay.
2. After the server sends `200 OK`, the signal drops and the reply never arrives. The phone shows **Payment pending** and tells you not to pay again.
3. Watch the status polling: the first retry is lost, the signal returns, a new TCP connection is made, and TLS resumes from the earlier session. The second retry succeeds.
4. The retries are `GET` requests that only read status. The payer was debited exactly once.

### 4. Bank outage and automatic reversal

1. Select **Beneficiary bank is down** and pay.
2. The payer is debited, but the credit request times out. The phone shows **Payment pending**.
3. NPCI re-checks twice, then declares the payment failed and reverses the debit.
4. The payer balance returns to what it was. The result screen says the debited amount was returned.

### 5. Tampering

1. Select **Attacker tampers with data** and pay.
2. An **attacker** badge appears on the internet node, and the log shows bits being flipped in the encrypted record.
3. The server rejects it with a TLS `bad_record_mac` alert. **Requests received** does not increase, because the tampered request never reached the application.

### 6. SQL injection

1. Select **SQL injection attempt** and pay.
2. The log shows a malicious payee value being sent, as a hostile client would do by skipping the app's own format check.
3. The firewall blocks it with `403 Forbidden`. A request was received, but no payment was processed and no database row was created.

### 7. Failure without a scenario

1. Select **Normal payment** and enter a wrong PIN such as `0000`. Watch the payer bank decline it.
2. Try an amount above ₹25,000 to trigger **Insufficient balance**.
3. Try a payee like `test@nobank` to trigger **Payee VPA not found**.

## A payment, step by step

For a normal payment the log walks through:

1. **Connection set-up (first payment):** a link-layer frame leaves the phone; a DNS A-record query goes out over UDP and is cached with a TTL; an IP packet is addressed to the server (NAT, TTL, routing); a TCP three-way handshake runs (SYN, SYN-ACK, ACK); a TLS 1.3 handshake runs (ClientHello, certificate and key share, Finished) to produce an encrypted tunnel using `TLS_AES_256_GCM_SHA384`.
2. **The request:** `POST /api/v1/transactions` with a bearer token and an `Idempotency-Key` header, shown in readable form with a note that on the wire it is ciphertext.
3. **Server checks:** firewall (no attack pattern), token (valid signature, expiry), rate limit (for example, 1 of 5 payments this minute).
4. **Database write:** the transaction is inserted as `INITIATED`, then updated to `PENDING`.
5. **UPI leg:** the server sends `ReqPay` to NPCI; NPCI looks up both banks; the payer bank verifies the PIN (carried as an encrypted blob only the bank can open) and debits; the beneficiary bank credits; NPCI returns `RespPay SUCCESS`.
6. **Final status:** the server saves `SUCCESS` and replies `200 OK`; the phone updates; a push message and SMS are sent so the user is told even if the app is closed.

With **Skip handshake after the first request** ticked, later payments reuse the open connection (keep-alive) and skip DNS, TCP and TLS. Otherwise the DNS answer is still reused from cache, but a full TCP and TLS handshake runs again.

### Registration flow

Registering also runs the full connection set-up, then `POST /api/v1/auth/register`, an SMS one-time code for **device binding** (shown as a toast on the phone), `POST /api/v1/auth/verify`, issue of a **15-minute session token**, and linking of the VPA to the bank account.

## Simulated API

These endpoints appear in the log. They are not real: they exist only as text and logic inside the page.

| Method and path | Used for | Notable details |
|---|---|---|
| `POST /api/v1/auth/register` | Start registration | Sends name, mobile number (`+91…`) and bank |
| `POST /api/v1/auth/verify` | Confirm the SMS code | Sends the one-time code and a device ID. Returns a session token |
| `POST /api/v1/transactions` | Create a payment | Bearer token, `Idempotency-Key` equal to the transaction ID, JSON body with payer, payee, amount, remark and the encrypted PIN |
| `GET /api/v1/transactions/{id}/status` | Check a payment's status | Used after a lost reply or a pending result. It only reads, it never re-sends the payment |

**Responses shown:** `200 OK` (final status, including a declined payment), `202 Accepted` (accepted but still pending), and `403 Forbidden` (blocked by the firewall).

**Session token:** a JWT (`HS256`) whose payload holds the VPA and a 15-minute expiry, built in the page with a random signature segment.

**Transaction IDs:** `PF` followed by ten digits taken from the current time, for example `PF1730381234`.

## Concepts it demonstrates

- **Network layers:** link, IP, TCP, TLS, DNS and application, shown in the order they actually occur.
- **DNS:** why it uses UDP, and how TTL caching skips repeat lookups.
- **TCP:** the three-way handshake and what "reliable, ordered delivery" means.
- **TLS 1.3:** certificates, key exchange, AES-256-GCM encryption, and tamper detection by authentication tag.
- **Authentication:** a signed JWT (HS256) with a short expiry, and device binding through an SMS code.
- **Request protection:** firewall rules, rate limiting and input validation.
- **Idempotency:** an `Idempotency-Key` plus a `UNIQUE(txn_id)` database constraint make retries and double taps safe.
- **Handling uncertainty:** the difference between failed and unknown, polling for status instead of re-sending, and exponential backoff.
- **UPI structure:** the PayFlow server, NPCI as the switch, and separate payer and beneficiary banks, with debit, credit, re-check and reversal messages.
- **Safe failure:** automatic reversal, so a failed payment never leaves the payer out of pocket.
- **Defence in depth:** input validation in the app, a firewall in front of the server, and parameterised queries behind it, so one layer failing doesn't expose everything.

## Controls and shortcuts

| Control | What it does |
|---|---|
| **Slow / Normal / Fast** | Changes the pace of the animation and log |
| **Step through** | Pauses at every event until you continue |
| **Next step** or **Space** / **→** | Advances one event in step-through mode (ignored while typing in a text field) |
| **Skip handshake after the first request** | Reuses the open connection for later payments |
| **Reset demo** | Clears all state and returns to the register screen |
| **Skip with a demo account** | Skips registration and signs in as `arpit@sbi` |

## Validation rules and demo data

- **VPA format:** must look like `name@bank` (at least 2 characters of letters, numbers, `.`, `_` or `-` before the `@`, then letters after it).
- **Own VPA:** you cannot pay yourself.
- **Amount:** a whole number of at least ₹1, and at most ₹1,00,000 per payment (the UPI limit).
- **Mobile number:** exactly 10 digits. **Name:** at least 2 characters.
- **Known bank handles:** `sbi`, `hdfcbank`, `icici`, `axisbank`, `ybl` and `paytm`. Any other handle is reported as not found.
- **Starting balance:** ₹25,000, restored on **Reset demo**.
- **Payee balances:** a random whole-hundred amount, created the first time each payee is paid.
- **Demo PIN:** `1234`.
- **Demo account:** `Arpit Kumar Jena`, `arpit@sbi`.
- **Pre-filled values:** registration is pre-filled with a sample name and number, and the send screen with a sample payee, amount and remark, so you can click straight through.
- **Illustrative values:** the server address, domain, token lifetime text and firewall rule number shown in the log are examples, not real services.

## How the code is organised

Everything lives in `index.html`, in three parts: markup, a `<style>` block, and one `<script>` wrapped in a self-invoking function with strict mode.

**Markup:** the header, the phone with its five screens (`s-reg`, `s-home`, `s-send`, `s-pin`, `s-res`), the SVG network diagram, the log card, the scenario picker and the state card.

**Styles:** colour tokens are CSS variables on `:root`, with separate values for light and dark. Components are small and flat: `.card`, `.phone`, `.screen`, `.btn`, `.pill` (status labels), `.chip` (layer labels) and `.row` (log rows).

**Script, by section:**

| Section | Responsibility |
|---|---|
| Helpers and constants | DOM shortcuts, `sleep`, random and hex helpers, rupee formatting, the `BANKS` table, layer names, `PACE` timings and the `SCEN` list |
| State object `S` | Single source of truth: speed, step mode, user, VPA, balance, ledgers, transactions, DNS and connection flags, request and processing counters, and the current transaction |
| Pacing | `beat()` waits a fraction of the chosen pace, or waits for **Next step** in step-through mode. `advance()` releases it |
| Topology animation | `route()` finds the path between nodes with a breadth-first search over the diagram. `fly()` moves the packet hop by hop, and can drop it mid-route |
| Log | `addRow()` writes an entry. `E()` combines logging, packet movement and a pause. `sep()` adds scenario separators |
| Phone UI | Screen switching, toast, renderers for the home list, the result screen, the database table and the ledgers |
| Connection set-up | `connect()` runs the link, DNS, IP, TCP and TLS steps, or reuses the connection when allowed |
| Registration | `register()` and `makeToken()` |
| Payment | `pay()` orchestrates a payment. `bankStage()` handles NPCI and the banks. `pollStatus()` handles retries. `finish()` and `failEarly()` set final results |
| Wiring | Keypad, form validation, scenario picker, controls, keyboard shortcut, clock, `resetDemo()` and `init()` |

Because the whole run is a chain of `await`ed events, adding a new step is as simple as adding another `E(...)` call in the right place.

## Customising the prototype

All of these are small edits in `index.html`:

| To change | Where |
|---|---|
| Starting balance | `balance:25000` in the state object `S`, and again in `resetDemo()` |
| Demo PIN | The `txn.pin !== '1234'` check in `bankStage()`, and the hint text on the PIN screen |
| Per-payment limit | The `amt > 100000` check in `sendContinue()`, the `max` on the amount input, and the error message |
| Supported banks | The `BANKS` object. The registration dropdown has its own list of options |
| Animation speed | The `PACE` object (milliseconds for slow, normal and fast) |
| Scenario names and descriptions | The `SCEN` array |
| Network diagram nodes and links | The `NODE` and `ADJ` objects, plus the matching SVG elements |
| Colours and theme | The CSS variables at the top of the `<style>` block |
| Demo account | The handler for the **Skip with a demo account** button in `init()` |

### Adding a new scenario

1. Add an entry to the `SCEN` array with an `id`, a title and a description.
2. Handle the new `id` in `pay()` (before or after the request), or in `bankStage()` if it happens at the bank.
3. Use `E(layer, title, detail, [from, to, label], kind)` for each event. Use `kind` of `'good'` or `'bad'` to colour the row.
4. Finish with `setPhone(...)` and `finish(...)`, or `failEarly(...)` if the payment never reaches the server.

## Design and accessibility

- Light and dark themes follow the system setting, with a full colour palette for both.
- Responsive layout, including safe-area handling for notched phones.
- Keyboard accessible: visible focus rings, labelled keypad buttons, and Space / → for stepping.
- Screen-reader friendly: status messages and errors use live regions, and the network diagram has a text description.
- Respects the reduced-motion setting by turning off animations.
- Status is never shown by colour alone; every state also has a text label and an icon.
- The IBM Plex typeface pair is used for text and for code-style detail blocks.

## Browser support

Works in current versions of Chrome, Edge, Firefox and Safari. It relies on modern CSS (grid, custom properties, `:has()` for highlighting the selected scenario) and modern JavaScript (async functions), so very old browsers are not supported.

## Testing checklist

There is no automated test suite. Use this list to check the main paths by hand:

- [ ] Registration with the pre-filled values reaches the home screen
- [ ] Registration rejects a short name and a mobile number that isn't 10 digits
- [ ] **Skip with a demo account** signs in as `arpit@sbi` with ₹25,000
- [ ] Payee validation rejects a missing `@`, your own VPA, ₹0, decimals and amounts over ₹1,00,000
- [ ] Normal payment ends in `SUCCESS`, and both ledgers change by the right amount
- [ ] A wrong PIN ends in `FAILED` with no debit
- [ ] An amount above the balance ends in `FAILED` with **Insufficient balance**
- [ ] An unknown bank handle ends in `FAILED` with **Payee VPA not found**
- [ ] **Double tap** shows two requests received but one payment processed
- [ ] **Network drops after paying** shows pending, then success, with one debit
- [ ] **Beneficiary bank is down** ends in `FAILED` and the balance is restored
- [ ] **Attacker tampers with data** ends in `FAILED` and no database row is created
- [ ] **SQL injection attempt** ends in `FAILED` with `403 Forbidden` and no database row
- [ ] Step-through mode pauses at each event, and Space and → advance it
- [ ] **Reset demo** restores the balance, counters, table and log
- [ ] Light and dark themes both look correct

## Troubleshooting

| Problem | What to try |
|---|---|
| The fonts look different or plain | The page couldn't reach Google Fonts. It falls back to system fonts, which is fine |
| Nothing happens after the PIN | Make sure you entered 4 digits. If you are in **Step through** mode, press **Next step** or Space |
| The animation feels too slow | Choose **Fast**, or turn off **Step through** |
| The scenario options are greyed out | A payment or registration is in progress. Wait for it to finish, or press **Reset demo** |
| Space doesn't advance the log | Step-through must be on, and your cursor must not be in a text field |
| The balance or counters look wrong after many tests | Press **Reset demo** to start from a clean state |
| The page looks cramped | Widen the window. Below about 980 px the layout stacks into one column |

## FAQ

**Is any of this real?**
No. There is no server, no bank and no network traffic beyond loading fonts. Every message in the log is generated by the page itself.

**Why is the PIN always 1234?**
It keeps the demo quick. The PIN check also lets you trigger a declined payment on purpose by entering anything else.

**Why does the PIN travel as an "encrypted blob"?**
In real UPI the PIN is encrypted on the device and only the issuing bank can read it. The prototype shows that idea in the log without implementing real cryptography.

**Why doesn't a failed payment just try again automatically?**
Re-sending a payment risks charging twice. PayFlow instead polls for status with `GET`, which only reads, and uses an idempotency key so even a duplicate `POST` can't create a second payment.

**What is the difference between PENDING and FAILED?**
`PENDING` means the outcome is unknown. `FAILED` means it is known that the payment did not complete. A payment can be debited and pending at the same time, which is why the app tells you not to pay again.

**Can I use this in a class or talk?**
Yes. Step-through mode and the three speeds are designed for presenting, and each scenario maps to one idea you can explain.

**Can I add my own scenario?**
Yes. See [Adding a new scenario](#adding-a-new-scenario).

## Glossary

| Term | Meaning |
|---|---|
| **UPI** | Unified Payments Interface, India's instant bank-to-bank payment system |
| **VPA** | Virtual Payment Address, like `name@bank`, used instead of an account number |
| **NPCI** | National Payments Corporation of India, which runs the UPI switch |
| **Payer bank / beneficiary bank** | The bank that sends the money, and the bank that receives it |
| **ReqPay / RespPay** | The UPI request and response messages for a payment |
| **Device binding** | Tying an account to one phone by sending a one-time code to its SIM |
| **JWT** | JSON Web Token, a signed token that proves who you are for a short time |
| **Idempotency key** | A unique ID sent with a request so that repeating it has the same effect as sending it once |
| **DNS** | The system that turns a name like `api.payflow.in` into an IP address |
| **TTL** | Time to live. For DNS, how long an answer can be cached. For IP, how many hops a packet may take |
| **NAT** | Network address translation, where a router maps private addresses to a public one |
| **TCP handshake** | The SYN, SYN-ACK and ACK exchange that opens a reliable connection |
| **TLS** | The protocol that encrypts and authenticates data between the app and the server |
| **Keep-alive** | Reusing an open connection for several requests instead of opening a new one each time |
| **Exponential backoff** | Waiting longer between each retry (here 1 s, then 2 s) so a weak network isn't flooded |
| **Reversal** | Returning a debited amount to the payer when the payment cannot be completed |
| **Firewall (WAF)** | A filter that blocks requests that look like attacks, such as SQL injection |

## Notes and limits

- All data, balances and "banks" are simulated and reset when you reload or press **Reset demo**.
- This is a teaching prototype, not production code or a security reference. Real UPI systems involve many more checks, partners and regulations.
- The one-time code is shown on screen as a toast because there is no real SMS.
- The rate limit, token lifetime text and firewall rule number in the log are fixed illustrative values, not live calculations.
- Log timestamps are simulated for readability and are not real measurements.

## Ideas for future work

These are suggestions, not promises:

- Wrong-PIN counter that actually locks UPI after three attempts
- A "request money" (collect) flow and a QR scan screen
- A transaction history screen with filters
- More failure cases: slow server, expired token, rate-limit hit, certificate error
- An exportable log (copy as text or download) for reports and write-ups
- A short guided tour that highlights each panel on first load
- Language options, including Hindi and other Indian languages
- Automated tests for the validation and state logic

## Contributing

Ideas and fixes are welcome. Open an issue to discuss a change, or send a pull request. Please keep the project to a single file with no build step unless there's a strong reason, and test the main paths with the checklist above before submitting.

## License

No license has been chosen yet. Until one is added, all rights are reserved by the authors. If you want others to be able to use or build on the prototype, add a `LICENSE` file (MIT and Apache-2.0 are common choices) and update this section.

## Project structure

```
index.html   the whole prototype (HTML, CSS and JavaScript in one file)
README.md    this file
```
