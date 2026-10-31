# PayFlow · UPI prototype

An interactive, single-file simulation of a UPI payment. It shows what really happens between tapping **Pay** on a phone and seeing "Payment successful": the DNS lookup, the TCP and TLS handshakes, authentication, firewall checks, database writes, the NPCI switch, the two banks, and what happens when something goes wrong along the way.

Everything runs in the browser. No real payments, banks, servers or network calls are involved.

---

## Table of contents

1. [Overview](#overview)
2. [Run it](#run-it)
3. [Screen layout](#screen-layout)
4. [The phone app](#the-phone-app)
5. [The inspector](#the-inspector)
6. [Scenarios](#scenarios)
7. [A payment, step by step](#a-payment-step-by-step)
8. [Concepts it demonstrates](#concepts-it-demonstrates)
9. [Controls and shortcuts](#controls-and-shortcuts)
10. [Validation rules and demo data](#validation-rules-and-demo-data)
11. [Design and accessibility](#design-and-accessibility)
12. [Notes and limits](#notes-and-limits)
13. [Project structure](#project-structure)

---

## Overview

Most people only see a spinner and then a green tick. PayFlow makes the invisible parts visible. You use a realistic mock UPI app on the left, and on the right an inspector narrates every event behind it, tagged by network layer, with the actual request and response text, while a packet travels across a diagram of the real payment path.

It was built to explain, in a hands-on way, how a payment can be fast, secure and safe against failure: why a double tap doesn't charge you twice, why a dropped signal doesn't lose your money, and how an attacker or a malicious input gets stopped.

## Run it

No build step and no dependencies. Open `index.html` in a browser, or serve the folder:

```bash
python3 -m http.server 8000
# then visit http://localhost:8000
```

The only external request is Google Fonts (IBM Plex Sans and IBM Plex Mono). If you are offline, it falls back to system fonts.

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

A toast notification slides down on completion, and an "No internet connection" banner appears on the result screen when the signal drops.

### Transaction states

`INITIATED` → `PENDING` → `SUCCESS` or `FAILED`

Each state has its own icon and colour (spinner, clock, tick and cross), and the phone and the server track their state separately so you can see when they disagree.

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

Rows are marked green when something goes right (a connection is established, a reversal succeeds) and red when something fails or is attacked. Each scenario starts with a separator line so runs are easy to tell apart.

### Server and bank state

- **Counters:** requests received and payments processed. These show, for example, that a double tap sends two requests but processes only one payment.
- **Server database table:** the last five transactions with ID, payee, amount and status, so you can watch a row appear as `INITIATED` and move through its states.
- **Bank ledgers:** live balances for the payer and the payee. The payer's balance drops when the payment is debited, the payee's rises when it is credited, and a row flashes whenever a value changes. Payees get a random starting balance the first time you pay them.

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

Other failures are also modelled: an unknown bank handle (**Payee VPA not found**), a wrong PIN (**Incorrect UPI PIN**, with the note that the bank locks UPI after 3 wrong tries), and a payment larger than your balance (**Insufficient balance**).

## A payment, step by step

For a normal payment the log walks through:

1. **Connection set-up (first payment):** a link-layer frame leaves the phone; a DNS A-record query goes out over UDP and is cached with a TTL; an IP packet is addressed to the server (NAT, TTL, routing); a TCP three-way handshake runs (SYN, SYN-ACK, ACK); a TLS 1.3 handshake runs (ClientHello, certificate and key share, Finished) to produce an encrypted tunnel using `TLS_AES_256_GCM_SHA384`.
2. **The request:** `POST /api/v1/transactions` with a bearer token and an `Idempotency-Key` header, shown in readable form with a note that on the wire it is ciphertext.
3. **Server checks:** firewall (no attack pattern), token (valid signature, expiry), rate limit (for example, 1 of 5
