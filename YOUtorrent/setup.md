# Nightline Setup Guide

This project is a single-file WebRTC video chat app built in [`index.html`](/C:/Users/NIKHIL/YOUtorrent/index.html).

It uses:

- PeerJS for signaling
- Browser `getUserMedia()` for camera and microphone
- WebRTC for peer-to-peer media
- No backend of your own

## What matters before you test

This app is not like a normal static HTML page.

Two conditions matter immediately:

1. It should be opened from a real website URL, ideally `https://`
2. Peer-to-peer WebRTC may still fail on some networks without a TURN server

So:

- Good: GitHub Pages, Netlify, Vercel, any HTTPS static host
- Good for local-only testing: `http://localhost`
- Bad for real device sharing: opening the file directly as `file:///.../index.html`

## Project files

- [`index.html`](/C:/Users/NIKHIL/YOUtorrent/index.html): the full app
- [`setup.md`](/C:/Users/NIKHIL/YOUtorrent/setup.md): this guide

## Current architecture

### Signaling

The app uses the public PeerJS cloud server to exchange:

- peer IDs
- offer/answer data
- ICE candidates

No custom backend is required for signaling.

### Media

Once both peers connect, the browser attempts a direct WebRTC media path.

That means:

- it can work well between many devices
- it can fail on strict NATs, mobile networks, office Wi-Fi, or carrier networks
- this is expected behavior for pure P2P without TURN relay

## Quick local test

If you just want to test on one machine first:

1. Open the app in a modern browser.
2. Click `Create Room`.
3. Allow camera and microphone access.
4. Open a second browser window or an incognito window.
5. Paste the room code into `Join Room`.
6. Allow camera and microphone access there too.

If both windows are on the same machine or same network, this usually works more easily than cross-device internet testing.

## Local serving options

Do not test this by double-clicking the HTML file and sending it around.

Use one of these approaches instead.

### Option 1: Python static server

If Python is installed:

```powershell
cd C:\Users\NIKHIL\YOUtorrent
python -m http.server 8080
```

Then open:

- `http://localhost:8080`

This is fine for local desktop testing on the same machine.

### Option 2: VS Code Live Server

If you use VS Code:

1. Open the folder
2. Start Live Server
3. Open the generated localhost URL

### Important note

`http://localhost` is treated specially by browsers and usually allows camera and microphone.

But `http://192.168.x.x:8080` on another phone or laptop may not behave the same way, especially on mobile. For real sharing, use HTTPS hosting.

## Best way to test on a friend's device

Deploy the app to a public HTTPS URL.

### Recommended static hosts

- GitHub Pages
- Netlify
- Vercel

Because the project is only one HTML file, any of these is enough.

## Deploy with GitHub Pages

### Step 1: Create a GitHub repository

Push this folder to GitHub.

Example:

```powershell
cd C:\Users\NIKHIL\YOUtorrent
git init
git add .
git commit -m "Initial Nightline app"
git branch -M main
git remote add origin <your-repo-url>
git push -u origin main
```

### Step 2: Enable Pages

In your GitHub repo:

1. Go to `Settings`
2. Open `Pages`
3. Set source to `Deploy from a branch`
4. Choose branch `main`
5. Choose folder `/ (root)`
6. Save

GitHub will give you a URL like:

- `https://yourname.github.io/your-repo/`

### Step 3: Test from two devices

1. Open the hosted URL on your device
2. Open the same URL on your friend's device
3. Create a room on one device
4. Join with the code on the other device
5. Allow camera and mic permissions on both devices

## Deploy with Netlify

This is often the fastest option for static apps.

### Drag-and-drop deploy

1. Zip the project folder or just keep the file ready
2. Go to Netlify
3. Create a new site by drag-and-drop
4. Upload the project folder
5. Open the generated HTTPS URL

Since there is no build step, deployment is trivial.

## Deploy with Vercel

Vercel also works with no build pipeline here.

1. Import the GitHub repository
2. Framework preset: `Other`
3. No build command needed
4. Output directory: leave default or project root
5. Deploy

## Browser requirements

Use recent versions of:

- Chrome
- Edge
- Safari
- Firefox

For the smoothest mobile testing:

- Android: Chrome
- iPhone: Safari

## Permission requirements

The app needs:

- camera permission
- microphone permission

If either device blocks access, the call will not start correctly.

### On phones

Make sure:

- the browser itself has OS camera permission
- the site has camera permission
- the site has microphone permission

If you denied once before, reset site permissions and reload.

## Common reasons it fails on another device

### 1. Opened as a file instead of a website

Problem:

- user opens `index.html` directly from local storage

Effect:

- camera/mic access may fail
- PeerJS may behave inconsistently

Fix:

- host the file on HTTPS

### 2. Camera/mic permission denied

Problem:

- browser or OS blocks media devices

Fix:

- re-enable permissions
- reload the page

### 3. Unsupported or restricted browser

Problem:

- older browser
- embedded in-app browser

Fix:

- use full Chrome, Safari, Edge, or Firefox

### 4. NAT or firewall issue

Problem:

- one or both users are behind a restrictive network

Effect:

- room code works
- connection gets stuck on `Connecting`
- remote video never appears

Fix:

- move one user to a different network
- try mobile hotspot
- add TURN relay support

### 5. Corporate, school, or carrier network restrictions

Problem:

- UDP/WebRTC traffic is blocked or constrained

Fix:

- test on a home Wi-Fi network or hotspot
- use TURN

## Important limitation of the current app

The current implementation relies on direct P2P connectivity.

That means it does not guarantee successful calls between arbitrary devices and networks.

This is the biggest reason something can work on your machine but fail on your friend's phone or laptop.

## What is missing for production reliability

To make this app much more reliable across real-world networks, add explicit ICE servers:

- STUN server
- TURN server

### Why STUN alone is not enough

STUN helps peers discover public-facing addresses.

But if the network is symmetric NAT or heavily restricted, STUN will not save the connection.

In those cases, TURN is required because media is relayed through a server.

### Production recommendation

Use:

- PeerJS for signaling
- your own TURN credentials, for example from:
- Twilio Network Traversal
- Metered
- Xirsys
- Open Relay only for experiments, not production

## Suggested next improvement

Patch [`index.html`](/C:/Users/NIKHIL/YOUtorrent/index.html) so PeerJS creates the connection with:

- explicit `config.iceServers`
- better timeout/error messaging when peers cannot connect
- clearer UI for `network blocked / relay required`

## Recommended testing checklist

Before saying the app is broken, test this sequence:

1. Same browser, same computer, two windows
2. Two devices on the same Wi-Fi
3. Two devices on different networks
4. One device on mobile hotspot
5. Retry after clearing site permissions

This helps isolate whether the issue is:

- UI bug
- permission problem
- peer ID issue
- browser issue
- network traversal issue

## Troubleshooting checklist

If the other device cannot connect:

1. Confirm both devices are using the same public HTTPS URL
2. Confirm both devices allowed camera and microphone
3. Confirm the join code matches exactly
4. Confirm both devices use supported browsers
5. Try a different network
6. Try mobile hotspot
7. If still stuck on `Connecting`, the likely cause is missing TURN support

## Simple usage instructions for end users

### Host

1. Open the site
2. Click `Create Room`
3. Allow camera and microphone
4. Copy or share the generated code
5. Wait for the second user

### Guest

1. Open the same site
2. Enter the shared code
3. Click `Join Room`
4. Allow camera and microphone
5. Wait for connection

## If you want this app to work better on friends' devices

The next concrete step is not more UI work.

It is adding TURN-backed ICE configuration to the PeerJS setup.

Without that, cross-network reliability will always be limited.

## Next step

If you want, I can patch [`index.html`](/C:/Users/NIKHIL/YOUtorrent/index.html) next so it supports:

- configurable ICE servers
- TURN-ready PeerJS setup
- better connection failure messaging
- a small settings block where you can paste TURN credentials
