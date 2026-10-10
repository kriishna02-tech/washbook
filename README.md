# WashBook — Hostel Washing Machine Booking

React + Firebase application for hostel laundry reservations. Residents can see machine availability, book a 30-, 45-, or 60-minute session, extend once by 15 minutes, and view their history. Administrators can manage maintenance, resolve sessions, and review discrepancy reports.

The shared backend uses Firestore transactions in callable Cloud Functions. Browser clients have read-only access under Firestore security rules. A browser demo is included so you can try the interface immediately.

## Requirements

| Tool | Version | Needed for |
| --- | --- | --- |
| Node.js and npm | Node.js 24+ | Running the app, tests, and build |
| Git | Installed and available in your terminal | Cloning and pulling updates |
| Java | 21+ | Firebase emulator tests and the local backend |

The browser demo, unit tests, and production build require no Java installation or Firebase account.

## Quick start

If you have not downloaded the project yet:

```powershell
git clone https://github.com/kriishna02-tech/washbook.git
cd washbook
```

If you already have the repository, open its folder in VS Code and select **Terminal > New Terminal**. Run all commands below from the folder containing `package.json`. To get the latest changes in an existing clone:

```powershell
git pull --ff-only
```

### Windows PowerShell

```powershell
npm.cmd ci
npm.cmd run dev
```

`npm.cmd` runs npm without depending on PowerShell's script execution policy.

### macOS or Linux

```bash
npm ci
npm run dev
```

Open [http://localhost:5173](http://localhost:5173) and select **Explore the demo**. Keep the terminal open while using the app; **Ctrl+C** stops the server. If Vite prints a different port, open the address it prints.

Demo mode is enabled by default when there is no `.env` file. If you have already configured one, set `VITE_DEMO_MODE=true` and restart the development server to return to the demo. Demo bookings are saved in your browser. Use the local Firebase backend below to test shared reservations between residents.

## Check the demo in your browser

Complete these steps in order after selecting **Explore the demo**:

| Step | Action | Expected result |
| --- | --- | --- |
| 1 | Select **Reserve machine** on an available machine, choose a duration, and confirm. | An active session appears with a countdown. |
| 2 | Check the other available machines while your session is active. | Their reservation buttons are disabled; you can hold only one active booking. |
| 3 | Select **+ 15 minutes**. | The end time increases by 15 minutes and the extension button becomes disabled. |
| 4 | Select **Finish session**, then open **History**. | The machine becomes available and the session appears as completed. |
| 5 | Select **Try admin view**, then **Set maintenance** on an available machine and enter a reason. | The machine becomes unavailable for reservations. **Return to service** makes it available again. |

Select **Resident view** to leave the administrator demo.

## Run tests and create a production build

Open a second terminal in the same repository folder. On Windows PowerShell, run these commands one at a time:

```powershell
npm.cmd test
npm.cmd run build
npm.cmd run test:full-emulator
```

On macOS or Linux, use `npm` in place of `npm.cmd`.

| Command | Expected result | Java required? |
| --- | --- | --- |
| `npm.cmd test` | 3 domain tests pass. | No |
| `npm.cmd run build` | TypeScript checking and Vite build succeed; output is written to `dist/`. | No |
| `npm.cmd run test:full-emulator` | 14 integration/security-rule tests and 2 callable HTTP tests pass. | Yes, 21+ |

The full suite starts and stops its own Auth and Firestore emulators. Stop any interactive emulator instance with **Ctrl+C** before running it so the ports are free. The first run downloads emulator binaries. The full test total is **19 tests**, including the 3 domain tests run separately. GitHub Actions runs the tests and build on Windows and Ubuntu.

The tests cover competing reservations, one active booking per resident, stale/repeated extensions, expired reservations, idempotent retries, ownership fencing, and owner/admin authorization. Security-rule tests attempt direct client writes, privilege escalation, and access to another resident's sessions. The callable tests use the Firebase web SDK against the exported HTTPS handlers on a local TCP port and verify real Auth emulator tokens.

To preview the production build after `npm.cmd run build`:

```powershell
npm.cmd run preview
```

Open the address printed in the terminal, normally [http://localhost:4173](http://localhost:4173).

## Run the shared Firebase backend locally

This mode uses local Firebase emulators and does not require a Firebase account. Install Java 21+ first. Stop the development server with **Ctrl+C** before changing modes.

### 1. Configure the app

In PowerShell, create `.env` only if it does not already exist:

```powershell
if (-not (Test-Path .env)) { Copy-Item .env.example .env }
```

On macOS or Linux, the equivalent command is:

```bash
test -f .env || cp .env.example .env
```

Open `.env` in your editor and set these two values, keeping the other sample values:

```dotenv
VITE_DEMO_MODE=false
VITE_USE_EMULATORS=true
```

### 2. Start the emulators in terminal 1

```powershell
npm.cmd run emulators
```

Wait until the emulators report that they are ready. Keep this terminal open.

### 3. Seed the database and start the app in terminal 2

Open another terminal in the repository folder:

```powershell
$env:FIRESTORE_EMULATOR_HOST = "127.0.0.1:8080"
npm.cmd run seed
npm.cmd run dev
```

On macOS or Linux, use `export FIRESTORE_EMULATOR_HOST=127.0.0.1:8080` and replace `npm.cmd` with `npm` in both terminals.

Open [http://localhost:5173](http://localhost:5173) and select **Enter local emulator**. This creates a local anonymous development user. The emulator console at [http://localhost:4000](http://localhost:4000) shows users and database records.

### 4. Test two residents

Use a normal browser window for one resident and an incognito window or a separate browser profile for another. Two normal tabs share the same authenticated user.

Book a machine as the first resident. Both windows should show the updated availability, and the second resident cannot reserve the occupied machine. If both residents attempt to reserve the same machine at the same time, only one reservation can succeed. Finish the session and check that the machine becomes available in both windows.

### Optional: make a local user an administrator

Copy the user's UID from the emulator console's Authentication page, then run this in PowerShell from the repository folder, replacing `YOUR_EMULATOR_UID`:

```powershell
$env:FIREBASE_AUTH_EMULATOR_HOST = "127.0.0.1:9099"
node functions/scripts/set-admin.mjs YOUR_EMULATOR_UID demo-washbook
```

On macOS or Linux, set the variable with `export FIREBASE_AUTH_EMULATOR_HOST=127.0.0.1:9099` before running the same `node` command. Sign out and sign in again to refresh the administrator claim. Production login uses Google authentication, and production roles require a privileged Admin SDK identity.

Restart the Vite server whenever you change `.env`. Local database data is temporary and can be recreated with `npm.cmd run seed` after restarting the emulators.

## Troubleshooting

| Problem | What to check |
| --- | --- |
| `npm.cmd` or `node` is not recognized | Install Node.js 24+ and reopen your terminal. Check `node --version` and `npm.cmd --version`. |
| PowerShell says `npm.ps1` cannot be loaded | Use the `npm.cmd` commands shown above. |
| npm cannot find `package.json` | Change to the `washbook` repository folder before running npm. |
| The app is not at port 5173 | Open the exact address printed by Vite. |
| Emulator tests report a Java error | Check `java -version`; Java 21+ must be available in the terminal. |
| An emulator port is already in use | Stop your existing emulator terminal with **Ctrl+C** before starting another instance or the automated suite. |
| Seeding says `FIRESTORE_EMULATOR_HOST` is missing | Set the variable in the same terminal where you run `npm.cmd run seed`. |

## Deploy to your Firebase project

1. Create a Firebase project, enable Firestore and Google sign-in, and add your site domain to Authentication's authorized domains.
2. Set the Firebase web configuration in `.env`; set both `VITE_DEMO_MODE=false` and `VITE_USE_EMULATORS=false`. Web configuration is not an Admin SDK credential. Never place a service-account key in the browser or Git.
3. Use Node 24, log in through `npx firebase login`, and select your project with `npx firebase use --add`.
4. Run `npm run build`, then `npx firebase deploy --only firestore,functions,hosting --project YOUR_PROJECT_ID`.
5. Create machine documents matching the seed schema through a privileged Admin SDK or Firestore console. The seed script deliberately refuses to write to production. Grant the administrator custom claim with the Admin SDK script using your explicit project ID and Application Default Credentials.

Cloud Functions and the scheduled expiry task require a Firebase billing-enabled project. No cloud resource is created and no paid service is enabled by cloning or running the local demo. Live deployment and Google sign-in are unverified until a real project is configured.

## Reservation invariants

- Machine, session, and user lease change in one server transaction.
- Machine version checks reject old snapshots. Session version checks prevent concurrent stale extensions or finish calls.
- A user lease enforces one active reservation per resident, even across different machines.
- The server's time determines eligibility. The browser clock only renders a countdown.
- Expired reservations can be reclaimed during the next transaction. A scheduled task also releases them every minute.
- Request IDs make retries idempotent; reuse for a different operation is rejected. Request-cache documents include `expiresAt`. Configure a Firestore TTL policy on that field to prune them automatically if desired.
- Administrator machine updates and report resolutions include audit records and reasons.
- Direct browser writes are denied, including for administrators. Admin reads use a signed custom claim, and all mutations pass through callable functions.

All Firestore transaction reads precede their writes. This avoids the SDK's read-after-write rejection while maintaining a consistent view of ownership and expiry.

References: [Firestore transactions](https://firebase.google.com/docs/firestore/manage-data/transactions), [callable functions](https://firebase.google.com/docs/functions/callable), [security rules](https://firebase.google.com/docs/firestore/security/get-started).

## Validation

See [the validation record](docs/VALIDATION.md) for completed checks and live-service requirements. Dependencies are recorded in `package-lock.json`, and [the CI workflow](.github/workflows/ci.yml) runs checks from this repository root on Windows and Ubuntu.

![WashBook](docs/washbook-desktop.png)
