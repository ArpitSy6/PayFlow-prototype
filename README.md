# PayFlow · UPI prototype

An interactive, single-file simulation of a UPI payment, built to explain what happens between tapping **Pay** and seeing the result. A phone mock-up sits on the left; on the right, an inspector shows the network path, security checks, server database and event log as the payment moves through each stage.

## Run it

No build step and no dependencies. Open `index.html` in a browser, or serve the folder:

```bash
python3 -m http.server 8000
# then visit http://localhost:8000
```

The only external request is Google Fonts (IBM Plex). If offline, it falls back to system fonts.

## What you can try

Pick a scenario before paying:

| Scenario | What it shows |
|---|---|
| Normal payment | The happy path from tap to bank and back |
| Network drops after paying | The reply is lost; the app checks status and nothing is debited twice |
| Double tap | The same request arrives twice; only one payment happens (idempotency key) |
| Beneficiary bank is down | The payment waits, then fails and the money is reversed |
| Attacker tampers with data | TLS detects the change and closes the connection |
| SQL injection attempt | The firewall blocks the malicious payload |

Controls: pacing (speed), step-through mode (Space / → to advance), skip handshake, and reset.

Use **skip registration** on the first screen to jump straight to the home screen with a demo user.

## Notes

- All data is simulated in the browser. No real payments, banks or network calls are involved.
- Light and dark themes follow the system setting.

## Structure

```
index.html   the whole prototype (HTML, CSS, JS)
README.md
.gitignore
```
