# Web Automation Container

This is a lightweight container environment configured to run web automation scripts using `playwright-core` and remote connections.
The container itself does not include a built-in browser. It serves as a runtime base image designed to remotely connect to and control external Chrome WebSocket endpoints, such as `chromedp/headless-shell`, for automation and scraping.

## Features

- **Standard Environment**: Built on standard Debian-based images for both Node.js and Bun, ensuring high compatibility.
- **Pre-installed Libraries**: Includes `playwright-core` and `otplib` (for Two-Factor Authentication/OTP) out of the box.
- **Multi-Runtime Available**: You can choose either a Node.js or a Bun image based on your needs.

## Docker Images & Tags

Images are published to the GitHub Container Registry (GHCR).

- [urihcim/webauto](https://github.com/urihcim/webauto-container/pkgs/container/webauto)

You can select the runtime environment and version by specifying the tag.

```bash
docker pull ghcr.io/urihcim/webauto:<TAG>
```

| Tag | Runtime Environment | Description |
|---|---|---|
| `latest` | Node.js | Default Node.js environment (latest `main` branch) |
| `node` | Node.js | Node.js environment (latest `main` branch) |
| `bun` | Bun | Bun environment (latest `main` branch) |
| `[version]` | Node.js | Versioned Node.js environment (e.g., `1.0.0`) |
| `[version]-node` | Node.js | Versioned Node.js environment (e.g., `1.0.0-node`) |
| `[version]-bun` | Bun | Versioned Bun environment (e.g., `1.0.0-bun`) |

## Usage

Prepare your script on the host machine and mount it to the container's `/app` directory to execute.

### Example (Node.js)

```bash
docker run --rm \
  -v $(pwd)/src:/app/src \
  ghcr.io/urihcim/webauto:node \
  node src/index.js
```

### Example (Bun)

```bash
docker run --rm \
  -v $(pwd)/src:/app/src \
  ghcr.io/urihcim/webauto:bun \
  bun src/index.ts
```

## Sample Code (Node.js/Bun)

Here is an example using Playwright to connect to a remote `chromedp/headless-shell`.

```javascript
import { chromium } from 'playwright-core';
import { authenticator } from 'otplib';

(async () => {
  // Retrieve the WebSocket endpoint and TOTP secret from environment variables
  const wsEndpoint = process.env.WS_ENDPOINT || 'ws://localhost:9222';
  const secret = process.env.OTP_SECRET;

  // Remotely connect to the browser
  const browser = await chromium.connectOverCDP(wsEndpoint);
  const context = browser.contexts()[0] || await browser.newContext();
  const page = await context.newPage();

  // Generate an OTP code if a secret is provided
  if (secret) {
    const token = authenticator.generate(secret);
    console.log(`Generated OTP: ${token}`);
  }

  // Navigate and perform tasks
  await page.goto('https://example.com');
  console.log(await page.title());

  await browser.close();
})();
```
