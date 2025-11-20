# Local Development Setup (ngrok + Resend)

Resend needs a publicly accessible HTTPS URL to deliver webhooks. During local development we tunnel our Next.js dev server with [ngrok](https://ngrok.com/). Below is a quick checklist to reproduce the environment.

1. **Install ngrok (one time)**
   ```bash
   # macOS (Homebrew)
   brew install ngrok/ngrok/ngrok

   # Windows / Linux
   # Download binary from https://ngrok.com/download and follow installer instructions
   ```
   After installing, run `ngrok config add-authtoken <YOUR_NGROK_TOKEN>` so tunnels can be created from the CLI.

2. **Start Next.js locally**
   ```bash
   pnpm dev
   # or npm run dev / yarn dev, depending on your setup
   ```
   By default this serves the app at `http://localhost:3000`.

3. **Expose your local server via ngrok**
   ```bash
   ngrok http http://localhost:3000 \
     --domain=<custom-subdomain>.ngrok.app  # optional, requires paid plan
   ```
   ngrok will print a `https://xxxxx.ngrok.app` URL. Keep this terminal running so the tunnel stays alive.

4. **Configure the Resend webhook**
    - Go to your Resend dashboard → *Inbound* → *Webhooks*.
    - Create (or edit) a webhook pointing to the ngrok HTTPS URL plus our webhook route, e.g.
      ```
      https://xxxxx.ngrok.app/api/email/resend-webhook
      ```
    - Select the events you want (usually “email.received”) and save. If Resend needs the signing secret, copy the value into your local `.env` as `RESEND_WEBHOOK_SECRET`.

5. **Test the round-trip**
    - Send an email into the address associated with your Resend Receiving domain.
    - Watch the ngrok console for incoming requests and your Next.js dev server for logs.
    - The `syncResendEmails` Cloud Function (deployed in Firebase) will still process the payload because the Next.js webhook proxy forwards the message to the Cloud Function.

:::tip
When you restart ngrok you will usually get a new URL (unless you use a reserved domain). Every time that happens, update the webhook URL in the Resend dashboard so events keep flowing to your local machine.
:::