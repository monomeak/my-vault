# Expose Local Jenkins with ngrok

**Goal:** Make a locally running Jenkins server (port `8080`) reachable from the internet — for GitHub webhooks, external pipeline testing, or remote access.

## Prerequisites

- Jenkins running locally on port `8080`
- ngrok installed and authenticated

## Steps

1. **Check Jenkins is running**
    
    ```bash
    curl http://localhost:8080
    ```
    
2. **Set up ngrok authentication** (first time only)
    
    ```bash
    ngrok config add-authtoken YOUR_NGROK_AUTHTOKEN
    ```
    
3. **Start the tunnel**
    
    ```bash
    ngrok http 8080
    ```
    
    Copy the generated URL, e.g. `https://abc123.ngrok-free.app`.
    
4. **Configure the GitHub webhook** (if applicable)
    
    Go to **Settings → Webhooks → Add webhook** in your GitHub repo:
    
    |Field|Value|
    |---|---|
    |Payload URL|`https://YOUR_NGROK_URL/github-webhook/`|
    |Content type|`application/json`|
    |Events|Just the push event (or as needed)|
    
5. **Enable the trigger in Jenkins**
    
    In the job configuration, enable:
    
    ```
    GitHub hook trigger for GITScm polling
    ```
    

## Troubleshooting

|Symptom|Likely Cause|
|---|---|
|ngrok returns `502 Bad Gateway`|Jenkins isn't running or not listening on `8080`|
|GitHub webhook returns `404`|Payload URL is missing the trailing `/github-webhook/`|
|Webhook delivered, pipeline doesn't run|Job trigger not enabled, wrong branch, or wrong repo|
|Need to inspect a request|Open ngrok's local dashboard: `http://localhost:4040`|

## Security

The ngrok URL exposes Jenkins to the public internet. Before sharing it:

- Use a strong administrator password
- Disable anonymous/administrator access
- Keep Jenkins plugins updated
- Restrict job permissions

## Quick Reference

```bash
# Verify Jenkins
curl http://localhost:8080

# Configure ngrok auth
ngrok config add-authtoken YOUR_NGROK_AUTHTOKEN

# Start tunnel
ngrok http 8080
```

```text
GitHub webhook URL:  https://YOUR_NGROK_DOMAIN/github-webhook/
ngrok inspector:     http://localhost:4040
```