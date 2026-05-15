
# NashNumber — Dokku Deployment Guide

Keep this file outside your git repo. It contains server-specific details.

---

## One-time Dokku setup (on your Lightsail instance)

SSH into your server, then run:

```bash
# Create the app
dokku apps:create nashnumber

# Set your domain
dokku domains:add nashnumber nashnumber.yourdomain.com

# Remove the default domain Dokku adds
dokku domains:remove nashnumber nashnumber.YOUR_SERVER_IP.sslip.io

# Set up SSL via Let's Encrypt
dokku plugin:install https://github.com/dokku/dokku-letsencrypt.git
dokku letsencrypt:set nashnumber email you@youremail.com
dokku letsencrypt:enable nashnumber

# Auto-renew SSL
dokku letsencrypt:cron-job --add
```

---

## First deploy (from your local machine)

```bash
cd nashnumber
git remote add dokku dokku@YOUR_LIGHTSAIL_IP:nashnumber
git push dokku main
```

Dokku detects Node.js via `package.json`, runs `npm install` and `npm start` automatically.

---

## Adding piano samples to the server

The soundfont files are not in the repo (too large). After deploying, copy them to the server:

```bash
# From your local machine, copy the samples folder to the server
scp -r public/soundfonts ubuntu@YOUR_LIGHTSAIL_IP:/tmp/soundfonts

# SSH into the server
ssh ubuntu@YOUR_LIGHTSAIL_IP

# Find where Dokku stores the app files
ls /home/dokku/nashnumber/

# Copy samples into the app's persistent storage
# First, create a Dokku storage mount so files survive redeploys
dokku storage:ensure-directory nashnumber-soundfonts
dokku storage:mount nashnumber /var/lib/dokku/data/storage/nashnumber-soundfonts:/app/public/soundfonts

# Copy samples into the storage directory
sudo cp -r /tmp/soundfonts/* /var/lib/dokku/data/storage/nashnumber-soundfonts/

# Redeploy to pick up the storage mount
git push dokku main
```

---

## Updating the app

```bash
git add .
git commit -m "your message"
git push dokku main
```

---

## Useful Dokku commands

```bash
# View logs
dokku logs nashnumber -t

# Restart the app
dokku ps:restart nashnumber

# Check app status
dokku ps:report nashnumber

# View domains
dokku domains:report nashnumber

# Renew SSL manually
dokku letsencrypt:enable nashnumber
```

---

## DNS setup (on AWS Lightsail or your registrar)

Add an A record pointing to your Lightsail static IP:

| Type | Name        | Value              |
|------|-------------|--------------------|
| A    | nashnumber  | YOUR_LIGHTSAIL_IP  |

If your domain is managed in Route 53 or Lightsail DNS, add the record there.
Allow up to 24 hours for DNS to propagate (usually much faster).