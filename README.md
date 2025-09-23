# 🔒 Security Check Workflow for NPM Projects

## Why This Exists (The Critical Incident)

The JavaScript ecosystem has been hit again — this time by an even more **dangerous supply chain attack**.

Recently, attackers compromised the NPM account of a popular maintainer (`@ctrl/tinycolor` and many other packages, millions of weekly downloads).

This wasn’t a simple malware drop. It behaved like a **worm**:

* 🪱 **Self-propagating** — once inside a project, it spread to other repos by modifying GitHub Actions workflows.
* 🕵️ **Credential theft** — stole NPM tokens, GitHub tokens, and even cloud provider secrets (AWS, GCP, Azure).
* ⚙️ **Persistence** — infected repos would *reinfect themselves* through CI/CD pipelines.
* 📤 **Exfiltration** — leaked secrets to external endpoints and even created rogue GitHub repos named `Shai-Hulud`.

This attack showed that **no project is “too small” or “too safe”**.
If a maintainer with millions of weekly downloads can get hit, **any developer’s supply chain can be weaponized overnight**.

---

## What This GitHub Action Does

To protect against this kind of threat, this workflow runs an **automated security audit** on your project’s dependencies and files.

Here’s what it covers:

1. **Triggers**

   * Runs on every push to `main`.
   * Runs on all pull requests.
   * Runs **daily at 6 AM UTC** (scheduled job).

2. **Steps in the Workflow**

   * ✅ Checks out your repo.
   * ✅ Sets up Node.js (v20).
   * ✅ Installs dependencies safely with `npm ci --ignore-scripts` (blocks malicious install scripts).
   * ✅ Runs `npm audit --audit-level=high --omit=dev` to catch vulnerabilities in production deps.
   * ✅ Runs a **malware scan script** that looks for known IoCs (like `shai-hulud`, suspicious webhooks, or malicious file hashes).
   * ✅ Runs a **malicious package scanner** to detect compromised NPM versions (based on a curated blocklist).
   * ✅ **NEW:** Runs **Snyk** as an extra layer of defense — Snyk’s database catches vulnerabilities *faster* and often before `npm audit` does.

3. **Why It Matters**

   * 💡 Detects known vulnerabilities in dependencies.
   * 💡 Warns you if malicious code sneaks in through transitive dependencies.
   * 💡 Provides an **early warning system** before secrets or workflows get compromised.
   * 💡 Strengthens your **supply chain defense** with minimal setup.
   * 💡 **Snyk integration** gives you broader coverage and earlier detection compared to the default NPM advisories.

---

## Example Workflow File

```yaml
name: Security Check

on:
  push:
    branches: [ main ]
  pull_request:
  schedule:
    - cron: "0 6 * * *" # daily

jobs:
  audit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: 20

      - run: npm ci --ignore-scripts
      - run: npm audit --audit-level=high --omit=dev

      # Optional malware scan for Shai-Hulud IoCs
      - run: |
          git ls-files ".github/workflows" | grep shai-hulud && echo "⚠️ Workflow infection detected"
          git ls-remote --heads origin | grep shai-hulud && echo "⚠️ Rogue branch detected"
          grep -R "webhook.site" . && echo "⚠️ Suspicious exfil endpoint"

      # Malicious package scanner
      - name: Scan for malicious packages
        run: |
          node --experimental-permission --allow-fs-read=. .github/scripts/malware-scan.js "."

      # NEW: Snyk vulnerability scan
      - name: Run Snyk to check for vulnerabilities
        uses: snyk/actions/node@v1
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
        with:
          command: test
```

---

## Why You Should Care

Supply chain attacks are no longer rare — they’re **increasing, targeted, and smarter**:

* 🧨 Even trusted packages can turn malicious overnight.
* 🧨 Malware now persists inside CI/CD pipelines and steals secrets.
* 🧨 The cost of compromise is huge: leaked tokens, production access, and user data.

By combining **npm audit**, **malware scanning**, and **Snyk’s intelligence**, this workflow gives you a **defense-in-depth strategy**.
It won’t stop every attack, but it ensures you **never ship known vulnerabilities or silent infections**.

---

## How to Enable It in Your Repo

1. Create the workflows folder (if missing):

   ```bash
   mkdir -p .github/workflows
   ```

2. Add the workflow file:

   ```bash
   .github/workflows/security-check.yml
   ```

3. Copy the code above.

4. Add the malicious package scanner script:

   ```bash
   mkdir -p .github/scripts
   cp malware-scan.js .github/scripts/
   ```

5. Set up Snyk:

   * Create a free [Snyk account](https://snyk.io).
   * Generate a Snyk API token.
   * Add it to your repo as a GitHub Secret named `SNYK_TOKEN`.

6. Commit & push:

   ```bash
   git add .github/workflows/security-check.yml .github/scripts/malware-scan.js
   git commit -m "Add security audit, malware scan, and Snyk workflow"
   git push origin main
   ```

---

## Show It Off: Status Badge

```markdown
![Security Check](https://github.com/your-username/your-repo/actions/workflows/security-check.yml/badge.svg)
```

✅ Green = your dependencies are clean
❌ Red = action required

---

## Final Words

The **Shai-Hulud worm** proved that the ecosystem’s weakest link is the **supply chain itself**.
We can’t blindly trust dependencies anymore — we need to **verify continuously**.

This workflow now goes beyond a basic `npm audit`. With **Snyk’s zero-day detection** + **malware scanning**, it’s a **practical, layered defense** for modern JavaScript projects.

If you found this useful, ⭐ star the repo and share it with your team.
The more developers adopt these practices, the harder it becomes for attackers to succeed.

Stay safe. Keep building. 🚀
