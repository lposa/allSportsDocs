# Go Live

### Go Live Checklist
- [ ] Make sure everything is tested and there are no breaking bugs
- [ ] Make sure build is passing without errors
- [ ] Make sure apphosting.yaml file is up to date with all .env variables and matches with the secrets in google secret manager
- [ ] Make sure all firebase extensions are properly configured (StripePayments)
- [ ] Make sure Resend is properly setup, and the webhook is configured to run on production url

### Possible issues:
1. Not all env variables are properly configured
2. Stripe firebase extension is not properly configured
3. Resend is not properly configured - correct webhook link should be https://usport.ai/api/email/resend-webhook
4. Code breaking bugs (should be avoided by testing first)
5. Build can fail, check issues here https://console.firebase.google.com/u/1/project/all-sports-us/apphosting/backends/us-sports/locations/us-central1/overview
