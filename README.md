# ROCK MASTER CONTROL

A free, static browser controller prototype for the TOYPOT Wonder Rock Master RC car using Web Bluetooth (BLE GATT).

## Important status and safety

- The forward packet and stop packet are the only commands described as physically verified.
- Reverse, left, and right default to the bytes reported during reverse-engineering, but are marked **unverified** in the interface.
- The app sends one direction at a time. Combined throttle + steering has not been established because the command protocol is not yet confirmed.
- A browser cannot guarantee a stop packet will reach the car after a radio failure, tab suspension, or phone/browser crash. Test with wheels raised and keep the car supported.
- No physical-car test has been performed by this project generator. Treat this as source code to test, not a confirmed working controller.

## Files

- `index.html` — complete single-page controller. No framework, build step, or paid service required.
- `README.md` — this guide.

## Run locally on your Mac

Web Bluetooth needs a secure context. `http://localhost` is treated as secure by modern browsers, but your Android phone cannot use your Mac's `localhost` as its own. For the phone, publish to HTTPS with GitHub Pages.

For local preview on the Mac:
1. Open the folder in VS Code.
2. Install/use the Live Server extension if you already have it, or run a simple local web server.
3. Open the page in a compatible browser. The Mac is only for preview/development; Bluetooth compatibility with the car is not guaranteed on desktop.

## Publish free with GitHub Pages

1. On your Mac, go to https://github.com and sign in or create a free account.
2. Click **New repository**. Name it `rock-master-control`. Choose **Public** and create it.
3. Upload `index.html` and `README.md` to the repository root (the top-level folder). On GitHub, use **Add file → Upload files**, drag the two files in, then commit.
4. In the repository, open **Settings → Pages**.
5. Under **Build and deployment**, select **Deploy from a branch**. Choose branch `main`, folder `/ (root)`, then **Save**.
6. Wait for the Pages URL to appear. It will look like `https://YOUR-USERNAME.github.io/rock-master-control/`. Use the exact URL GitHub shows.
7. Open that HTTPS URL in Chrome on your Vivo phone. Keep the car powered on and nearby.

GitHub Pages is free for public repositories. GitHub's menus can change slightly over time.

## Test on the Vivo Android phone

1. Charge the car and place it on a stable support so all wheels are off the ground. Turn the physical car switch ON.
2. Open the published HTTPS page in **Google Chrome**. If prompted, allow nearby devices/Bluetooth permission.
3. Tap **CONNECT CAR** and choose `YC_CAR_DEMO` in the browser's device picker.
4. Confirm the dashboard says **CONNECTED**.
5. Briefly press and hold **FORWARD**. It repeatedly sends `AA 00 02 00 00 00 00 42 64 01` about every 100 ms.
6. Release the button. The page queues `AA 00 02 00 00 00 00 40 00 01` (STOP).
7. Only after forward/stop are behaving safely, test reverse, left, and right separately, with short taps and wheels raised. Stop immediately if behavior is unexpected.
8. If the page cannot see BLE, check that you're using Chrome on Android, the page URL is HTTPS, Bluetooth is on, and the car is on. Some phone/browser builds may not support Web Bluetooth correctly.

## Packet details

Service: `FFF0`  
Write characteristic: `FFF2`

Forward (verified):
`AA 00 02 00 00 00 00 42 64 01`

Stop (verified):
`AA 00 02 00 00 00 00 40 00 01`

Other directions use the template:
`AA 00 02 00 00 00 00 [direction] [speed/mode] 01`

The settings panel lets you edit the speed/mode byte and unverified reverse/left/right direction bytes as two-digit hexadecimal values. Forward is currently fixed to `42` with the speed/mode byte from the settings field (default `64`). Stop is fixed to `40 00`.

## Technical behavior

- Requests a BLE device named exactly `YC_CAR_DEMO`.
- Connects to service `FFF0` and characteristic `FFF2`.
- Uses write-without-response if the characteristic advertises it; otherwise uses write-with-response.
- Serializes writes through a promise queue and repeats a movement packet approximately every 100 ms after each write completes.
- Attempts STOP on control release, emergency stop, focus loss, page hidden, and disconnect.
- Reconnection is manual using **CONNECT CAR**.

## Limitations

- The browser/device may suspend the page or interrupt BLE. Stop-on-release is best effort, not a safety-rated failsafe.
- If your car needs notifications, another initialization packet, a different device-name filter, or a different packet format for unverified directions, this prototype will need adjustments.
- If your browser reports `Web Bluetooth is unavailable`, use a compatible Android Chrome version or consider a native Android BLE app as a fallback.
