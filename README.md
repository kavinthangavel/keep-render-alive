# 🚀 Render Keep-Alive Service (Simplified)

## 📜 Overview

The Render Keep-Alive Service is a simplified setup using a GitHub Actions workflow to prevent your web services (especially those on free tiers like Render, Heroku, etc.) from idling or going to sleep due to inactivity. It achieves this by periodically pinging specified health check endpoints defined in a `websites.json` file that you manage manually within your GitHub repository.

This project is ideal for developers looking for a straightforward, secure, and low-maintenance way to ensure continuous availability for their hobby projects or demos.

## ✨ Features

- **Automated Pinging**: Uses GitHub Actions to periodically ping configured services.
- **Manual Configuration**: Website list (`websites.json`) is manually edited and committed to your GitHub repository.
- **Secure & Simple**: No separate admin application to host or secure. All configuration is managed via Git.
- **Customizable**: Define which services to ping and their health endpoints in `websites.json`.
- **Scheduled**: GitHub Action runs on a configurable cron schedule (e.g., every 14 minutes).

## 🛠️ How It Works

The project relies on two main components:

1.  **`websites.json` (Manually Managed in GitHub Repository)**:
    -   A JSON file stored in your GitHub repository (e.g., at the root).
    -   You manually edit this file to list the websites to be pinged, their URLs, and whether they are enabled.
    -   This file is the single source of truth for the GitHub Actions workflow.

2.  **GitHub Actions Workflow (`.github/workflows/keep-alive.yml`)**:
    -   Runs on a configurable schedule (e.g., every 14 minutes).
    -   Checks out the latest version of your repository.
    -   Reads the `websites.json` file.
    -   Dynamically creates a matrix of enabled websites to ping.
    -   Pings the specified health endpoint (e.g., `/healthz` or the site URL itself) for each active service using `curl`.

### Workflow

1.  **Manual Configuration**: You (the administrator) directly edit the `websites.json` file in your GitHub repository to add, remove, or modify website entries.
2.  **Commit Changes**: You commit and push the changes to `websites.json` to your repository's main branch.
3.  **Scheduled GitHub Action**: The `keep-alive.yml` workflow triggers at its next scheduled time.
4.  **Fetch Latest Config**: The workflow checks out the repository, getting the latest `websites.json`.
5.  **Ping Services**: The workflow iterates through the enabled websites in `websites.json` and sends HTTP requests to keep them alive.

## 📁 Project Structure

```
.
├── .github/
│   └── workflows/
│       └── keep-alive.yml  # GitHub Actions workflow for pinging
├── websites.json           # Manually edited list of websites to ping
└── README.md               # This file
```

## 📋 Prerequisites

- **Git**: For version control and managing `websites.json`.
- **GitHub Account**: To host the repository and run GitHub Actions.

## 💻 Setup and Usage

Follow these steps to set up and use the simplified keep-alive service.

### 1. Clone or Create the Repository

If you are starting from scratch, create a new GitHub repository. If you are adapting this project, clone your existing repository.

```bash
# Example: Cloning an existing repository
git clone <your-repository-url>
cd <repository-folder-name>
```

### 2. Create/Configure `websites.json`

In the root of your repository, create or edit the `websites.json` file. This file will contain the list of services you want to keep alive.

*Example `websites.json` structure:*
```json
{
  "settings": {
    "pingIntervalMinutes": 14,
    "enableMultipleServices": true,
    "maxRetries": 3
  },
  "websites": [
    {
      "id": "service1",
      "name": "My Awesome App",
      "url": "https://my-awesome-app.onrender.com/healthz",
      "enabled": true,
      "addedAt": "2025-06-07T10:02:00.000Z",
      "lastPingResult": "pending"
    },
    {
      "id": "service2",
      "name": "Another Demo Project",
      "url": "https://another-project.herokuapp.com",
      "enabled": false,
      "addedAt": "2025-06-07T10:03:00.000Z",
      "lastPingResult": "disabled"
    }
  ]
}
```

**Fields Explained:**

-   **`settings`**: (Optional, but good for reference)
    -   `pingIntervalMinutes`: Reference for how often pings are intended. The actual interval is set in the GitHub Action's cron schedule.
    -   `enableMultipleServices`: If `true`, the GitHub Action might process services in parallel (depends on workflow setup).
    -   `maxRetries`: Reference for how many times a ping might be retried (depends on workflow setup).
-   **`websites`**: An array of website objects.
    -   `id` (String): A unique identifier for the website entry (e.g., "service1", "my-app").
    -   `name` (String): A human-readable name for the service.
    -   `url` (String): The full URL to be pinged (e.g., `https://your-service.onrender.com/healthz` or just the root URL if it responds to pings).
    -   `enabled` (Boolean): If `true`, this website will be pinged by the GitHub Action. If `false`, it will be skipped.
    -   `addedAt` (String - ISO Date): Optional, when the service was added.
    -   `lastPingResult` (String): Optional, can be updated by the GitHub Action if configured to do so (e.g., "healthy", "unhealthy", "error"). This requires the Action to have write permissions back to the repo, which adds complexity and is not part of the default simplified setup.

**Manually edit this file to add your services.**

### 3. Configure the GitHub Actions Workflow (`.github/workflows/keep-alive.yml`)

Ensure the `.github/workflows/keep-alive.yml` file is present in your repository. This workflow is responsible for reading `websites.json` and pinging the services.

A typical `keep-alive.yml` might look like this:

```yaml
name: Keep Alive Services

on:
  schedule:
    # Runs every 14 minutes
    - cron: '*/14 * * * *'
  workflow_dispatch: # Allows manual triggering

jobs:
  ping_services:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Repository
        uses: actions/checkout@v4

      - name: Read Websites Configuration
        id: read_config
        run: |
          config=$(cat websites.json | jq -c '.websites[] | select(.enabled == true) | .url')
          echo "urls_to_ping=$config" >> $GITHUB_OUTPUT
        # Ensure jq is installed if not available by default on runner
        # For more complex parsing or if jq is not preferred, a script (Node.js, Python, shell) can be used.

      - name: Ping Websites
        if: steps.read_config.outputs.urls_to_ping != ''
        run: |
          for url in ${{ steps.read_config.outputs.urls_to_ping }};
          do
            # Remove surrounding quotes from jq output if any
            clean_url=$(echo $url | tr -d '\\"') # Ensure this line is correctly handling quotes for shell
            echo "Pinging $clean_url"
            # Fetch the full response. --fail makes curl exit with an error if HTTP status code is >= 400.
            # The response will be printed to the workflow log.
            response_output=$(curl --silent --fail --retry 3 --retry-delay 5 "$clean_url")
            curl_exit_code=$? # Capture exit code of curl

            if [ $curl_exit_code -eq 0 ]; then
              echo "Successfully pinged $clean_url"
              echo "Response from $clean_url:"
              echo "$response_output"
            else
              echo "Failed to ping $clean_url (Exit code: $curl_exit_code)"
              echo "Response/Error from $clean_url (if any):"
              echo "$response_output" # Print output even on failure, as it might contain error details
            fi
          done

      - name: No services to ping
        if: steps.read_config.outputs.urls_to_ping == ''
        run: echo "No enabled services found in websites.json to ping."
```

**Key parts of the workflow:**

-   **`on.schedule.cron`**: Defines how often the workflow runs. `\'*/14 * * * *\'` means every 14 minutes. Adjust as needed (e.g., Render free tier sleeps after 15 minutes of inactivity).
-   **`actions/checkout@v4`**: Fetches the latest version of your repository, including `websites.json`.
-   **Reading `websites.json`**: The example uses `cat` and `jq` to parse `websites.json` and extract URLs of enabled services. You might need to install `jq` or use a different method (e.g., a small script) if your runner doesn't have `jq` or if your JSON structure is more complex.
-   **Pinging**: Iterates through the extracted URLs and uses `curl` to send a request. The `curl` command can be customized (e.g., using `GET` instead of `HEAD`, adding specific headers).

### 4. Commit and Push Changes

After setting up `websites.json` and `.github/workflows/keep-alive.yml`:
```bash
git add websites.json .github/workflows/keep-alive.yml README.md
git commit -m "Configure keep-alive service with initial websites and workflow"
git push origin main
```

### 5. Monitor GitHub Actions

-   Go to the "Actions" tab in your GitHub repository.
-   You should see the "Keep Alive Services" workflow.
-   It will run automatically on the schedule you defined. You can also trigger it manually if `workflow_dispatch` is enabled.
-   Check the workflow logs to ensure it's reading `websites.json` correctly and pinging your services.

## 🔐 Security Considerations

-   **Repository Visibility**: If your `websites.json` contains URLs to sensitive or private internal services, ensure your GitHub repository is private.
-   **GitHub Actions Permissions**: By default, GitHub Actions workflows in a repository have read access to that repository. If your workflow needs to write back (e.g., to update `lastPingResult` in `websites.json`), it would require write permissions, which should be configured carefully using a `GITHUB_TOKEN` with appropriate scopes, passed as a secret to the Action. The simplified setup described here assumes read-only access to `websites.json` by the Action.

## 📄 License

This project is licensed under the MIT License. See the `LICENSE` file for details.

(If you don't have a `LICENSE` file, you should add one. Here's a template for MIT):

```text
MIT License

Copyright (c) [Year] [Your Name or Organization Name]

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```
Replace `[Year]` and `[Your Name or Organization Name]` accordingly.
