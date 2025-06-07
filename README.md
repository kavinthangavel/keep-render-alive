# 🚀 Render Keep-Alive Service

Keep your free-tier web apps awake—automagically!  
No more “spinning up” delays. No more sleeping demos.  
Just set it and forget it. 💤❌

---

## What Is This?

A super-simple GitHub Actions workflow that pings your web services (like Render, Heroku, etc.) every 14 minutes so they never fall asleep.

- No servers to run.
- No admin dashboard.
- Just edit a file, commit, and let GitHub do the magic.

---

## How Does It Work?

1. **List your sites** in `websites.json` (in this repo).
2. **GitHub Action** runs every 14 minutes.
3. **Each enabled site gets pinged** (so it stays awake!).
4. **You chill.** 😎

---

## Quick Start

1. **Clone this repo** (or add the files to your own):

   ```pwsh
   git clone <your-repo-url>
   cd <your-repo>
   ```

2. **Edit `websites.json`:**

   ```json
   {
  "settings": {
    "defaultPingIntervalMinutes": 14,
    "enableMultipleServices": true,
    "maxRetries": 3
  },
     "websites": [
       {
         "id": "my-app",
         "name": "My Render App",
         "url": "https://my-app.onrender.com/healthz",
         "enabled": true,
         "pingIntervalMinutes": 14,
         "addedAt": "2025-06-07T00:00:00.000Z",
         "notes": "Main application endpoint"
       }
     ]
   }
   ```

   - Add as many sites as you want!
   - Set `"enabled": false` to pause pinging a site.
   - addedAt,notes is Optional

3. **Check the workflow:**  
   The file `.github/workflows/keep-alive.yml` is already set up to run every 14 minutes.

4. **Push your changes:**

   ```pwsh
   git add websites.json
   git commit -m "Add my app to keep-alive"
   git push
   ```

5. **Done!**  
   GitHub Actions will keep your sites awake.  
   Check the “Actions” tab for logs and results.

---

## FAQ

**Q: Can I add more than one site?**  
A: Yes! Just add more entries to `websites.json`.

**Q: Is this secure?**  
A: Don’t put private/internal URLs in a public repo.

**Q: Can I change the ping interval?**  
A: Yes, edit the cron schedule in `.github/workflows/keep-alive.yml`.

**Q: What if a site is down?**  
A: The workflow logs will show if a ping failed.

---

## License

MIT — Use it, share it, remix it.  
(See `LICENSE` for details.)

---

**Happy hacking! 🚦✨**
