# PayFlow · UPI prototype

**[Live demo](https://yourname.github.io/payflow-prototype/)**

An interactive, single-file simulation of a UPI payment. It explains what happens between tapping **Pay** on a phone and seeing "Payment successful", including the parts users never see: DNS, TCP and TLS handshakes, authentication, firewall checks, database writes, the bank switch, and what happens when things go wrong.

Everything runs in the browser. No real payments, banks or servers are involved.

## What it is

The screen is split into two halves:

- **The phone (left):** a mock UPI app with registration, home, send money, PIN entry and a result screen. It behaves like a real payment app, including input validation, a PIN pad, transaction history and status labels.
- **The inspector (right):** a live view of everything happening behind the scenes while you pay.
  - **Path of a payment:** a diagram of the route from phone to internet, PayFlow server, NPCI switch, payer bank and beneficiary bank, with a packet moving along it.
  - **Step-by-step log:** every event, tagged by layer (App, DNS, TLS, TCP, IP, Link, UPI, Security), with the actual request and response text.
  - **Server and bank state:** requests received, payments processed, the transactions table and the bank ledger.

## Run it

No build step and no dependencies. Open `index.html` in a browser, or serve the folder:

```bash
python3 -m http.server 8000
# then visit http://localhost:8000
```

The only external request is Google Fonts (IBM Plex Sans and Mono). If you are offline, it falls back to system fonts.

## How to use it

1. **Register** with the pre-filled details, or click **Skip with a demo account** to go straight to the home screen.
2. **Choose a scenario** in the inspector (see below).
3. Click **Send money**, enter a payee VPA (such as `name@bank`), an amount and an optional remark.
4. Enter any 4-digit **PIN** on the keypad.
5. Watch the phone and the inspector together as the payment moves through each stage.

### Controls

| Control | What it does |
|---|---|
| Speed | Slows down or speeds up the animation |
| Step through | Pauses at every event. Press **Next step**, or use **Space** / **→** |
| Skip handshake after the first request | Reuses the open connection, like a real app with keep-alive |
| Reset demo | Returns to the registration screen with a fresh state |

## Scenarios

| Scenario | What happens | What it demonstrates |
|---|---|---|
| Normal payment | The request goes from phone to server, bank and back | The full happy path |
| Network drops after paying | The bank debits, but the reply never reaches the phone | The app must not assume failure. It checks status and shows "pending" rather than letting you pay twice |
| Double tap | The same request arrives twice | Idempotency: the server recognises the key, so only one payment is made |
| Beneficiary bank is down | The payment waits, then fails | Timeouts and automatic reversal of the money |
| Attacker tampers with data | A man-in-the-middle changes bytes in transit | TLS detects the change and closes the connection before it reaches the payment code |
| SQL injection attempt | A malicious payee value is sent | The firewall blocks it, and parameterised queries protect the database anyway |

## Concepts it covers

- **Connection set-up:** DNS lookup (UDP, with TTL caching), TCP three-way handshake, and TLS 1.3 key exchange with certificate checks.
- **Encryption in transit:** requests are shown readable on the app side, then as authenticated ciphertext (AES-256-GCM) on the wire.
- **Authentication:** a signed token (JWT, HS256) with an expiry time, checked on every request.
- **Request protection:** firewall rules, rate limiting and input validation.
- **Idempotency:** an `Idempotency-Key` header plus a `UNIQUE(txn_id)` rule in the database, so retries never create duplicate payments.
- **Transaction states:** `INITIATED` → `PENDING` → `SUCCESS` or `FAILED`, with the status polled when the reply is lost.
- **UPI rules:** VPA format checks, whole-rupee amounts, and the ₹1,00,000 per-payment limit.

## Notes and limits

- All data, balances and "banks" are simulated. The starting balance and demo user are fixed for each session.
- The IP address, domain and rule numbers shown are illustrative, not real services.
- This is a teaching prototype, not a security reference or production code.
- Light and dark themes follow the system setting.

## Project structure

```
index.html   the whole prototype (HTML, CSS and JavaScript)
README.md    this file
```

## Contributing

Ideas and fixes are welcome. Open an issue or a pull request describing the change.
